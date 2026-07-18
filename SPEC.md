---
status: aligned
last-updated: 2026-07-19
topic: spec 守卫 skill 生态
---

# spec 守卫 skill 生态 规范

一套用于长线项目开发时动态记录需求与边界、约束 agent 实现不偏离、控制开发流程的 SKILL 生态。由 3 个 skill 组成：`spec-clarify`（user-invoked 编排器）、`spec-grilling`（model-invoked 核心访谈循环）、`spec-modeling`（model-invoked 动态记录 + 软提醒防偏离）。

## 需求（Requirements）

- R1（无处不在型）：系统应当由 3 个 skill 组成，调用方向单向——`spec-clarify` → `spec-grilling` → `spec-modeling`，不允许反向调用。
- R2（状态驱动型）：当用户键入 `/spec-clarify` 时，系统应当先读项目根的 `SPEC.md`/`CONTEXT.md`/`docs/adr/` 判断新建或续接，再确认主题后调用 `spec-grilling` 启动阶段 ①。
- R3（事件驱动型）：当 `spec-grilling` 访谈中术语/需求/边界/决策敲定时，系统应当立即触发 `spec-modeling` 写入对应文档，不批量。
- R4（不期望型）：如果 `spec-modeling` 在实现期检测到违反不变式的偏离，系统应当自动返工，不提示操作者。
- R5（状态驱动型）：当实现期发现 SPEC 本身有误时，系统应当将 `status` 从 `implementing` 回退到 `draft`，记录"规范偏离"到 `docs/deviations.md`，保留已写代码待重评估，回阶段 ① 只针对冲突条款重访。
- R6（无处不在型）：系统应当仅由用户显式触发 `/spec-clarify` 启动流程编排；`spec-clarify` 设 `disable-model-invocation: true`，模型不得自动调用它。
- R7（事件驱动型）：当 `spec-grilling` 访谈中需要 throwaway prototype 厘清具体设计问题时，系统应当在用户同意后构建 prototype，遵守原型隔离三原则。
- R8（事件驱动型）：当进入阶段 ② 前，系统应当删除所有 throwaway prototype 文件，不纳入业务代码。
- R9（无处不在型）：系统应当将所有实现期偏离记录到 `docs/deviations.md`（单一真相源），SPEC.md 不含偏离记录段落。
- R10（无处不在型）：系统应当仅从 `SPEC.md` 的 YAML frontmatter `status` 字段解析 SPEC 状态，不解析正文自然语言。
- R11（事件驱动型）：当 `spec-grilling` 访谈中用户提出一条要求时，系统应当主动提议分类（定为不变式/需求/边界/ADR），给出推荐分类及理由，由用户拍板——分类决策不由 `spec-modeling` 在写入时做。判断标准用精炼三问（恒成立/违反即灾难/可证伪），详见 `SPEC-FORMAT.md` 的"写入前自检"。

## 边界（Boundaries）

### 范围内
- 3 个 skill 的 SKILL.md 与配套格式模板（SPEC-FORMAT / CONTEXT-FORMAT / ADR-FORMAT / DEVIATIONS-FORMAT）
- 项目文档：SPEC.md / CONTEXT.md / docs/adr/ / docs/deviations.md
- 四阶段门禁：澄清访谈 → 动态记录 → 实现校验 → 收尾同步
- 偏离分级机制：硬阻断（违反不变式）+ 软提醒（其他三类偏离）
- 状态机含回退路径：`implementing → draft`
- throwaway prototype 在访谈期有条件使用

### 范围外（不做）
- 不实现 issue tracker 集成（GitHub/Linear）——mattpocock 的 triage skill 不纳入本生态
- 不实现 TDD 红绿重构循环——mattpocock 的 tdd skill 不纳入本生态
- 不实现代码库架构改进——mattpocock 的 improve-codebase-architecture skill 不纳入本生态
- 不提供 PRD 生成——mattpocock 的 to-prd skill 不纳入本生态
- SPEC.md 不含实现细节——实现细节放代码注释或 ADR
- CONTEXT.md 不含实现决策——实现决策放 docs/adr/
- 不做多上下文项目的 CONTEXT-MAP.md 生成器——仅记录格式规范，不自动生成
- 偏离记录不存放在 SPEC.md——单一真相源在 docs/deviations.md
- 状态字段不出现在 SPEC.md 正文——只在 YAML frontmatter

## 不变式（Invariants）

- I1：调用方向始终单向——`spec-clarify` → `spec-grilling` → `spec-modeling`，任何 skill 不得反向调用 user-invoked skill。
- I2：阶段 ① 出口前不写任何业务实现代码（throwaway prototype 不算业务代码，受原型隔离三原则约束）。
- I3：`docs/deviations.md` 是偏离记录的唯一真相源——SPEC.md、CONTEXT.md、ADR 均不得含偏离记录段落。
- I4：SPEC.md 的 frontmatter 字段集固定为 `status` / `last-updated` / `topic` 三字段，不得增减。
- I5：违反不变式的偏离触发硬阻断——agent 自动返工，不提示操作者。
- I6：throwaway prototype 必须同时满足三原则（位置 `docs/prototypes/`、标注 `THROWAWAY PROTOTYPE`、阶段 ② 前删除），任一缺失即违反不变式。
- I7：`spec-clarify` 必须设 `disable-model-invocation: true`；`spec-grilling` 与 `spec-modeling` 不得设此字段。
- I8：SPEC 状态字段值域固定为 `draft | aligned | implementing | verified | superseded`，不得引入其他值。
- I9：`spec-modeling` 写入不变式前必须做可证伪性自检——能否写出返回 true/false 的检查函数。不可证伪则提示用户重写表述，但仍写入（不阻断）。`spec-modeling` 不重新做分类决策（分类已由 `spec-grilling` 与用户确定）。

## 验收标准（Acceptance）

- AC1：给定项目根无 SPEC.md，当用户键入 `/spec-clarify`，那么系统判定为新建长线项目，进入阶段 ①。
- AC2：给定 spec-grilling 访谈中术语敲定，当触发 spec-modeling，那么 CONTEXT.md 立即写入该术语，不批量。
- AC3：给定实现期检测到违反不变式（如 I5），当 spec-modeling 识别偏离类型，那么 agent 自动返工，不提示操作者。
- AC4：给定实现期检测到越界偏离，当 spec-modeling 识别偏离类型，那么系统提示用户偏离点+根因+后果+建议，然后继续执行。
- AC5：给定实现期发现 SPEC 条目本身有误，当用户确认回退，那么 status 从 `implementing` 改为 `draft`，规范偏离记入 docs/deviations.md，已写代码保留待评估。
- AC6：给定访谈中需要 prototype，当用户同意，那么 prototype 放入 `docs/prototypes/`、顶部标注 THROWAWAY、阶段 ② 前被删除。
- AC7：给定偏离记录需要归档，当 spec-modeling 写入，那么记录含五字段（类型/根因/后果/处理决定/日期），存于 docs/deviations.md。
- AC8：给定 agent 需判断 SPEC 所处阶段，当读取 SPEC.md，那么只解析 YAML frontmatter 的 `status` 字段，不解析正文。
- AC9：给定 spec-grilling 访谈中用户提出要求，当 spec-grilling 提议分类，那么给出推荐分类+理由（用精炼三问判断），由用户拍板后触发 spec-modeling 按分类写入；spec-modeling 写入不变式前做可证伪性自检，不可证伪则提示用户但不阻断。
