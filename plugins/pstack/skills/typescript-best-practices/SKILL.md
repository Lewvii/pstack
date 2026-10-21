---
name: typescript-best-practices
description: TypeScript 最佳实践。在阅读或编辑任何 .ts 或 .tsx 文件时使用。
paths: ["**/*.ts", "**/*.tsx"]
---

# TypeScript best practices

先应用 the **type-system-discipline** principle skill。

| 规则 | 摘要 |
|------|---------|
| 可辨识联合 | 用 `kind` 字面量判别字段为各种变体建模，使不可能的状态无法表示。不要用可选字段拼成的袋子。 |
| 品牌类型 | 用 `& { readonly __brand: "X" }` 给原始类型打上品牌，使它们不会互相混淆。在边界处校验一次。品牌来自可验证的类型守卫（`input is AgentId`），绝不来自 `as`。 |
| 构造式建模 | 把形状建得让非法值根本无法构造出来。用 `[T, ...T[]]` 表示非空，用 `[T, T][]` 表示偶数长度，用 `start` 加 `duration` 表示区间。不要运行时守卫，也不要指望细化类型。 |
| 最简单的不全类型 | 只要对其的操作都是完备的，就保持 `T[]`。仅当宽松类型迫使你写 `!`、cast 或 "should never happen" 抛出时，才收紧为 `NonEmpty<T>`。 |
| `unknown` 优于 `any` | 外部数据是 `unknown`。 |
| schema 先于守卫 | 在手工逐属性写类型守卫之前，使用仓库的运行时 schema 库，并从 schema 推断类型，例如 `z.infer`。 |
| 不用 `as` 断言 | 每个 `as` 都是一次迟早发生的运行时崩溃。只在验证之后才 cast。 |
| 收窄层级 | 判别字段 switch > `in` 运算符 > `typeof`/`instanceof` > 用户定义类型守卫 > `as`。 |
| 类型守卫 | 必须验证其声称。说谎的守卫比 `as` 更糟，因为 bug 藏在一个声称安全的名称背后。命名为 `isX` 或 `hasX`。 |
| 穷尽性 | 在 default 分支内联 `const _exhaustive: never = x;`，使新增变体时编译器报错。 |
| `satisfies` 优于 `as` | 校验值而不放宽字面量类型。 |
| 边界校验 | 在数据进入的地方解析成具名的领域类型。`Record<string, unknown>`（无论怎么拼写）止步于那次解析。内部信任类型。见 the **boundary-discipline** principle skill。 |
| 由 schema 派生类型 | 在声明新接口之前，先用 `Pick`/`Omit`/`Parameters`/`ReturnType`/`Awaited`/`typeof`。 |
| 对象参数 | 传对象而非位置参数，让参数顺序自解释。热路径上跳过（逐帧渲染、分词器、解析器）。 |
| 真实测试 | 能跑的就不要 mock。优先使用框架真实的测试原语并做泄漏／可释放性检查，在运行中的构建里验证 UI。只 mock 本地跑不起来的东西。 |
| 结构化遥测 | 优先使用结构化日志诊断，带足上下文，能从 id 调试。发布代码中不要 `console.log`。 |

示例见 `references/patterns.md`。
