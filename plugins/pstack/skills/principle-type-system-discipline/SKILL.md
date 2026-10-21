---
name: principle-type-system-discipline
description: "在设计类型、审查函数签名，或用任何静态类型语言写代码时套用。让非法状态无法表达，为语义原语加 brand，在边界处解析外部数据，穷尽变体，从权威 schema 派生类型。"
user-invocable: false
---

# 类型系统纪律

让编译器拒绝不可能的状态、类型不匹配的原语，以及未处理的变体。优先让错误与特例根本不成立，而不是不断增加处理分支。TypeScript 语法见 `typescript-best-practices` skill。

- **为语义原语加 brand。** `UserId` 和 `OrderId` 底层都是字符串，但不应可以互换。给每个类型一个独立的类型，例如 TypeScript 中的 branded intersection，或其他语言中的 newtype。在创建时校验一次，下游信任类型。如果两个参数共享同一原语类型但含义不同，就为它们加 brand。
- **外部数据在解析之前都是无类型的。** RPC payload、JSON、IPC 消息、CLI 参数、配置文件、环境变量、数据库行，都是如此。每个边界都要有一个 parse 函数，把非结构化输入转成带类型的模型。验证该放在哪里，见 the **boundary-discipline** principle skill。
- **从权威 schema 派生类型。** 当 protocol buffer、OpenAPI spec、GraphQL schema、数据库迁移、design-token 文件或其他文件拥有一份形状定义时，从它派生，而不是手写一份并行的类型。见 the **encode-lessons-in-structure** principle skill。
