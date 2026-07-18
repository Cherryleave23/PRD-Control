# CONTEXT.md 格式

`CONTEXT.md` 是项目的术语表（glossary）。由 `spec-modeling` 在访谈期边谈边写。

## 关键约束

`CONTEXT.md` **只是 glossary**——定义"是什么"，不定义"做什么"。它：

- 不是 spec（需求放 `SPEC.md`）
- 不是草稿本
- 不放实现决策（实现决策放 `docs/adr/`）
- 不放实现细节

## 单上下文结构模板

```markdown
# {上下文名}

{1-2 句：这个上下文是什么，为什么存在}

## 术语表（Language）

**订单（Order）**：
{1-2 句：术语定义，说明"是什么"而非"做什么"}
_避免_：Purchase, transaction

**发票（Invoice）**：
交付后发给客户的付款请求。
_避免_：Bill, payment request

**客户（Customer）**：
下单的个人或组织。
_避免_：Client, buyer, account
```

## 规则

- **有立场**：同一概念有多个词时，选最佳的作为规范术语，其余列入 _避免_。
- **定义紧凑**：1-2 句，定义"是什么"，不定义"做什么"。
- **只收本项目特有术语**：通用编程概念（超时、错误类型、工具模式）不入，即使项目大量使用。
- **自然成簇时分组**：术语自然聚簇时用子标题分组；单一领域扁平列表即可。

## 多上下文：CONTEXT-MAP.md

若项目有多个上下文（如订单、计费、履约各自独立），根目录用 `CONTEXT-MAP.md` 映射，各上下文目录内放各自的 `CONTEXT.md`：

```markdown
# Context Map

## Contexts
- [Ordering](./src/ordering/CONTEXT.md) —— 接收并跟踪客户订单
- [Billing](./src/billing/CONTEXT.md) —— 生成发票并处理付款
- [Fulfillment](./src/fulfillment/CONTEXT.md) —— 拣货并发货

## Relationships
- **Ordering → Fulfillment**：Ordering 发出 OrderPlaced 事件，Fulfillment 消费以开始拣货
- **Ordering → Billing**：Ordering 发出 OrderPlaced 事件，Billing 消费以生成发票
```

多上下文项目的系统级决策放根目录 `docs/adr/`，上下文专属决策放各自 `docs/adr/`。
