---
name: why
description: "用于「X 为什么是这样设计的」「我们为什么选了 Y」、设计理由、回归问题、事后复盘，或需要数据支撑的阈值。先发现可用的 MCP，然后并行查询每一类证据（源码管理、issue 跟踪、长文文档、实时聊天、基础设施可观测性、错误跟踪、产品分析数仓），最后返回一份带出处的决策与权衡解读。运行时行为的问题用 how。"
---

# Why

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

调查代码背后的动机和意图。配套的 `how` 技能回答代码做什么、怎么工作。`why` 回答是哪些力量造就了它的形状。

下面的每次派生都对应 `pstack-models.md` 中的一个角色行，以及 [Models](#models) 中的一个默认值。把 `model` 设为该行的值，若表格或该行缺失则用默认值。当值为 `auto` 或 `inherit-parent` 时不设置 `model`。如果 `Agent` 工具拒绝某个 slug，使用默认值并说明这一点。如果它拒绝默认值，根据其错误信息选择同族中最接近的合法 slug。按任务挑选每个子代理的推理等级，参见 the **poteto-mode** skill 的 Subagents 段。

每次派生都用 `general-purpose`，并设 `readonly: false`（agent 模式）。**不要用 readonly/Ask 模式。** 它会剥掉 MCP 访问，从而使依赖 MCP 的调查者和综合者的引用抽查失效。调查者仍然不应该写任何东西。

## 工作姿态

以一个**谨慎、小心、精确的调查者**身份工作。对你「知道的」和「推断的」要诚实区分。完整的置信度框架和措辞指南见 `references/epistemics.md`。综合者必须遵守它。

## 第 1 步：理解目标与问题

**目标**通常是一段代码、一种模式、一个功能，或一个具名的设计决策。**问题**通常是设计理由、权衡、驱动它的边界情况、外部约束、死代码，或一次大范围的历史梳理。

如果目标模糊（「我们为什么这么做？」却指代不明），根据对话上下文做最佳猜测（打开的文件、最近的编辑、光标位置、刚讨论过的内容）。简要说明你的理解，这样用户能及时纠正方向，然后继续。

## 第 2 步：建立代码锚点

在派生调查者之前，先就地建立种子上下文：文件路径与行号范围、关键符号（函数、类、常量）、最近几次触及目标的提交、合并提交里的 PR 号（主题行中形如 `(#1234)` 的模式），以及任何关联的工单 ID。用 `git blame -L <start>,<end> <file>` 找最后一次触碰的提交，用 `git log --follow -p -- <file>` 获取跨重命名的完整历史与补丁，用 `git log --oneline -20 -- <file>` 查看带 PR 号的最近提交，用 `git log -1 --format=%B <commit>` 提取 PR 号。然后为任何实质性提交拉取 PR 正文和讨论：`gh pr view <number> --json title,body,author,createdAt,mergedAt,labels,closingIssuesReferences,comments,reviews`。这个锚点会传给每个调查者和综合者。

## 第 3 步：并行派生调查者

**默认做完整的并行调查。** 目标是完整的覆盖地图，而不是最小化的地图。把空结果记录下来，不要跳过搜索。

### 发现

派生调查者之前，先列出 Claude Code 环境中可用的 MCP。使用系统提示词顶部的工具列表（每个 MCP 都以 `mcp__<server>__<name>` 为前缀作为工具出现）。否则读取插件/项目里的 `.mcp.json`，或运行 `claude mcp list`。

把每个可用的 MCP 映射到一类证据：

1. 源码管理历史
2. Issue / 工单跟踪
3. 长文文档
4. 实时团队聊天
5. 基础设施可观测性
6. 错误 / 异常跟踪
7. 产品分析数仓

源码管理始终可以通过 git 和 `gh` 获得。其余六类，依据 MCP 名称、服务器说明、工具名和资源描述来归类。如果一个 MCP 可以归入多类，选择与其主要证据相符的那一类。把有歧义的情况记进覆盖地图。

为每个有匹配 MCP 的类别派生一个调查者，全部放在一条消息里并发运行，每个调查者只负责一个工具或 MCP。不要让一个 agent 覆盖多个 MCP。`model`：取 `why investigators` 行，默认值见 [Models](#models)。

每个调查者的提示词是填好的 `references/investigator-prompt.md`，再加上：
- 所选 MCP 对应的类别 playbook `references/sources/<source>.md`，由 `references/source-playbook.md` 中的示例改编而来
- **如果目标代码看起来是防御性的**（空值检查、重试逻辑、超时处理、限流、功能开关、出站守卫、OOM 处理），加上 `references/sources/incident-postmortem.md`
- 代码锚点和用户的原始问题

### 调查者名单，每个可用的证据类别一个

下面每个类别都独特地适合揭示某一种「为什么」，见「最擅长」之后所述。用它来预判能拿回什么、在某类返回空时点名缺口，以及只在罕见的、可证明无关的情况下为跳过做辩护。

1. **源码管理**（git 历史、用 `gh` 查 PR、代码注释、测试）。始终派生，这是唯一保证可得的来源。最擅长*评审期间记录下的实现期理由*。
2. **Issue / 工单跟踪**（例如 Linear、Jira、GitHub Issues、Plane、Shortcut 的 MCP）。最擅长*产品或业务上的强制驱动因素*。当「为什么」在工程之外时最强。
3. **长文文档**（例如 Notion、Confluence、Google Docs、Coda 的 MCP）。最擅长*长篇的设计理由*，在变成代码之前就写下来的那种。
4. **实时团队聊天**（例如 Slack、Discord、Microsoft Teams、Mattermost 的 MCP）。最擅长*从未落到文档里的实时讨论*。当源码管理、工单和文档的纸面痕迹都很稀薄时尤其重要。
5. **基础设施可观测性**（例如 Datadog、New Relic、Honeycomb、Grafana、Splunk 的 MCP）。最擅长*驱动了这段代码的基础设施与运行时现实*。当目标是在响应基础设施信号（超时、重试、限流、熔断）时最强。
6. **错误 / 异常跟踪**（例如 Sentry、Rollbar、Bugsnag、Airbrake 的 MCP）。最擅长*驱动了防御性或纠正性代码的具体异常与错误轨迹*。对 catch 块、空值守卫、类型检查、重试等防御手段最强。
7. **产品分析数仓**（例如 Databricks、Snowflake、BigQuery、ClickHouse、dbt、Redshift 的 MCP）。最擅长*塑造了代码的产品与数据现实*。对开关门控的代码、实验驱动的发布、数据迁移，以及「这个数字从哪来的」这类问题最强。

### 何时跳过某个调查者

只能有**明确、书面**的理由，并写进最终的 Sources Consulted。有效的理由有两种：

- **该类别在本环境中没有可用的 MCP**。把它标记为缺口而不是选择：「实时团队聊天已跳过。没有匹配的 MCP，因此无法检索对话记录。」
- **该来源可证明无关**，而不只是「大概无关」。这是很高的门槛：「错误 / 异常跟踪已跳过。目标是构建期脚本，没有运行时代码路径。」

如果你的范围评估表明目标只是一个单提交的琐碎对象，且 PR 描述已经包含完整答案，你可以在**确认**七个可用类别的搜索都会冗余**之后**直接就地作答。要明确说明这一点。这应当是罕见情况。

## 第 4 步：综合

派生一个综合者。`model`：取 `why synthesizer` 行，默认值见 [Models](#models)。它拿到：

1. 调查者的发现，包括任何空结果和任何有理由跳过的类别
2. 代码锚点
3. 用户的原始问题
4. `references/epistemics.md` 中的认知框架
5. `references/synthesizer-prompt.md` 中的综合者提示词模板

## 第 5 步：呈现

把综合者的输出呈现给用户。可以为了清晰做轻微编辑，或补充对话中的上下文，但**不要改写置信度措辞**。

## 输出格式

结构采用 `references/synthesizer-prompt.md` 中的那一套：The Question、The Code in Question、What We Found、What We Can Reasonably Infer、Competing Hypotheses、What We Don't Know、Sources Consulted、Confidence Summary。按需调整，但保持置信度的区分不被混同，Sources Consulted 保持每个调查者一行，包括没有任何返回或被跳过的那些，并写明原因。

在 Sources Consulted 之后，如果用户的 `why` 问题是要改动这段代码的前置动作，就把来龙去脉的发现转成一组 Preserve / Change / Avoid / Risk 约束，用于规划改动。

## 需要避免的常见失败模式

- **近因偏差**。认为最近的提交就是权威。当前的形状往往是许多早期决策层层累积的结果。要往回追。

## Models

Role defaults, stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in the `pstack-models.md` override sheet overrides each at runtime; `/setup-pstack` writes it and lists its path per runtime.

- why investigators: `opus`
- why synthesizer: `opus`

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
