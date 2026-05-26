# AP-10677 开发总结

## 涉及服务

* cloud-future-streamer ([567ded8a](https://git.toolsapple.net/ApolloX/cloud-future-streamer/commit/567ded8af6e9b2bb912a0f7bd5cc0cfda0cc42dd)): [PR](https://git.toolsapple.net/ApolloX/cloud-future-streamer/pull/89)
* new-binance-mgs ([b722abb7](https://git.toolsapple.net/ApolloX/new-binance-mgs/commit/b722abb7892c631a72f867d5e4ae2911ff47fc94)): [PR](https://git.toolsapple.net/ApolloX/new-binance-mgs/pull/451)

## 改动说明

为前端盈亏平衡价功能提供后端支持。修复双向全仓（Hedge Mode + Cross Margin）场景下 FundingFee 统计错误的 bug，并新增查询开仓仓位累计 fundingFee 和 commissionFee 的接口。

### cloud-future-streamer

* 修复双向全仓 FundingFee：ME 消息不含 PositionItem 时，查出该用户该 symbol 下 LONG/SHORT 所有方向仓位，按持仓数量比例拆分 fundingFee
* 新增 `getOpenPositionHistoriesBySymbols` Mapper，按 userId + symbols 批量查询开仓仓位
* 新增 `getPositionHistoryTxFee` / `getPositionHistoryTxFeesByIds` Mapper，聚合计算 commissionFee，支持全量和增量两种查询
* 新增 Feign 接口 `getOpenPositionFee`，暴露内部 `POST /position/openPositionFee`
* 实现 `getOpenPositionFees` Service，含 Guava cache 优化：超过阈值的仓位记录 maxInsertTime，后续增量查询
* 新增 Controller 实现 `POST /position/openPositionFee`

### new-binance-mgs

* 新增 `POST /v1/private/future/user-data/position-fee` 对外接口，从 `getFutureUserId()` 获取 userId，调用 Feign `getOpenPositionFee`

## 改动接口

### new-binance-mgs

#### POST /v1/private/future/user-data/position-fee (新增)

> 查询用户的FundingFee和手续费

请求参数：
```json
{ "symbols": ["BTCUSDT", "ETHUSDT"] }`
```

返回格式：
```json
[
    {
        "id":"123", 
        "symbol":"BTCUSDT", 
        "positionSide":1,
        "fundingFee":"0.2", 
        "commissionFee":"-0.3" 
    },
    {
        "id":"124", 
        "symbol":"BTCUSDT", 
        "positionSide":2,
        "fundingFee":"0.2", 
        "commissionFee":"-0.3" 
    }
]
```

## 改动DB

无

## 配置修改

### cloud-future-streamer

| 操作 | 配置 Key | 默认值 | 说明 |
| --- | --- | --- | --- |
| 新增 | position.tx.count.threshold | 2000 | position\_transaction\_history 记录数超过该值时启用增量缓存查询 |

## 改动XXL-JOB

### cloud-future-streamer

- FundingFeeCalc(新增)
    - 说明: 定期计算FundingFee
    - 执行频率: 0 0/5 * * *
    - 执行参数: 无

## 注意

- 要更新 cloud-futurestreamer-web的jar包，然后再发布 cloud-mgs-future 服务
