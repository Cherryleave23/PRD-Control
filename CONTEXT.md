# spec 守卫 skill 生态 上下文

本术语表定义 spec 守卫 skill 生态中的特有术语，确保 agent 与用户使用共享语言。

## 术语表（Language）

**user-invoked skill**：
只能由用户显式键入命令触发的 skill，设 `disable-model-invocation: true`，不被模型自动调用，也不被其他 skill 调用。
_避免_：手动 skill、显式 skill

**model-invoked skill**：
可被用户键入或模型自动触发的 skill，不设 `disable-model-invocation`，可被其他 skill 调用（但不能调用 user-invoked skill）。
_避免_：自动 skill、隐式 skill

**偏离（Deviation）**：
实现期的代码与 SPEC.md 条目之间的不一致，分四类：越界、缺斤少两、违反不变式、触碰范围外。
_避免_：偏差、不一致、差异

**违反不变式（Invariant Violation）**：
实现破坏了 SPEC.md 中"系统在任何情况下都要满足的约束"，等价于"与设计边界完全相反"，触发硬阻断自动返工。
_避免_：约束违反、不变量破坏

**软提醒（Soft Reminder）**：
对越界/缺斤少两/触碰范围外的偏离，提示用户偏离点+根因+后果+建议后继续执行，不阻断。
_避免_：弱提示、非阻断提示

**硬阻断（Hard Block）**：
对违反不变式的偏离，agent 自动返工，不提示操作者。是"不阻断"原则的唯一例外。
_避免_：强制返工、自动回滚

**强提醒（Strong Reminder）**：
偏离处理的三级光谱中间档。仍不阻断（决定权在用户），但风险可见性更高 + 建议更明确。典型场景：返工缺陷 + 功能异常、模型自添加设计 + 功能异常。与"软提醒"的区别在于建议的坚决程度，而非流程阻断。
_避免_：强制提示、高风险提醒

**规范偏离（Spec Deviation）**：
实现期发现 SPEC 条目本身有误（非实现错误，是规范错误），触发状态回退 `implementing → draft`，记录到 docs/deviations.md。
_避免_：SPEC 错误、规范错误

**throwaway prototype（用完即弃原型）**：
访谈期为厘清具体设计问题构建的可运行原型，非业务代码，受原型隔离三原则约束，阶段 ② 前必须删除。
_避免_：原型、demo、示例

**原型隔离三原则（Prototype Isolation Principles）**：
throwaway prototype 必须同时满足：位置在 `docs/prototypes/`、顶部标注 `THROWAWAY PROTOTYPE`、阶段 ② 前强制删除。任一缺失即违反不变式 I6。
_避免_：原型三规则

**根因×后果（Root Cause × Consequence）**：
偏离诊断的二维评估——根因分实现意外/返工缺陷/模型自添加设计/其他，后果分功能正常/功能异常，组合决定处理建议。
_避免_：偏离二维分析

**单一真相源（Single Source of Truth）**：
同一信息只在一个地方存放，避免维护漂移。如偏离记录只在 `docs/deviations.md`，不在 SPEC.md。
_避免_：SSOT、唯一真相源

**不变式分类决策（Invariant Classification）**：
spec-grilling 在访谈中对用户提出的要求提议分类（定为不变式/需求/边界/ADR），给推荐+理由，由用户拍板。判断用精炼三问：恒成立、违反即灾难、可证伪。spec-modeling 不做分类决策，只写入已确定的分类。
_避免_：不变式归类、要求分类

**可证伪性自检（Falsifiability Check）**：
spec-modeling 写入不变式前的轻量表述质量检查——能否写出返回 true/false 的检查函数。不可证伪（如"系统应当合理"）则提示用户重写表述，但仍写入，不阻断。不重新做分类决策。
_避免_：可检查性验证、表述自检

**精炼三问（Three Questions）**：
判断一条要求是否该升格为不变式的三个标准：① 恒成立（系统有合法状态会违反吗？有则不是不变式）；② 违反即灾难（违反会导致数据损坏/安全漏洞/资金错误/业务崩溃吗？不会则降级）；③ 可证伪（能写出 true/false 检查函数吗？不能则重写）。详见 `SPEC-FORMAT.md` 的"写入前自检"。
_避免_：不变式三问、三问自检

**三类误判（Three Misclassifications）**：
不变式分类时的三种常见错误：把行为当不变式（应是需求）、把技术选型当不变式（应是 ADR）、把模糊表述当不变式（应重写）。作为精炼三问的辅助锚点。
_避免_：常见分类错误
