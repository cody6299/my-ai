# AP-10677 开发总结

## 涉及服务

* cloud-future-streamer ([567ded8a](https://git.toolsapple.net/ApolloX/cloud-future-streamer/commit/567ded8af6e9b2bb912a0f7bd5cc0cfda0cc42dd)): [PR](https://git.toolsapple.net/ApolloX/cloud-future-streamer/pull/89)
* new-binance-mgs ([b722abb7](https://git.toolsapple.net/ApolloX/new-binance-mgs/commit/b722abb7892c631a72f867d5e4ae2911ff47fc94)): [PR](https://git.toolsapple.net/ApolloX/new-binance-mgs/pull/451)

## 改动说明

为前端盈亏平衡价功能提供后端支持。修复双向全仓（Hedge Mode + Cross Margin）场景下 FundingFee 统计错误的 bug，并新增查询开仓仓位累计 fundingFee 和 commissionFee 的接口。

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

## 发版顺序
- 修改Apollo配置
- 发布 cloud-future-depositwithdraw 服务
- 更新 cloud-future-depositwithdraw 的Jar包
- 发布 cloud-mgs-depositwithdraw服务
- 配置 FundingFeeCalc 定时任务
