---
name: spec-clarify
description: 长线项目开发的统一入口，编排"澄清访谈→动态记录→实现校验→收尾同步"四阶段门禁。当用户要启动新功能/模块的长线开发、或需要对齐需求与边界时调用。
disable-model-invocation: true
---

# spec-clarify —— 长线项目开发编排器

## 你的角色

你是 user-invoked 薄包装编排器。你不亲自做访谈，也不亲自写文档——你的职责是：

- 开启四阶段门禁；
- 在阶段切换时检查出口条件；
- 把控制权交接给 `spec-grilling` 和 `spec-modeling`。

你不会被模型自动调用（`disable-model-invocation: true`），只能由用户显式触发。这是刻意的：流程编排的启动权属于用户，避免模型擅自开启长线流程。

## 四阶段门禁

| 阶段 | 名称 | 主导 skill | 出口门禁（满足后才进下一阶段） |
|---|---|---|---|
| ① | 澄清访谈 | spec-grilling | 决策树所有分支解决 + 用户确认共识 |
| ② | 动态记录 | spec-modeling | SPEC.md 含需求+边界+不变式 + CONTEXT.md 术语齐 + 必要 ADR 已写 |
| ③ | 实现校验 | spec-modeling | 实现前预检完成 + 实现中偏离软提醒已启用（不阻断） |
| ④ | 收尾同步 | spec-modeling | SPEC 状态更新 + 术语/ADR 同步 + 偏离清单归档 |

**两条硬约束**：
- 阶段 ① 出口前不写任何实现代码（对应 grilling 的"共识前不行动"）。
- 阶段 ② 文档未齐不进入实现——SPEC 三段缺一不可。

## 启动流程

1. **读取现状**：读项目根的 `SPEC.md`、`CONTEXT.md`、`docs/adr/`（若存在），判断本次是新建长线项目还是续接。
2. **确认主题**：向用户确认本次要澄清的主题（一个功能/模块/决策）。
3. **启动阶段 ①**：调用 `spec-grilling` skill 开始访谈。

## 阶段切换检查清单

每次阶段切换前，逐项确认：

- **① → ②**：决策树所有分支是否解决？用户是否明确确认"已达成共识"？
- **② → ③**：SPEC.md 是否含需求+边界+不变式三段？CONTEXT.md 术语是否齐？关键决策是否已落 ADR？
- **③ → ④**：用户是否宣告实现完成？
- **④ → 结束**：SPEC 状态是否更新？偏离清单是否归档？

## 续接长线项目

长线项目跨多次会话。每次 `/spec-clarify` 时：

- 优先读现有 `SPEC.md`/`CONTEXT.md`/`docs/adr/`，从上次中断的阶段继续。
- 若现有文档与当前理解冲突，先回到阶段 ① 厘清冲突，而非直接覆盖。
- 读取 `SPEC.md` 的 **YAML frontmatter `status` 字段**判断所处阶段——**只解析 frontmatter，不解析正文自然语言**（防止正文出现"当前实现中"等措辞被误判为状态）。状态值域固定为：

  - `draft` —— ①②中（SPEC 可修改）
  - `aligned` —— ②完成可进③
  - `implementing` —— ③中
  - `verified` —— ④完成
  - `superseded` —— 已被新 spec 取代

### 状态机含回退

状态机非纯线性——实现期（`implementing`）若发现 SPEC 条目本身有误（规范偏离，非实现错误），回退到 `draft`：

```
draft ──→ aligned ──→ implementing ──→ verified
  ▲           │             │
  │           │             │ 发现 SPEC 有误（规范偏离）
  │           │             ▼
  └───────────┴────── 回退（记规范偏离到 docs/deviations.md，保留代码待评估）

任意状态 ──→ superseded（被新 SPEC 取代）
```

回退时：`status` 改回 `draft`，规范偏离记入 `docs/deviations.md`，已写代码保留待重评估，回阶段 ① 只针对冲突条款重访（不推翻整个 SPEC）。详细流程见 `spec-modeling` 的"规范偏离"章节。

## 极简原则

本 skill 的 body 刻意保持精炼。所有访谈纪律见 `spec-grilling`，所有文档纪律见 `spec-modeling`。不要在此重复细节——单一真相源，改动只改一处。
