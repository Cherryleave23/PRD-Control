---
status: aligned
last-updated: 2026-07-20
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
- R12（事件驱动型）：当 `spec-grilling` 访谈中连续 2 个分支未引出新决策点时，系统应当主动汇总已有决策清单，并询问用户是否达成共识——防止决策树无限细分消耗用户注意力。
- R13（无处不在型）：系统应当将根因×后果矩阵与偏离分级表作为单一真相源维护在 `DEVIATIONS-FORMAT.md`，其他文件（spec-modeling 等）只引用、不内联副本。
- R14（事件驱动型）：当规范偏离触发状态回退时，系统应当在 `docs/deviations.md` 偏离记录中标注"待评估代码位置"字段（受影响的文件路径），并在续接时由 `spec-clarify` 扫描提醒用户。

## 边界（Boundaries）

### 范围内
- 3 个 skill 的 SKILL.md 与配套格式模板（SPEC-FORMAT / CONTEXT-FORMAT / ADR-FORMAT / DEVIATIONS-FORMAT）
- 项目文档：SPEC.md / CONTEXT.md / docs/adr/ / docs/deviations.md
- 四阶段门禁：澄清访谈 → 动态记录 → 实现校验 → 收尾同步
- 偏离分级机制：硬阻断（违反不变式）+ 强提醒 + 软提醒，矩阵单一真相源在 DEVIATIONS-FORMAT.md
- 状态机含回退路径：`implementing → draft`，规范偏离含待评估代码位置字段
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
  > 自检：`all(not skill.disable_model_invocation for skill in callees)` —— 被调 skill 不得有 disable-model-invocation
- I2：阶段 ① 出口前不写任何业务实现代码（throwaway prototype 不算业务代码，受原型隔离三原则约束）。  
  > 自检：`phase == 1 and file_written_to in src_dir → file_is_prototype or violation`
- I3：`docs/deviations.md` 是偏离记录的唯一真相源——SPEC.md、CONTEXT.md、ADR 均不得含偏离记录段落。  
  > 自检：`'偏离记录' in SPEC.md or '偏离' in CONTEXT.md → grep for deviation section → violation if found`
- I4：SPEC.md 的 frontmatter 字段集固定为 `status` / `last-updated` / `topic` 三字段，不得增减。  
  > 自检：`set(spec.frontmatter.keys()) == {'status', 'last-updated', 'topic'}`
- I5：违反不变式的偏离触发硬阻断——agent 自动返工，不提示操作者。  
  > 自检：`deviation.type == '违反不变式' → agent.response_to_user is None and agent.reworks`
- I6：throwaway prototype 必须同时满足三原则（位置 `docs/prototypes/`、标注 `THROWAWAY PROTOTYPE`、阶段 ② 前删除），任一缺失即违反不变式。  
  > 自检：`all(p in docs/prototypes/ for p in prototypes) and all('THROWAWAY' in read(p) for p in prototypes) and (phase < 2 or len(prototypes) == 0)`
- I7：`spec-clarify` 必须设 `disable-model-invocation: true`；`spec-grilling` 与 `spec-modeling` 不得设此字段。  
  > 自检：`spec_clarify.disable_model_invocation == True and spec_grilling.disable_model_invocation is None and spec_modeling.disable_model_invocation is None`
- I8：SPEC 状态字段值域固定为 `draft | aligned | implementing | verified | superseded`，不得引入其他值。  
  > 自检：`spec.status in {'draft','aligned','implementing','verified','superseded'}`
- I9：`spec-modeling` 写入不变式前必须做可证伪性自检——能写出返回 true/false 的检查函数吗？能则以 `> 自检：\`{伪代码}\`` 格式附伪代码在不变式条目后；不能则提示用户重写表述，但仍写入（不阻断）。`spec-modeling` 不重新做分类决策（分类已由 `spec-grilling` 与用户确定）。  
  > 自检：`all('> 自检：' in inv for inv in spec.invariants if inv.falsifiable is True)`
- I10：根因×后果矩阵与偏离分级表的单一真相源是 `DEVIATIONS-FORMAT.md`——spec-modeling 等其他文件通过引用获取矩阵内容，不得内联副本。  
  > 自检：`not (grep '处理建议矩阵' spec_modeling_SKILL.md or grep '偏离分级表' spec_modeling_SKILL.md without '详见 DEVIATIONS-FORMAT')` —— spec-modeling 正文不得含矩阵表格
- I11：`docs/deviations.md` 偏离记录模板含六字段（类型/根因/后果/处理决定/日期/待评估代码位置），"待评估代码位置"为可选字段，仅规范偏离时填写。  
  > 自检：`all(len(deviation.fields) == 6 for deviation in deviations) and all(d.code_location is None or d.type == '规范偏离' for d in deviations)`

## 验收标准（Acceptance）

- AC1：给定项目根无 SPEC.md，当用户键入 `/spec-clarify`，那么系统判定为新建长线项目，进入阶段 ①。
- AC2：给定 spec-grilling 访谈中术语敲定，当触发 spec-modeling，那么 CONTEXT.md 立即写入该术语，不批量。
- AC3：给定实现期检测到违反不变式（如违反 I1-I9 中任一），当 spec-modeling 识别偏离类型为"违反不变式"，那么 agent 自动返工，不提示操作者。
- AC4：给定实现期检测到越界偏离，当 spec-modeling 识别偏离类型，那么系统提示用户偏离点+根因+后果+建议，然后继续执行。
- AC5：给定实现期发现 SPEC 条目本身有误，当用户确认回退，那么 status 从 `implementing` 改为 `draft`，规范偏离记入 docs/deviations.md（含待评估代码位置字段），已写代码保留待评估。
- AC6：给定访谈中需要 prototype，当用户同意，那么 prototype 放入 `docs/prototypes/`、顶部标注 THROWAWAY、阶段 ② 前被删除。
- AC7：给定偏离记录需要归档，当 spec-modeling 写入，那么记录含六字段（类型/根因/后果/处理决定/日期/待评估代码位置），存于 docs/deviations.md。
- AC8：给定 agent 需判断 SPEC 所处阶段，当读取 SPEC.md，那么只解析 YAML frontmatter 的 `status` 字段，不解析正文。
- AC9：给定 spec-grilling 访谈中用户提出要求，当 spec-grilling 提议分类，那么给出推荐分类+理由（用精炼三问判断），由用户拍板后触发 spec-modeling 按分类写入；spec-modeling 写入不变式前做可证伪性自检（附伪代码），不可证伪则提示用户但不阻断。
- AC10：给定 spec-grilling 访谈中连续 2 个分支未引出新决策点，当 grilling 识别此条件，那么主动汇总已有决策清单并询问用户是否达成共识。
- AC11：给定偏离诊断进入 Step 4 查矩阵，当 agent 需要根因×后果矩阵，那么从 `DEVIATIONS-FORMAT.md` 获取矩阵内容，不依赖其他文件的副本。
- AC12：给定规范偏离记录包含待评估代码位置，当 spec-clarify 续接项目，那么扫描 `docs/deviations.md` 中该字段非空的条目并提醒用户"以下代码待重评估"。
