# Global Claude Rules

## 语言

默认使用中文与用户沟通，除非用户明确要求使用其他语言。

---

## 代码提交规范

- 禁止以任何方式将代码推送到远程，只能在本地修改和提交。
- Commit message 必须使用英文。
- Commit message 禁止包含任何 AI 相关信息。

### Commit Message 格式

示例：

```
feature: Add new Mapper & Entity
- TaskEntity.java: new Entity for Task
- TaskMapper.java: new Mapper for Entity.java
- +200 files more...
```

规则：

- 首行以 `feature` 或 `fix` 开头：
  - `feature`：新功能
  - `fix`：修复现有功能
- 首行冒号后跟一句简短的改动说明，不超过 20 个字符。
- 之后以列表形式列出每个修改的文件：
  - 最多列出 10 行；超出时以 `+N files more...` 形式补充。
  - 只写文件名，不写全路径。
  - 文件名后跟一句简短的修改说明，不超过 20 个字符。

---

## PR 生成规范

- 禁止以任何方式自动提交 PR，只需给出 PR 所需内容，由我手动提交。
- PR 内容必须使用英文。
- PR 内容禁止包含任何 AI 相关信息。

### PR 内容格式

#### Title

```
AP-1238: Forward claim requestedAmount to staking-api for EIP-712 assertion
```

- 若当前上下文存在开发标识 ID（如 Jira ID），则以该 ID 开头。
- ID 后跟一句简短的改动说明。

#### Content

```
## Summary

- `WithdrawHelper.claimRewards()` now passes `requestedAmount` (parsed from `ClaimRewardsArg`) into `ClaimRewardsRequest`, so staking-api can assert it matches the server-computed total yield before executing the claim
- Bumps staking-api dependency `1.2.3` → `1.2.4-SNAPSHOT` (ApolloX/staking-api#92)

## Test plan

- Claim with correct `requestedAmount` succeeds end-to-end
- Claim with stale/mismatched amount returns `CLAIM_AMOUNT_MISMATCH` (195020)
- Disaster-mode fallback carries `requestedAmount` through the same request object
```

- **Summary**：整体改动概述，以列表形式列出，不超过 8 条。
- **Test plan**：关键验证点说明，以列表形式列出，不超过 5 条。

---

## Code Review 规范

- 进行 Code Review 时，禁止直接修改代码；应先列出所有需要改动的点，经用户确认后再执行。
- 首次反馈时，简短列出所有问题，每个问题给出编号和一句话说明。
- 用户以编号沟通，针对某个问题展开讨论时，提供以下详细信息：
  - 出现问题的文件
  - 出现问题的行号
  - 给出的改动建议
- 用户完成修改后，重新检查并逐条标注每个改动点的完成情况。
