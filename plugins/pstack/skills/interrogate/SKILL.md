---
name: interrogate
description: "用于 \"interrogate\"、\"adversarial review\"、\"multi-model review\"、\"challenge this\"、\"stress test this code\"、\"find blind spots\" 或 \"tear this apart\"。多个 LLM 评审者从相互独立的角度挑战改动。"
---

# Interrogate

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

为每个配置的模型派生一个评审者，对代码改动做对抗式评审。对抗信号来自模型的多样性，而不是指派的角色。交付物是一份综合后的结论。不要自动应用改动。

## 第 1 步：范围

从上下文中确定要评审什么：

- 如果用户指向了具体文件或某个 diff，就用它。
- 如果在功能分支上，运行 `git diff main...HEAD`（或相应的基线分支）获取完整变更集。
- 如果用户的消息提到了最近的工作，收集相关文件。

把 diff 或文件内容打包，连同评审者理解代码所需的周边上下文文件一起提供。

## 第 2 步：意图

在派生评审者之前，根据用户的消息、提交信息、PR 描述（如果有）以及代码，用一段清晰的话陈述意图。如果你对意图没有把握，先问用户再继续。

## 第 3 步：派生评审者

在一条消息里用 `Agent` 工具启动所有评审者，`pstack-models.md` 中 `interrogate reviewers` 行的每个条目对应一个，按条目数依次标记为 Reviewer A、B 等。如果该表格或该行缺失，使用表中的默认值。

| Subagent | Default model |
|----------|---------------|
| Reviewer A | `opus` |
| Reviewer B | `fable` |
| Reviewer C | `sonnet` |
| Reviewer D | `haiku` |

每个评审者都使用 `subagent_type: "general-purpose"`、`readonly: true`，并把 `model` 设为它配置的条目；没有配置行时用表中默认值。条目为 `auto` 或 `inherit-parent` 时省略 `model`，让该评审者运行在父模型上。

如果 `Agent` 工具拒绝某个配置条目，就让该评审者使用其所属家族在表中的默认值运行，并说明这一点。家族按模型名划分，比如 Opus、Fable 或 Sonnet。没有匹配的家族时，使用 Reviewer A 的默认值。如果它拒绝了表中的默认值，就从错误信息给出的合法 slug 中选最接近的等价项派生（优先同家族、同推理档位），并另开一个 PR 更新默认表。不要因为 slug 问题阻塞评审。绝不要把别名条目当作被拒绝的 slug，也不要对它应用上述任何一种回退。

阅读 `references/reviewer-prompt.md`，为每个评审者填写同一份模板，填入陈述的意图、diff 或文件内容、`references/rubric.md` 中的评分标准，以及 `references/code-quality-review.md` 中的代码质量视角。按任务挑选每个子代理的推理等级，参见 the **poteto-mode** skill 的 Subagents 段。

## 第 4 步：综合

结果陆续返回时，建立一幅统一的图景。解析每个评审者的发现。把以不同方式描述同一问题的发现合并，并注明每条是哪些模型提出的。两个或更多模型独立提出的发现信号最强。单个模型提出的发现也要读，但相应地降低权重。记下分歧。如果一个模型标记了某个问题，而另一个明确持相反意见，这就是下结论时的背景。

## 第 5 步：主评审判断

你是主评审者，一位务实的资深工程师，而不是中立的汇总者。完整框架见 `references/lead-judgment.md`。把每条发现归入四个类别之一，注明提出它的模型，并附一行理由。

- **Act on（需处理）**。在实际目标下影响正确性、安全性或可维护性的真实问题。它们会阻塞一个真实的 PR。
- **Consider（可考虑）**。合理，但你不确定它们是否值得现在付出处理成本。值得用户关注。
- **Noted（已记录）**。技术上成立但不可执行。依赖具体情境、属于过早优化，或在当前阶段影响很小。
- **Dismissed（驳回）**。错误、吹毛求疵或缺少上下文，附简短理由。

## 输出格式

### Intent
> [第 2 步陈述的意图段落]

### Reviewers
- Reviewer [label]: [model name], [N findings]（每个评审者一条）

### Act On
[每条：描述，哪些模型提出，为什么重要。]

### Consider
[每条：描述，哪些模型提出，取舍是什么。]

### Noted
[简短列表。]

### Dismissed
[每条附简短理由。]

### Agreement Map
[模型在哪里一致、在哪里分歧，以及这种模式说明了什么。]

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
