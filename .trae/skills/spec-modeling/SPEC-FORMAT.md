# SPEC.md 格式

`SPEC.md` 是项目需求/边界/不变式的单一真相源。由 `spec-modeling` 在访谈期边谈边写、在实现期对照检查、在收尾期同步状态。

## 顶部状态

`SPEC.md` 的 YAML frontmatter 字段集**固定为三字段，不得增减**（不变式 I4）：

```markdown
---
status: draft | aligned | implementing | verified | superseded
last-updated: {YYYY-MM-DD}
topic: {功能/模块名}
---
```

**解析规则**：agent 读 SPEC.md 时，**只认 frontmatter 的 `status` 字段**判断所处阶段，不解析正文中的状态文字（防止正文里出现"当前实现中"等自然语言被误判为状态）。

状态值域固定为（不变式 I8）：
- `draft` —— ①②中（SPEC 可修改）
- `aligned` —— ②完成可进③
- `implementing` —— ③中
- `verified` —— ④完成
- `superseded` —— 已被新 spec 取代

状态机含回退路径：`implementing` 发现 SPEC 本身有误时回退到 `draft`（规范偏离，详见 spec-modeling 的"规范偏离"章节）。

## 完整结构模板

```markdown
---
status: draft
last-updated: 2026-07-19
topic: 文章草稿功能
---

# 文章草稿功能 规范

{1-2 句：这个 spec 是关于什么的，为什么存在}

## 需求（Requirements）

用 EARS 句式，每条独立可验，编号便于引用。

- R1（无处不在型）：系统应当 {始终成立的行为}。
- R2（状态驱动型）：当 {系统处于某状态}，系统应当 {行为}。
- R3（事件驱动型）：当 {外部事件} 发生，系统应当 {行为}。
- R4（不期望型）：如果 {错误/异常条件}，系统应当 {容错行为}。

## 边界（Boundaries）

显式的 no 和显式的 yes 同样有价值。

### 范围内
- ...

### 范围外（不做）
- ...（明确划出，防止范围蔓延）

## 不变式（Invariants）

系统在任何情况下都要满足的条件，无论外部输入如何。

- I1：{不变式陈述}。
- I2：{不变式陈述}。

## 验收标准（Acceptance）

给定/当/那么 句式，可映射为测试用例。

- AC1：给定 {前置}，当 {动作}，那么 {可观测结果}。

## 偏离记录

**SPEC.md 不含偏离记录段落**（不变式 I3）。偏离记录的单一真相源是 `docs/deviations.md`，格式见 `DEVIATIONS-FORMAT.md`。
```

## EARS 句式说明

EARS（Easy Approach to Requirements Syntax）让需求可验、无歧义：

| 类型 | 句式 | 适用场景 |
|---|---|---|
| 无处不在型 | 系统应当 {行为}。 | 始终成立的行为，无前置条件 |
| 状态驱动型 | 当 {系统处于某状态}，系统应当 {行为}。 | 依赖系统内部状态 |
| 事件驱动型 | 当 {外部事件} 发生，系统应当 {行为}。 | 依赖外部触发 |
| 不期望型 | 如果 {错误/异常条件}，系统应当 {容错行为}。 | 异常/错误处理 |

## 编号约定

- 需求：`R1`、`R2`...（按出现顺序）
- 边界条目：范围内/范围外各用列表，不必编号
- 不变式：`I1`、`I2`...
- 验收标准：`AC1`、`AC2`...
- 偏离记录编号 `D1`、`D2`... 存放在 `docs/deviations.md`，不在 SPEC.md

编号一旦分配不再复用——删除条目时留空号或重新发布新 SPEC，避免引用错位。
