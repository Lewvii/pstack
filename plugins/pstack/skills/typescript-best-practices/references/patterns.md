# TypeScript 模式

`SKILL.md` 中每条规则的代码示例。底层原则与语言无关。参见 **type-system-discipline** 和 **boundary-discipline** principle skills。

<a id="branded-types"></a>
## 品牌类型（Branded types）

给原始类型打上品牌，让它们无法被混用。在边界处校验一次。下游代码信任这个类型。

```ts
type AgentId = string & { readonly __brand: "AgentId" };

function isAgentId(input: string): input is AgentId {
  return isUUID(input);
}

function parseAgentId(input: string): AgentId {
  if (!isAgentId(input)) throw new Error(`Invalid agent id: ${input}`);
  return input;
}

function focusAgent(id: AgentId): void {
  /* input is trusted */
}
```

沿用 `readonly __brand: 'X'` 这种形态。不要另创新的约定。

<a id="discriminated-unions"></a>
## 可辨识联合

用字面量判别字段为变体建模。每个变体共用同一个字段名，且每个变体的取值唯一，这样不可能的组合就无法被表示出来。

```ts
// Don't. Boolean + optionals lets contradictory states exist.
type DiffState = { loading: boolean; diff?: GitDiff; error?: string };

// Do. Only valid states exist.
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

选定一个判别字段名（`kind`、`type`、`tag`）并坚持使用。

<a id="constructive-modeling"></a>
## 构造式建模

用全部合法的部件来构建类型，而不是先定义一个宽松的类型再用运行时检查去限制它。

非空，通过可变参数元组实现：

```ts
type NonEmpty<T> = [T, ...T[]];

// Don't: T[] plus a length check every caller must repeat
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}

// Do: an empty value of the type can't exist
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}
```

当传入的是普通 `T[]` 时，用一个守卫收窄一次。此后这个事实就随类型传递：

```ts
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

偶数长度，表示为成对元素：

```ts
type Pairs<T> = [T, T][];
```

时间区间，表示为起点加时长：

```ts
// Don't: a comment holds the invariant
type TimeRange = { start: Date; end: Date }; // start <= end

// Do: a negative range can't be written; derive end when needed
type TimeRange = { start: Date; durationMs: number };
```

让 `durationMs` 保持为普通 number。只有当一个原始数值可能被传到需要时长的地方时，才给它打品牌（按品牌类型一节），不要出于条件反射去做。选择让错误状态无法被构造的表示方式，再在其上暴露你需要的读取方式（`pairs.flat()`、一个 `rangeEnd()` 辅助函数）。

<a id="simplest-total-type"></a>
## 最简单的全函数类型

不要什么都加强。当对 `T[]` 的每个操作都是全函数时，就保留 `T[]`：

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0); // [] is 0, fine
```

当宽松类型迫使你在某个使用点撒谎时，再去加强。征兆是 `!`、`arr[0] as T`，以及一个 "should never happen" 的 throw：

```ts
// Don't: partiality smuggled past the compiler
function newestSession(sessions: Session[]): Session {
  return sessions.at(0)!;
}

