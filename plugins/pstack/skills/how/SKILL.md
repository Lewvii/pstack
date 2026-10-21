---
name: how
description: "用于「X 是怎么工作的」、改动前的代码走查，以及位置、归属、分层类问题（「这个该放哪」「哪个包负责这个」「这是对的那一层吗」）。讲解子系统架构、运行时流程、上手心智模型。动机方面的问题用 why。"
---

# How

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

探索代码库，回答「X 是怎么工作的？」这类问题。产出达到资深工程师初接手某个子系统时所需层次的架构说明：足够建立起可用的心智模型，又不至于读起来像带注释的源码。

每次派生都是用 `subagent_type: "general-purpose"` 和 `readonly: true` 的 `Agent` 子代理。探索者使用 `pstack-models.md` 中的 `how explorer` 行（默认见 [Models](#models)），每个讲解者使用 `how explainer` 行（默认见 [Models](#models)）。把 `model` 设为该行的值，若表格或该行缺失则用默认值。当值为 `auto` 或 `inherit-parent` 时不设置 `model`。如果 `Agent` 工具拒绝某个 slug，使用默认值并说明这一点。如果它拒绝默认值，根据其错误信息选择同族中最接近的合法 slug。按任务挑选每个子代理的推理等级，参见 the **poteto-mode** skill 的 Subagents 段。

## 1. 评估复杂度

如果范围有歧义，先说出你的理解再去探索。用户可以纠正方向。

- **简单**（单个模块、一个小工具，或诸如「函数 X 是怎么工作的」这类窄问题）：不要探索者。派生一个讲解者，在一次执行中完成探索和讲解，提示词由 `references/explainer-prompt.md` 去掉探索者发现段构成。跳到第 4 步。
- **复杂**（跨多个文件或服务的子系统、横切功能、完整的架构概览）：进入第 2 步。

拿不准时，走简单路径。

## 2. 探索（仅复杂情形）

把问题拆成 2 到 4 个角度，每个角度是子系统的一个独立切片。在一条消息里派生所有探索者，每个都带上 `references/explorer-prompt.md` 并填好各自的角度。

## 3. 综合（仅复杂情形）

所有探索者返回后，派生一个讲解者，传入 `references/explainer-prompt.md` 和每个探索者的发现。

## 4. 呈现

呈现讲解者的输出。为了清晰或补充对话中的上下文做轻微编辑可以。不要大幅重写。

## 输出格式

讲解使用 `references/explainer-prompt.md` 中定义的各段，不适用的略去：Overview、Key Concepts、How It Works、Where Things Live、Gotchas。

## Models

Role defaults, stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in the `pstack-models.md` override sheet overrides each at runtime; `/setup-pstack` writes it and lists its path per runtime.

- how explorer: `opus`
- how explainer: `opus`

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
