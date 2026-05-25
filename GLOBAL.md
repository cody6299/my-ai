# Global Claude Rules

## 语言
默认使用中文与用户沟通，除非用户明确要求使用其他语言

----

## 代码提交规范
-   禁止任何方式将代码提交到远程,只能在本地修改和提交
-   comment 必须使用英文
-   comment 禁止包含任何AI相关的信息

### comment格式定义

----
feature: Add new Mapper & Entity
-   TaskEntity.java: new Entity for Task
-   TaskMapper.java: new Mapper for Entity.java
-   +200 files more......
----

-   使用feature或fix开头
    -   feature: 新功能
    -   fix: 修复现有功能
-   后面紧跟着一行简短的改动说明,不要超过20个字符
-   接下来使用列表的方式,列出每个修改的文件
    -   最多列出10行
    -   如果还有更多,则使用 +200 files more...... 这种形式表现出来
    -   只列出文件名不要使用全路径
-   文件名后面跟着一行简短的修改说明,不要超过20个字符

----

## PR生成规范

-   禁止任何方式自动提交PR，只需要给出提交PR所需的内容由我手动提交
-   提交PR的内容必须使用英文
-   PR内容禁止包含任何AI相关的信息

### 提交PR内容格式定义

#### Title

----
AP-1238: Forward claim requestedAmount to staking-api for EIP-712 assertion

----

-   如果当前上下文有一个开发的标志ID,例如一个JiraId,则使用这个标志ID开头
-   后面跟着本次改动的简短说明, 保持在20个字符以内

#### Content

----
### Summary

-   `WithdrawHelper.claimRewards()` now passes `requestedAmount` (parsed from `ClaimRewardsArg`) into `ClaimRewardsRequest`, so staking-api can assert it matches the server-computed total yield before executing the claim
-   Bumps staking-api dependency `1.2.3` → `1.2.4-SNAPSHOT` (ApolloX/staking-api#92)

### Test plan

-   Claim with correct `requestedAmount` succeeds end-to-end
-   Claim with stale/mismatched amount returns `CLAIM_AMOUNT_MISMATCH` (195020)
-   Disaster-mode fallback carries `requestedAmount` through the same request object

----

-   Summary 是整体改动的概述, 列表的形式列出
    -   不超过8行
    -   每行内容不超过20个字符
-   Test plan 是本次改动关键的验证点简短说明，列表的形式列出
    -   不超过5行
    -   每行内容不超过20个字符

----

### Code Review规范

-   使用 code review时禁止直接改动代码,而是要列出需要改动点,然后由用户确认是否要改动
-   开始只是简短列出本次code review需要改动的点，对每个问题给出一个编号，然后是问题的简短说明
-   用户使用编号沟通，详细讨论某个问题，这时候再列出问题的详细信息，包括
    -   出现问题的文件
    -   出现问题的行号
    -   给出的改动建议
-   用户修改完再次检查，列出每个改动点的完成情况