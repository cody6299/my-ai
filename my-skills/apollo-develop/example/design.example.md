# AP-11148 技术设计

## 涉及服务
- [cloud-future-depositwithdraw](https://git.toolsapple.net/ApolloX/cloud-future-depositwithdraw)

---

## 涉及数据库

- `internal_transfer_withdraw_limit_record`（新增）
  ```sql
  CREATE TABLE `internal_transfer_withdraw_limit_record` (
      `id`             BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT COMMENT 'PK',
      `transaction_id` BIGINT           NOT NULL DEFAULT 0               COMMENT 'unique transaction id',
      `from_uid`       BIGINT           NOT NULL DEFAULT 0               COMMENT 'source user uid',
      `to_uid`         BIGINT           NOT NULL DEFAULT 0               COMMENT 'to user uid',
      `asset`          VARCHAR(64)      NOT NULL DEFAULT ''              COMMENT 'asset',
      `amount`         DECIMAL(65,18)   NOT NULL DEFAULT '0'             COMMENT 'amount',
      `info`           VARCHAR(1024)    NOT NULL DEFAULT ''              COMMENT 'info',
      `db_create_time` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT 'db create time',
      `db_modify_time` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT 'db modify time',
      PRIMARY KEY(`id`),
      UNIQUE `uniq_transaction_id`(`transaction_id`),
      INDEX `idx_to_asset`(`to_uid`, `asset`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
  ```

---

## 涉及Apollo配置

| Key | 说明 | 默认值 | 改动 |
|---|---|---|---|
| `kafka.asset.transfer.consumer.broker` | asset_transfer topic 的 Kafka broker 地址 | 无 | 新增 |
| `kafka.asset.transfer.topic` | 消费 topic | `asset_transfer` |
| `internal.transfer.asset.whitelist` | 需处理 ChainLimit 的 asset 白名单（逗号分隔） | `USDT` | 新增 |
| `internal.transfer.usdt.chain.id` | USDT Internal Transfer 记入的 chain_id（BSC 对应值） | 无 | 新增 |

---

## 改动清单

### cloud-future-depositwithdraw

- `KafkaConfig.java`（修改）
  - 新增 `@Value("${kafka.asset.transfer.consumer.broker:}")` 字段 `assetTransferConsumerBrokers`
  - 新增 Bean `assetTransferListenerContainerFactory`，Value 用 `StringDeserializer`（消息为 JSON 字符串，非 protobuf bytes）
  - 原因：为 asset_transfer topic 提供独立的 Kafka Consumer 配置

- `AssetTransferMessage.java`（新增）
  - 字段：`eventType`, `timestamp`, `transactionId`, `fromUid`, `toUid`, `asset`, `amount`, `info`
  - 原因：Kafka 消息反序列化 DTO

- `AssetTransferListenService.java`（新增）
  - `@KafkaListener(topics = "${kafka.asset.transfer.topic:asset_transfer}", groupId = "depositwithdraw_asset_transfer", containerFactory = "assetTransferListenerContainerFactory")`
  - 过滤条件：`eventType = "INTERNAL"` + `amount != 0` + `fromUid != toUid` + asset 在白名单内
  - 根据 `toUid` 查 `UserTierMapper.getUserTierByUid(toUid)` 拿 `accountId` 和 `sourceAddr`
  - 调用 `processMessage()`，内部用 `TransactionTemplate` 管理事务：
    - INSERT `internal_transfer_withdraw_limit_record`，`DuplicateKeyException` 则跳过
    - 调用 `transactionSummaryService.updateDepositAmountForAeUser()`，返回 0 则 throw 触发回滚
    - `transactionTemplate.execute()` 外层 try-catch 吞掉异常，不影响 Kafka 消费
  - 注入 `@Resource(name = FutureServiceDB.TRANSACTION_TEMPLATE) TransactionTemplate transactionTemplate`
  - 原因：消费 Kafka 消息，将 USDT Internal Transfer 金额记入收款方 BSC ChainLimit

- `InternalTransferWithdrawLimitRecordEntity.java`（新增）
  - 字段：`id`, `transactionId`, `fromUid`, `toUid`, `asset`, `amount`, `info`, `dbCreateTime`, `dbModifyTime`
  - 原因：幂等记录表对应 Entity

- `InternalTransferWithdrawLimitRecordMapper.java`（新增）
  - `int insert(InternalTransferWithdrawLimitRecordEntity record)`
  - 原因：INSERT 操作，`transaction_id` 唯一索引保证幂等

---

## 调用链路

```
改动前：无

改动后：
Kafka(asset_transfer)
  → AssetTransferListenService.listen()
      过滤: eventType=INTERNAL, amount≠0, fromUid≠toUid, asset 在白名单
      → UserTierMapper.getUserTierByUid(toUid)
      → transactionTemplate.execute()
          → InternalTransferWithdrawLimitRecordMapper.insert()   [DuplicateKey → 跳过]
          → TransactionSummaryService.updateDepositAmountForAeUser(
                accountId, userTier.getSourceAddr(), asset, chainId, amount)
              [返回 0 → throw → 回滚 INSERT]
      [外层 catch → log error，Kafka 消费正常继续]
```

---

## 注意事项

- `userTier.getSourceAddr()` 而非 `.sourceAddr`：Aster 用户的 `sourceAddr` 带 `"astherus"` 前缀，getter 会自动截取
- `updateDepositAmountForAeUser` 内部吞掉了所有异常返回 0，需检查返回值为 0 时在事务回调内 throw 触发回滚
- 事务使用 `TransactionTemplate`（`FutureServiceDB.TRANSACTION_TEMPLATE`），不用 `@Transactional`，外层 catch 防止异常扩散影响 Kafka offset 提交
- 单条消息处理失败只 log error、continue，整批 ack 正常提交，防止消息积压
