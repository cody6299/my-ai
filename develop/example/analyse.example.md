# AP-11148 需求分析

## 基本信息
- **Jira**: https://apollox.atlassian.net/browse/AP-11148
- **标题**：Transfer支持USDT，并且纳入目标地址的BSC Chainlimit
- **概述**: 内部转账功能支持USDT资产，且限制ChainLimit

## 需求背景
Internal Transfer（Aster 现货账户间转账）当前只支持 ASTER 币种。本次新增 USDT 支持，并将 USDT internal transfer 的金额记入收款方的 BSC ChainLimit（即该 USDT 只能从 BSC 网络提现）。

---

## 涉及服务
- cloud-future-depositwithdraw
  > 负责充提逻辑相关的业务
- cloud-mgs-depositwithdraw
  > 充提业务对外网关,调用 cloud-future-depositwithdraw服务

## 改动详情

### Kafka 消息
- **Broker**：`kafka-common1.cloudfutures.internal:9092,kafka-common2.cloudfutures.internal:9092,kafka-common3.cloudfutures.internal:9092`
- **Topic**：`asset_transfer`
- **消息格式**：
  ```json
  {
    "eventType": "INTERNAL",
    "timestamp": "...",
    "transactionId": "11223344",
    "fromUid": 123,
    "toUid": 124,
    "asset": "USDT",
    "amount": "123.456",
    "info": "xxx"
  }
  ```

### 1. 新增 Kafka ListenerContainerFactory
参考 `KafkaConfig.java` 中的 `onChainAckListenerContainerFactory`，新增 `assetTransferListenerContainerFactory`：
- broker 地址通过 Apollo 配置注入：`kafka.asset.transfer.consumer.broker`
- `batchListener = true`，`AckMode = MANUAL`

### 2. 新建 `AssetTransferListenService`，消费 Topic `asset_transfer`
处理逻辑：
1. 解析消息 JSON
2. 过滤：`eventType = "INTERNAL"` + `amount != 0` + `from_uid != to_uid`
3. 只处理 `to_uid`（资金接收方）
4. Apollo 配置控制需处理的 asset 白名单（默认 `USDT`）
5. 根据 `to_uid` 调用 `UserTierMapper.getUserTierByUid(to_uid)` 拿到 `UserTierEntity`
   - `accountId` = `userTier.accountId`
   - `address` = `userTier.address`
6. 幂等检查 + 更新 ChainLimit（见下）

### 3. 幂等 + ChainLimit 更新（同一事务）
```
BEGIN TRANSACTION
  INSERT INTO internal_transfer_withdraw_limit_record
    (transaction_id, from_uid, to_uid, asset, amount, info)
  -- INSERT 成功（transaction_id 唯一索引未冲突）则继续，冲突则跳过
  CALL transactionSummaryService.updateDepositAmountForAeUser(
    accountId   = userTier.accountId,
    address     = userTier.address,
    currency    = "USDT",
    chainId     = ${internal.transfer.usdt.chain.id},   // Apollo 配置
    depositAmount = amount
  )
COMMIT
```

### 4. 新增 Apollo 配置项
| 配置 Key | 说明 | 示例值 |
|---|---|---|
| `kafka.asset.transfer.consumer.broker` | asset_transfer topic 的 Kafka broker 地址 | `kafka-common1...:9092,...` |
| `internal.transfer.asset.whitelist` | 需要处理 ChainLimit 的 asset 白名单 | `USDT` |
| `internal.transfer.usdt.chain.id` | USDT Internal Transfer 记入的 chainId | `（BSC 对应值）` |

### 5. 新建表
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
