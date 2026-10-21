# 认知框架

当证据是历史性的、零碎的、有时相互矛盾时，如何推断置信度，以及如何在告诉用户时不把它压平成虚假的确定性。代码展示的是它做什么，而不是*它为什么存在*。「为什么」存在于提交、PR、工单、文档和对话之中，而这些都不完整、有偏差，有时甚至缺失。假装不是这样，只会产出听起来很自信、实则误导用户的猜测。

<a id="claims-about-human-decisions"></a>
## 关于人类决策的论断

在调查某条规则是否允许某个动作之前，先确定拥有该动作的组件或阶段，以及这个问题是否影响当前任务。动作归别处所有，可能让这个问题变得不必要；但它不会终止一项相关的跨组件调查。

当一条人类裁定的确切范围很重要时，找到这个人原始的、可归属的原话。保留来源指针、说话人，以及足以确立范围的原始措辞。把 agent 的总结或解读标记为总结或解读，不论它有多久。仅凭转述无法确立原始指令的确切范围。如果原文不可得，就如实说明，并让该范围保持不确定；不要凭空捏造一条硬性限制或许可。

例如，"avoid that endpoint during this migration" 与 agent 的 "the user permanently forbids that endpoint" 是两个不同的论断。把有范围限定的原文和更宽泛的解读分开报告。同一解读在多份笔记中的副本构成一条相互依赖的证据链，而不是多个独立的决定。在提高置信度之前，先把它们追溯到源头。本规则针对人类裁定；普通的事实性报告不要求每个论断都附逐字引文。

<a id="confidence-tiers"></a>
## 置信度分级

输出中的每个论断都必须落在一个等级中。等级决定它所在的段落和它的措辞。

<a id="1-direct"></a>
### 1. Direct

有明确的文字引证回答了这个问题：作者实际*写下*的、说明原因的内容，而不是「代码做了 X，所以作者一定想要 X」。例如，PR 描述 "this fixes the bug where users with >1000 items couldn't paginate"，工单 "we're adding this because customer Acme requested it in their security review"，代码注释 "// clamp to 100 because the upstream API rejects larger values"，设计文档 "we chose option A over option B because we need persistence across restarts"，或作者的一条聊天消息 "switching to this approach since the old one was flaky in tests"。

措辞：自信、现在时、带引证。"This exists because X."

<a id="2-supported"></a>
### 2. Supported

多条间接证据汇聚到同一结论。没有单一来源明确陈述它，但跨来源的模式让它很可能成立。例如，PR 标题写着 "improve performance"，工单打了 "perf" 标签，周边的提交都触及同一条热路径。或者随改动一起加入的几个测试都在测极大的输入。或者作者同一周的其他 PR 都提到了同一次事故。

措辞：自信但明显是推导出来的，逐条引证。"The evidence points strongly to X: [the specific pieces]."

<a id="3-inferred"></a>
### 3. Inferred

对上下文的一种合理解读，但没有任何内容明确支持它。它是*你的解读*，不是记录中的事实。例如，PR 没说原因，但错误当时正在生产环境发生（根据事故频道的时间线），而且修复很仓促（当天合并），所以它很可能是个热修复。或者函数名暗示了重试逻辑，而它的重试次数 3 与代码库其他地方团队惯用的 3 次重试一致。

措辞：有所保留，并把推理链写明。"Given A and B, C seems likely because D."

<a id="4-speculative"></a>
### 4. Speculative

证据单薄的合理假设，其他解释同样说得通。值得提出，因为用户可能知道哪个是对的，但要标记为猜测。例如 "This might be a workaround for a browser bug that's since been fixed, but we found no contemporary evidence of that"，或 "It's possible this threshold was chosen to match an SLA commitment, but no SLA doc references it."

措辞："One possibility is X, but we have no direct evidence." 通常放在 Competing Hypotheses 中，与其他可能性并列。

<a id="5-unknown"></a>
### 5. Unknown

你查过了，但没能查明。这是一个有效且重要的结果。把它记录下来。

措辞："We searched X, Y, and Z and found no evidence of why." 要具体说明你搜了*什么*。"We couldn't find out" 远不如 "We searched the ticket tracker with keywords A and B, scanned the 6 PRs that touched this file since 2023, and grep'd the repo for string literals matching the threshold. None surfaced a rationale." 有用。

<a id="phrasing-guide"></a>
## 措辞指南

**确定性词汇**意味着 Direct 或 Supported 证据，需要紧挨着引证。绝不要把它们用于推断："because"、"the reason is"、"was designed to"（声称了作者意图）、"fixes" / "addresses" / "solves"（声称改动达成了目标）、"the team decided"（声称发生过一次团队决策）。

**保留语**表示解读，而非报告。在 What We Can Reasonably Infer 中可以大量使用："appears to"、"seems to"、"likely"、"suggests"、"is consistent with"、"one reading is"、"plausibly"、"may have been"、"the evidence points toward"。

**要避免的词：**

- "obviously"。如果真的显而易见，用户就不会问了
- "clearly"。几乎总是出现在一个并不清楚的论断之前
- "of course"。同上
- "just"（如 "it's just X for performance"）。带有轻视意味，通常掩盖了不确定性
- "I think" / "I believe"。你是在综合证据，不是在发表个人观点。改用 "the evidence suggests"。

<a id="avoid-rationalization"></a>
## 避免合理化

今天看起来「说得通」的代码，当初写下它的理由可能已不再适用，甚至在当时就是错的。不要给混乱的历史事后套上一个干净的理由。不要假设作者做了「正确」的事，再倒推为它辩护。不要假设一致的模式是有意为之，它可能只是复制粘贴。不要把缺乏证据当成否定的证据（"no one mentioned security concerns, so it must not have been a concern"）。

<a id="the-sycophancy-trap"></a>
## 迎合陷阱

用户常常在问题里嵌入一个假设："Why do we do it this way, I assume it's for performance?" 不要简单地确认它。这个猜测是调查的提示，而不是待验证的结论。把它当作多个候选之一，独立检查证据。如果证据支持它，就带着引证这么说。如果不支持，也如实说，并给出证据*确实*支持的内容。

<a id="when-evidence-contradicts"></a>
## 当证据相互矛盾时

在输出中同时呈现两个来源，并附引证。不要挑那个更符合整洁叙事的。例如，工单说 "we need this for customer X's compliance requirement"，而 PR 说 "cleaning up tech debt in this area"。两者可能都对（工单驱动了这项工作，PR 给出的是作者的表述框架），也可能有一个是错的。让用户来判断。

<a id="when-evidence-is-missing"></a>
## 当证据缺失时

诚实的「我们不知道」是最有价值的输出之一。用户由此得知答案不在显而易见的地方，需要去问人（原作者、产品负责人、团队负责人），或者可以判定这个问题不值得继续追。不标记缺口、反而用自信的猜测填上它，会实实在在地伤害用户，因为他们会依据这个猜测行动。具体地命名每个缺口：问题是什么、搜了哪些来源、在每个来源里搜了什么、找到了什么（什么都没有，或只有不相干的材料）。

<a id="before-finalizing"></a>
## 定稿之前

综合者在交付输出之前，要检查 "What We Found" 和 "What We Can Reasonably Infer" 中的每个论断。每个论断都有引证，否则移到 Inferred 或 Hypotheses。它的措辞与其等级相符。它没有把代码当作其自身意图的证据（如果有，就删除或重新归类）。输出包含 "What We Don't Know" 一节。如果完全没有提到缺口，那就很可疑。要么证据异常完整，要么有东西被掩盖了。
