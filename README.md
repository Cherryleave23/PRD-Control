# PRD-Control

> 长线项目开发的规范守卫 SKILL 生态 —— 动态记录产品需求与边界，约束 Agent 实现不偏离，四阶段门禁控制开发流程。

PRD-Control 是一套面向 TRAE（及兼容 Claude Code skill 系统）的 SKILL 生态，专为**长线项目开发**设计。它解决 AI 编程最常见的失败模式：misalignment（你说的和 agent 理解的不一样）、scope creep（范围蔓延）、constraint drift（约束漂移）、context loss（跨会话上下文丢失）。

架构照搬 [mattpocock/skills](https://github.com/mattpocock/skills) 的 grill-me 三层生态（user-invoked 薄包装 → model-invoked 核心循环 → model-invoked 动态记录），在其基础上叠加三个扩展点：**SPEC.md 三段式**（需求+边界+不变式）、**偏离防偏离机制**（硬阻断 + 软提醒分级）、**四阶段门禁**（澄清→记录→实现→收尾）。

## 核心理念

- **可预测性优先** —— agent 每次跑同样的流程，而非产出同样的输出。
- **小、可组合、开发者保持控制权** —— 沿用 mattpocock 哲学，不夺走决策权。
- **单一真相源** —— 同一信息只在一个地方存放，避免维护漂移。
- **正面表述优于否定** —— 告诉 agent 该做什么，而非不该做什么。

## 生态架构

3 个 skill 单向调用，职责清晰：

```
用户键入 /spec-clarify  (user-invoked, disable-model-invocation: true)
        │
        ▼  编排四阶段门禁
   spec-clarify ──启动①──▶ spec-grilling (model-invoked 核心访谈)
                                  │
                                  ▼ 边谈边联动（术语/需求/边界/决策敲定即写）
                           spec-modeling (model-invoked 动态记录 + 偏离防偏离)
```

| Skill | 类型 | 职责 |
|---|---|---|
| **spec-clarify** | user-invoked 薄包装 | 编排"澄清访谈→动态记录→实现校验→收尾同步"四阶段门禁 |
| **spec-grilling** | model-invoked 核心 | relentless 访谈：一次一问、给推荐答案、事实查环境/决策问用户、共识前不行动 |
| **spec-modeling** | model-invoked 记录+守卫 | 边谈边写 SPEC/CONTEXT/ADR；实现期对照 SPEC 偏离防偏离（硬阻断+软提醒分级） |

**调用方向单向**：`spec-clarify` → `spec-grilling` → `spec-modeling`，不允许反向（model-invoked 不调用 user-invoked）。

## 四阶段门禁

| 阶段 | 名称 | 主导 skill | 出口门禁 |
|---|---|---|---|
| ① | 澄清访谈 | spec-grilling | 决策树所有分支解决 + 用户确认共识 |
| ② | 动态记录 | spec-modeling | SPEC.md 含需求+边界+不变式 + CONTEXT.md 术语齐 + 必要 ADR 已写 |
| ③ | 实现校验 | spec-modeling | 实现前预检 + 实现中偏离防偏离已启用 |
| ④ | 收尾同步 | spec-modeling | SPEC 状态更新 + 术语/ADR 同步 + 偏离清单归档 |

**两条硬约束**：阶段 ① 出口前不写任何业务实现代码；阶段 ② 文档未齐不进入实现。

## 偏离防偏离机制（核心特性）

### 分级处理

| 偏离类型 | 处理方式 |
|---|---|
| **违反不变式**（=与设计边界完全相反） | **硬阻断** —— agent 自动返工，不提示操作者 |
| 越界 / 缺斤少两 / 触碰范围外 | **软提醒** —— 提示用户 + 给建议，继续执行 |

硬阻断是"不阻断"原则的**唯一例外**，边界严格限定为"违反不变式"。理由：不变式是系统在任何情况下都要满足的约束，违反它意味着系统进入未定义状态，可能引发数据损坏或级联故障。

### 软提醒的根因×后果诊断

对软提醒类偏离执行二维评估：

- **根因**：实现意外 / 返工缺陷 / 模型自添加设计 / 其他
- **后果**：功能正常 / 功能异常

组合决定处理建议（如"模型自添加设计 + 功能异常 → 强提醒建议移除并重做"）。Step 1-3（类型/根因/后果）是事实，agent 自查；Step 4（处理建议）是决策，用户拍板。

### 规范偏离（SPEC 本身有误）

实现期发现 SPEC 条目本身有误时：状态回退 `implementing → draft`，记规范偏离到 `docs/deviations.md`，保留已写代码待评估，回阶段 ① 只针对冲突条款重访（不推翻整个 SPEC）。

## 文档体系

项目根目录维护 4 类文档，单一真相源：

```
/
├── SPEC.md                    # 需求 + 边界 + 不变式 + 验收标准（frontmatter status 锁定三字段）
├── CONTEXT.md                 # 术语表（glossary，不放实现细节）
├── docs/
│   ├── adr/                   # 架构决策记录（三条件全满足才写）
│   │   └── 0001-{slug}.md
│   ├── deviations.md          # 偏离记录唯一真相源（五字段结构化）
│   └── prototypes/            # throwaway prototype（阶段②前必须删除）
└── src/                       # 业务代码
```

### SPEC.md 三段式

1. **需求（Requirements）** —— EARS 句式，每条独立可验（无处不在型/状态驱动型/事件驱动型/不期望型）
2. **边界（Boundaries）** —— 显式范围内 + 范围外（不做），显式的 no 和显式的 yes 同样有价值
3. **不变式（Invariants）** —— 系统在任何情况下都要满足的条件
4. **验收标准（Acceptance）** —— 给定/当/那么 句式，可映射为测试

### SPEC 状态机（含回退）

```
draft ──→ aligned ──→ implementing ──→ verified
  ▲           │             │
  │           │             │ 发现 SPEC 有误（规范偏离）
  │           │             ▼
  └───────────┴────── 回退（记规范偏离，保留代码待评估）

任意状态 ──→ superseded（被新 SPEC 取代）
```

`status` 字段只在 YAML frontmatter，agent 只解析 frontmatter，不解析正文自然语言。

## throwaway prototype 边界

访谈中途为厘清设计问题可构建 throwaway prototype，受**原型隔离三原则**约束（同时满足才合法）：

1. **目的明确** —— 仅为厘清当前访谈中的具体设计问题，非实现功能
2. **位置隔离** —— 放 `docs/prototypes/`，绝不放 `src/`
3. **生命周期标注** —— 顶部标注 `<!-- THROWAWAY PROTOTYPE: 验证完毕后删除，不纳入业务代码 -->`，阶段 ② 前必须删除

任一缺失即违反不变式。

## 快速开始

### 安装

将 `.trae/skills/` 下的 3 个 skill 目录复制到 TRAE 的 skill 目录：

- **项目级**：`<项目根>/.trae/skills/`
- **全局级**（TRAE）：`c:\Users\<用户>\.trae-cn\skills\`（Windows）或对应平台的用户级 skill 目录

### 使用

在长线项目开发时键入：

```
/spec-clarify
```

编排器会：
1. 读项目根的 `SPEC.md`/`CONTEXT.md`/`docs/adr/` 判断新建或续接
2. 向你确认本次澄清的主题
3. 调用 `spec-grilling` 开始访谈（一次一问 + 推荐答案）
4. 访谈中敲定的术语/需求/边界自动联动 `spec-modeling` 落盘
5. 进入实现阶段后启用偏离防偏离机制

## 目录结构

```
PRD-Control/
├── README.md                          # 本文件
├── SPEC.md                            # 本生态自身的规范（自举）
├── CONTEXT.md                         # 本生态术语表
├── docs/
│   └── adr/
│       └── 0001-硬阻断例外.md          # 关键决策 ADR
└── .trae/
    └── skills/
        ├── spec-clarify/
        │   └── SKILL.md               # user-invoked 编排器
        ├── spec-grilling/
        │   └── SKILL.md               # model-invoked 核心访谈循环
        └── spec-modeling/
            ├── SKILL.md               # model-invoked 动态记录 + 偏离防偏离
            ├── SPEC-FORMAT.md         # SPEC.md 格式模板
            ├── CONTEXT-FORMAT.md      # CONTEXT.md 术语表格式模板
            ├── ADR-FORMAT.md          # ADR 决策记录格式模板
            └── DEVIATIONS-FORMAT.md   # 偏离记录五字段模板
```

## 与 mattpocock/skills 的关系

本生态照搬 [mattpocock/skills](https://github.com/mattpocock/skills) 的三层架构与核心理念（可预测性、单一真相源、开发者控制权），在其基础上扩展：

| 扩展点 | mattpocock 原版 | PRD-Control |
|---|---|---|
| 文档体系 | CONTEXT.md（术语）+ ADR | + SPEC.md 三段式（需求+边界+不变式）+ deviations.md |
| 偏离处理 | 无 | 硬阻断（违反不变式）+ 软提醒分级（根因×后果诊断）|
| 流程控制 | 无显式门禁 | 四阶段门禁（澄清→记录→实现→收尾）|
| prototype | 独立 skill | 原型隔离三原则（位置+标注+强制删除）|
| 状态机 | 无 | SPEC frontmatter status，含回退路径 |

## 适用场景

- **长线项目开发** —— 需求复杂、跨多次会话、需防止范围蔓延与约束漂移
- **跨设备办公** —— 文档外化设计天然支持，换设备零摩擦续接
- **少量 agent 分工** —— SPEC 契约 + 偏离可追溯 + ADR 对齐
- **spec-driven 开发** —— 偏好先规范后实现的工程纪律

**不适用**：一次性脚本、快速原型、不需规范约束的小型任务。

## 设计决策

关键决策记录在 `docs/adr/` 与 SPEC.md 不变式段落：

- **硬阻断例外**（ADR-0001）—— 违反不变式自动返工，是"不阻断"原则的唯一例外
- **偏离记录单一真相源** —— 只在 `docs/deviations.md`，SPEC.md 不含偏离记录段落
- **状态机含回退** —— `implementing → draft` 规范偏离路径
- **frontmatter 字段锁定** —— SPEC.md 三字段固定，agent 只解析 frontmatter
- **throwaway prototype 三原则** —— 位置+标注+强制删除

## License

MIT

## 致谢

- [mattpocock/skills](https://github.com/mattpocock/skills) —— 三层架构与工程纪律灵感来源
- [GitHub Spec Kit](https://github.com/github/spec-kit) —— EARS 需求句式与 spec-driven 方法论
