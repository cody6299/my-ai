---
name: apollo-develop
description: Apollo 项目全流程开发工作流。当用户需要启动或继续一个 Apollo 需求开发（需求分析/设计/开发/测试/发布）时使用。管理本地文档、与 Jira/Confluence/Git 源码仓库交互。
---

# Apollo 项目开发指引

## 关键字约定

| 关键字 | 含义 |
|--------|------|
| **必须** | 一定要满足，无法满足时停止并与用户讨论 |
| **禁止** | 绝对不能做 |
| **可能** | 不确定，需与用户沟通确认 |
| **尽量** | 非强制，尽量满足，无需与用户沟通 |

## 环境变量

| 变量 | 说明 |
|------|------|
| `WORK_PATH_KNOWAGE` | 本地知识库路径 |
| `WORK_PATH_SOURCECODE` | 源码仓库根路径 |
| `WORK_PATH_DOCUMENT` | 本地文档根路径 |

读取不到任何变量时，向用户询问，**禁止**使用空值代替。

## 概念

- **JiraId**：唯一标识，格式 `AP-1234`，贯穿全流程
- **本地文档**：`${WORK_PATH_DOCUMENT}/AP-1234/` 下的 Markdown 文件，无需确认即可操作，但**必须告知**用户
- **远程文档（Confluence）**：根路径 `https://apollox.atlassian.net/wiki/spaces/CF/pages/1194262531`，**上传前必须获得用户确认**
- **需求文档（Jira）**：可通过 MCP 读写，**禁止修改已有内容**，只能追加
- **知识库**：`${WORK_PATH_KNOWAGE}`，分析需求前优先查阅
- **源码仓库**：`${WORK_PATH_SOURCECODE}`，Git 地址 `https://git.toolsapple.net/`

## 核心约束

1. 源码只能本地修改和 commit，**禁止 push 到远程**（即使用户已明确同意）
2. **禁止自动提交 PR**，只输出 PR 所需内容由用户手动提交
3. 上传远程文档（Confluence）**必须得到用户确认**，禁止自动上传
4. 每个流程（除「需求开始」）必须**按顺序执行**，且**必须得到用户许可**才能进入下一流程
5. 一次需求可能涉及多个源码仓库，**必须先了解各仓库对应关系**

## 流程概览

| 流程 | 说明 | 产出文档 |
|------|------|----------|
| 需求开始 | 获取 JiraId，初始化目录和 process.md | — |
| 需求分析 | 理解需求，确认实现方向 | `analyse.md` |
| 需求设计 | 具体实现方案和任务拆分 | `design.md`, `tasks.md` |
| 需求开发 | 逐任务编写代码，收尾生成总结 | `summary.md` |
| 需求测试 | 解决联调/测试中发现的问题 | `fix.xxx.md` |
| 需求完成 | 生成生产发布文档 | `publish.md` |

贯穿全程：`process.md`（进度状态）、`memory.md`（上下文记忆，由 AI 自行维护）

## 会话开始：状态判断

每次进入会话，**必须先加载 `process.md` 和 `memory.md`**，再判断：

**无 `process.md`** → 从「需求开始」流程执行

**开发未结束** → 列出各流程状态，询问是否继续：
```
需求 AP-1234 正在 **需求开发** 流程中：
| 流程 | 状态 | 文档 |
| 需求分析 | 完成 | analyse.md |
| 需求设计 | 完成 | design.md, tasks.md |
是否继续？
```
若处于「需求开发」流程，额外列出 `tasks.md` 的任务完成情况。

**开发完成、测试未完成** → 输出：
```
需求 AP-1234 开发已完成，是否有问题需要修复？还是全部完成？
```

**全部完成（finish:done）** → 列出所有本地文档路径，询问用户有什么问题

## 获取 JiraId（优先级从高到低）

1. 用户明确告知
2. 当前目录名为 JiraId 格式（`AP-\d+`）
3. 当前 Git 分支名为 JiraId 格式
4. `WORK_PATH_DOCUMENT` 下有多个 JiraId 目录，找出 `process.md` 中未完成的，列出后询问
5. 直接询问用户

## process.md 格式

```
analyse:done
design:done
develop:in_progress
```

- 流程开始 → 写入 `xxx:in_progress`
- 流程结束 → 更新为 `xxx:done`
- 全部完成 → 追加 `finish:done`

## 详细内容

- 各流程详细步骤：`references/flows.md`
- 本地文档格式规范：`references/doc-formats.md`
