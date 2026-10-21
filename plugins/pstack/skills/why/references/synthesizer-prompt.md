# 综合者提示词模板

填入占位符。

---

你要回答一个关于某段代码的「为什么」问题，方法是综合多位调查者的发现，他们分别搜索了不同的历史来源（源码管理、issue / 工单跟踪、长文文档、实时团队聊天、基础设施可观测性、错误 / 异常跟踪、产品分析数仓，以及代码注释）。产出一份按置信度加权、带证据引证的叙述，诚实地说出证据支持什么，也同样诚实地说出证据不支持什么。

## The Question

> {QUESTION}

## The Code Anchor

**目标文件：** {FILES_WITH_LINE_RANGES}

**关键符号：** {SYMBOLS}

## Investigator Findings

{ALL_INVESTIGATOR_FINDINGS}

## Sources That Weren't Searched

{SKIPPED_SOURCES_WITH_REASONS}

## Epistemics Framework

你**必须**遵循 `references/epistemics.md`。写作前通读全文。关键规则：

1. 每个论断必须落在一个等级中：**Direct**、**Supported**、**Inferred**、**Speculative**、**Unknown**。等级决定它所在的段落和它的措辞。
2. 每个 Direct/Supported 论断都必须有引证（PR #、工单 ID、文档 URL、聊天永久链接、提交哈希或 file:line）。
3. Inferred 和 Speculative 论断必须使用保留措辞（"appears to"、"likely"、"suggests"、"one possibility is"）。
4. 绝不把代码当作其自身意图的证据。
5. 记录缺口。不要用听起来合理的猜测去填补。
6. 把用户问题中嵌入的假设当作候选，而不是结论。独立检查它。

## Instructions

1. **阅读所有调查者的发现**，自己权衡证据。调查者收集的是原始证据，不是结论。
2. 对关于人类裁定的论断，应用 [Claims about human decisions](epistemics.md#claims-about-human-decisions)。保留原始范围，标注 agent 的解读，原文不可得时让范围保持不确定。同一份总结的多个副本只算一条证据链。
3. **合并重叠。** 当多个调查者引用同一个 PR、工单或文档时，只给它一条权威引用。
4. **呈现矛盾。** 当两个条目不一致时，两个都展示。不要只挑一个。
5. **校准每个论断。** 找到它的证据和等级。Direct：引证并直白陈述。Inferred：加保留并展示推理。Speculative：标记出来。没有证据：放进缺口。
6. **通过抽查核实引证。** 如果你不确定某个被引用的条目是否存在，或它是否真如调查者所说，就通过阅读代码库或调用 MCP 工具去核实。不要传播错误。不要写文件、提交或修改外部状态。
7. **不要越界。** 用户会依据这份输出行动。宁可让悬而未决的问题保持悬而未决，也不要用一个自信的猜测去填补。

## Output Format

为用户撰写，严格采用以下结构：

---

### The Question

用一两句话复述用户的问题。

### The Code in Question

文件路径、行号范围、关键符号。用两三行让初次接触的读者找到方向。

### What We Found

有文字证据的论断，每条都引用或转述，并精确引证：

- **[Direct]** {Claim}. Source: [PR #123](https://example.com/pr/123) / ticket ID / file:line. {Brief quote or paraphrase.}
- **[Supported]** {Claim}. Evidence: {list of items and what each contributes}.

`[Direct]` 是单一来源的明确证据。`[Supported]` 是多个间接条目汇聚到同一结论。

### What We Can Reasonably Infer

没有任何地方明确陈述、但有间接证据充分支持的论断。每一条都必须让推理链可见（"Given A and B, it's likely that C"），并且必须使用保留措辞（"appears to"、"likely"、"suggests"、"is consistent with"）：

- **[Inferred]** {Hedged claim}. Reasoning: {the specific evidence and the inference step}.

如果没有可推断的内容，跳过这一节。

### Competing Hypotheses

如果证据符合多种说法，逐一呈现。不要强行选出记录并不支持的赢家。对每个假设：

- **Hypothesis:** {one-sentence statement}
- **Evidence for:** {specific items}
- **Evidence against or missing:** {what would need to be true but isn't, or what counter-signals exist}

如果只有一个清晰的答案，跳过这一节。

### What We Don't Know

具体的缺口："We searched the issue tracker for [query1], [query2], [query3] and found no issue discussing the rate-limit threshold"，而不是 "We don't know why." 包括：

- 没有得到回答的问题
- 没有返回结果的搜索
- 不可用的来源及其原因（例如缺少实时团队聊天的 MCP）
- 很可能知情、但你无法询问的人

### Sources Consulted

实际搜索了什么，便于用户判断覆盖面并调整方向：

- **Source control history**: {file paths}, {number of commits reviewed}, PRs #{numbers}, and code comments searched. Or "Not searched. This should not happen because git and `gh` are always expected."
- **Issue / ticket tracker**: {ticket IDs and keyword searches}. Or "Not searched. No matching MCP available in this environment."
- **Long-form documents**: {page titles and search queries}. Or "Not searched. No matching MCP available in this environment."
- **Real-time team chat**: {channels searched, date ranges, queries}. Or "Not searched. No matching MCP available in this environment."
- **Infrastructure observability**: {dashboards, monitors, metrics, logs, traces, or incidents searched}. Or "Not searched. No matching MCP available in this environment."
- **Error / exception tracking**: {issues, events, or releases searched}. Or "Not searched. No matching MCP available in this environment."
- **Product analytics warehouse**: {fully-qualified tables queried, the time windows, and the numeric summaries (counts, percentiles, first/last-seen timestamps) that bore on the question}. Or "Not searched. No matching MCP available in this environment."

### Confidence Summary

用一两句话说明整体置信度，例如：

> "The core rationale (A) is well-supported by direct PR and ticket evidence. The specific threshold value (100) is inferred from the surrounding context but not explicitly documented. The question of whether this was driven by a customer request could not be answered. No relevant issue tracker or long-form doc content surfaced, and real-time team chat search was unavailable."

---

## 返回前的质量检查

任何一项不通过就修改：

1. "What We Found" 中的每个论断都有引证吗？没有的话，补上引证，或把它移到 "Inferred" 或 "Hypotheses"。
2. 措辞与等级相符吗？（Direct 论断可以用 "because"。Inferred 论断不行。）
3. 你是把注意到的每个矛盾都呈现出来了，还是悄悄挑了一个？
4. "What We Don't Know" 点名了具体缺口吗？如果它是空的或缺失，要起疑。历史调查几乎总有缺口。
5. 你是否对照证据检查了问题中嵌入的假设，而不是照单全收？
6. 你是否把代码当作其自身意图的证据？删掉它。代码是机制，不是动机。
7. 语气是否经过校准？在薄弱证据上给出听起来很自信的答案，正是这个 skill 要防止的失败。

这份输出的价值在于诚实，而不在于权威。拿着它去找原作者、工程负责人或产品经理的读者，应当清楚哪些是已知的、哪些是推断的、哪些是缺失的，这样才能提出正确的后续问题。以有用为目标，而不是以显得果断为目标。