// Do: strengthen the input; the assertion disappears
function newestSession(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

把返回值弱化为 `Session | undefined` 是另一种全函数签名。

<a id="unknown-over-any"></a>
## 用 `unknown` 而不是 `any`

外部数据一律是 `unknown`。使用前先收窄。

```ts
// Don't
function handle(input: any) {
  return input.foo.bar;
}

// Do
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    // narrowed; compiler verifies access
  }
}
```

外部来源包括 RPC 载荷、`JSON.parse`、`postMessage`、IPC、文件内容、环境变量、数据库结果。

<a id="schemas-before-hand-rolled-guards"></a>
## 先用 schema，再考虑手写守卫

在为外部数据逐个属性地手写类型守卫之前，先找找仓库里的运行时 schema 库和已有的 schema。让一个 schema 负责校验，并从它推导出 TypeScript 类型。不要同时维护一个 schema、一个重复的 interface 和一个守卫，它们会彼此漂移。

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

当失败是预期中的分支时，用 `safeParse`。如果仓库使用另一个 schema 库，就用它对应的类型推导辅助工具。不要为了一个守卫引入新的 schema 依赖。这条规则偏向代码库已经信任的那套 schema 体系。

<a id="no-as-casts"></a>
## 不要用 `as` 强转

每个 `as` 都是一次潜在的运行时崩溃。只在类型系统已经验证过该断言之后才做强转。

```ts
import { z } from "zod";

// Don't
const user = data as User;

// Don't
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data;
}

// Do
const userSchema = z.object({ id: z.string(), name: z.string() });
type User = z.infer<typeof userSchema>;

function parseUser(data: unknown): User {
  return userSchema.parse(data);
}
```

当类型先于 schema 存在时，用校验器所证明的类型去标注它。这样编译器就会拒绝一个证明得比类型少的校验器。从下面的对象中去掉 `name`，赋值就无法通过编译。

```ts
type User = { id: string; name: string };

const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

从现有代码中重构掉一个 `as` 时，先弄清 TypeScript 为什么推断不出来：

- 缺少判别字段：加上一个，改用可辨识联合。
- 源类型过宽（例如 `Record<string, unknown>`）：把它收窄。
- 无类型的边界：用拥有该形状的 schema 去解析。只在没有 schema 的地方才新增。
- 确实无法表达：使用品牌类型或 `satisfies`。

<a id="narrowing-hierarchy"></a>
## 收窄的优先级

从最好到最后手段：

1. **可辨识联合的 switch / if。** 编译器自动收窄。
2. **`in` 运算符。** `"key" in obj` 收窄到包含该 key 的变体。
3. **`typeof` / `instanceof`。** 用于原始类型和类实例。
4. **用户自定义类型守卫。** 当以上都不够时。
5. **`as` 强转。** 仅在校验之后。

```ts
function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2; // narrowed to circle
  return s.width * s.height; // narrowed to rect
}
```

<a id="type-guards"></a>
## 类型守卫

守卫必须真正验证它的断言。一个撒谎的守卫比 `as` 更糟。

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

能用判别字段收窄时，优先用它。

<a id="exhaustiveness"></a>
## 穷尽性检查

在 default 分支中，把判别对象赋给一个 `never` 类型的局部变量。

```ts
// Value-returning switch
function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}

// Void switch
function handle(s: Shape): void {
  switch (s.kind) {
    case "circle":
      drawCircle(s);
      break;
    case "rect":
      drawRect(s);
      break;
    default: {
      const _exhaustive: never = s;
      void _exhaustive;
    }
  }
}
```

有返回值的 switch 用 return 风格，语句式 switch 用 void 风格。

<a id="satisfies-over-as"></a>
## 用 `satisfies` 而不是 `as`

`satisfies` 做校验，但不会拓宽字面量类型。

```ts
// Don't. Widens, loses literal types.
const config = { theme: "dark", cols: 3 } as Config;

// Do. Validates AND preserves literal types.
const config = { theme: "dark", cols: 3 } satisfies Config;
// config.theme is "dark" (literal), not string
```

<a id="boundary-validation"></a>
## 边界校验

在数据进入的地方校验一次。在内部信任类型。参见 **boundary-discipline** principle skill。

- **线上传输格式**（proto、JSON-RPC）：用 `ignoreUnknownFields` 解析，这样向前兼容的变更不会破坏旧客户端。
- **持久化的 JSON：** 带版本的数据块，解析外包一层 try/catch。
- **不要在调用链深处重复校验。**

<a id="schema-derived-types"></a>
## 从 schema 推导的类型

当 `.proto`、OpenAPI 规范、GraphQL schema 或数据库迁移已经定义了某个形状时，从生成的类型推导，而不是重复定义。

```ts
// Don't. Duplicate shape, drifts when the schema changes.
type CheckSummary = {
  totalCount: number;
  checks: { name: string; status: string }[];
};
function renderChecks(s: CheckSummary) {
  /* ... */
}

// Do. Derive from the generated schema type.
import type { ChecksMessage } from "<generated module>";
function renderChecks(s: Pick<ChecksMessage, "totalCount" | "checks">) {
  /* ... */
}
```

在写新的 interface 之前，先考虑 `Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof`。

<a id="object-args"></a>
## 对象参数

```ts
// Don't. Swap two args, still compiles.
openFile(uri, {
  startLineNumber: 10,
  startColumn: 1,
  endLineNumber: 10,
  endColumn: 1,
});

// Do. Order-independent, self-documenting.
openFile({
  uri,
  selection: {
    startLineNumber: 10,
    startColumn: 1,
    endLineNumber: 10,
    endColumn: 1,
  },
});
```

在热路径上跳过：逐帧渲染、分词器、解析器，以及任何分配开销要紧的紧凑循环。
