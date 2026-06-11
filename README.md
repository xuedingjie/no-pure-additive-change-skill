# No Pure Additive Change Skill

**语言切换**：中文 | [English](README.en.md)

这是一个面向 AI 编码 Agent 的通用 `rule + skill` 包，用来约束后端需求实现时不要默认走“纯加法”路线。

所谓“纯加法”，是指接到需求后直接新增字段、表、枚举、状态、分支、回调、定时任务、统计字段或第三方 payload，却没有判断现有模型是否已经变形、source of truth 是否被拆散、数据库落库事实是否真实有效、后续数据量和并发场景是否还能支撑。

这个 skill 的目标不是让 Agent 大规模重构，而是让 Agent 在实现前先做一次有证据的设计判断：

- 能简单新增的，必须证明简单新增不会制造新的债务。
- 该局部修正模型的，不要继续加字段、加表、加分支。
- 涉及迁移的，必须给迁移方案和迁移测试。
- 涉及业务行为重构的，必须给业务测试和业务不变量。
- 涉及竞态的，必须按多实例或多 Pod 部署来设计幂等、锁和事务边界。

## 适用场景

当需求涉及以下内容时，建议启用这个 skill：

- 新增或修改数据库字段、表、索引
- 新增状态、枚举、类型、渠道、来源
- 新增回调、推送、同步、批任务、定时任务、统计、报表
- 新增第三方请求/响应/result 字段
- 修改持久化业务状态
- 修改金额、账单、支付、退款、结算、账户流水、冻结/解冻、融资、风控、合同、状态流转等逻辑
- 在已有 Service 或核心业务类里继续新增分支
- 因为当前模型不适配需求而想再加一个字段或再建一张表
- 拆分、合并、重命名、迁移、重新解释或回填数据

不适合用于纯文案、格式化、局部测试 fixture 或不影响持久化状态和模型边界的小修复。

## 它会强制什么

- **纯加法是兜底方案，不是默认方案。**
- Agent 必须比较“最小新增”和“局部模型修正/演进式重构”。
- 任何设计判断、重构方案、迁移方案、测试方案都必须有证据行。
- 数据库落库事实优先于代码推断。
- 但数据库数据必须先证明对当前业务问题真实有效，不能只凭表存在、字段存在、记录存在下结论。
- 涉及金额、账单、支付、退款、冻结、融资、流水、回调、状态时，必须说明有效记录过滤、金额单位、精度、正负方向、状态语义和对账路径。
- 迁移必须有迁移测试。
- 行为重构必须有业务回归测试和业务不变量。
- 涉及竞态、回调、重试、定时任务、重复请求或并发状态变化时，默认按多实例/多 Pod 部署设计。

<details>
<summary><strong>点击展开：真实有效的数据库证据要求</strong></summary>

数据库证据不是“表里有这个字段”这么简单。Agent 必须说明：

- 业务事实落在哪张表、哪个字段
- 字段业务含义是什么
- 金额单位是什么，例如分、元、美元、基点、第三方自定义单位
- 金额精度和舍入规则是什么
- 金额正负方向是什么，例如应收、应付、退款、冻结、释放、借、贷、收入、支出
- 状态字段表示当前态、历史事件、失败尝试，还是第三方状态
- 哪些记录是有效记录
- 如何排除删除、失败、冲正、作废、测试、历史、临时、迁移数据
- 写入路径、读取路径、派生路径分别是什么
- 唯一键、幂等键、自然键是什么
- 索引是否支撑查询、写入、迁移和对账
- 事务边界在哪里
- 多实例并发下是否会重复写、乱序写或覆盖更新
- 是否能和账单、支付、流水、结算、冻结、融资或第三方记录对账

如果这些问题答不清，不能进入实现。

</details>

<details>
<summary><strong>点击展开：Agent 输出必须包含的判断</strong></summary>

Agent 在实现前应输出类似下面的判断块：

```markdown
### No Pure Additive Design Judgment

**One-line Judgment**
- ...

**Evidence Lines**
- E1 `path/file.java:123`: observed fact and why it matters.
- E2 `references/context-map.md`: project metadata or source-of-truth evidence.
- A Assumption: business meaning that requires confirmation.

**Source Of Truth**
- Current authority:
- Persisted database fact:
- Data validity conditions:
- Amount unit/sign/precision, if relevant:
- Status semantics, if relevant:
- Reconciliation/check path:
- Write path:
- Read/derivation path:

**Option Comparison**
1. Minimal Additive:
2. Local Model Correction:
3. Controlled Evolutionary Refactor:
4. Staged Redesign, if relevant:

**Recommended Plan**
- ...

**Migration Test Plan**
- ...

**Business Test Plan**
- ...

**Concurrency And Multi-Instance**
- ...
```

</details>

## 文件说明

| 文件 | 说明 |
|---|---|
| `AGENTS.md` | 仓库级规则，可以复制到目标项目或合并进已有 Agent 指令 |
| `skills/no-pure-additive-change/SKILL.md` | 可复用的 Agent skill 主体 |
| `skills/no-pure-additive-change/references/context-map-template.md` | 项目上下文模板，用于填写核心表、高风险服务、source of truth、有效记录过滤等 |
| `skills/no-pure-additive-change/references/pressure-scenarios.md` | 该 skill 要阻断的典型失败场景 |
| `README.en.md` | 英文说明，可从顶部语言切换入口进入 |

## 使用方式

1. 将 `AGENTS.md` 复制到目标仓库，或把其中的 `No Pure Additive Change` 规则合并进已有 Agent 指令。
2. 将 `skills/no-pure-additive-change/` 复制到目标仓库或 Agent 的 skill 目录。
3. 基于 `references/context-map-template.md` 填写项目自己的上下文。
4. 要求 Agent 在实现涉及持久化状态、模型边界、数据迁移、状态流转、金额或并发逻辑的需求前，先读取 `skills/no-pure-additive-change/SKILL.md`。

## 项目接入前必须校准

这个 skill 是通用版，但风险判断必须由项目事实校准。

接入具体项目时，至少要补齐：

- P0/P1 表
- 高风险服务、模块或类
- source of truth map
- 金额单位、精度、正负方向
- 状态语义
- 有效记录过滤条件
- 对账路径
- 迁移和回滚约定
- 部署模式和并发假设

不要把这个 skill 当成业务证据本身。它只负责逼迫 Agent 去找证据、比较方案、暴露假设，并在证据不足时停止实现。

## 推荐接入方式

对于老系统或高债务系统，建议先小范围接入：

1. 先选订单、账单、支付、回调、状态流转等高风险域。
2. 整理项目专属的 `context-map-template.md`。
3. 要求 Agent 每次改动前输出设计判断块。
4. Code Review 时重点审查证据行、有效记录过滤、迁移测试、业务测试和多实例并发设计。
5. 每次发现 Agent 走了错误捷径，就把失败模式补充进 `pressure-scenarios.md`。

## License

未指定许可证前，请按私有知识资产处理。
