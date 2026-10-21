---
name: arena
description: "针对同一任务并行生成 N 个候选，选出一个基底，再把落选者最强的部分嫁接到基底里。用于 /arena、“arena this”、“throw it in the arena”，或当对一件非平凡产物只尝试一次就会锁死错误形态时。"
---

# Arena

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

在启动任何东西之前，先开一个待办清单，每个阶段一条（Frame、Fan out、Cross-judge、Pick、Graft、Verify）。

## 阶段 A：定框

每个候选拿到的是同一份提示词，所以提示词就是契约。

1. 写清每个候选要产出的产物是什么。
2. 推导评分表：针对*这一*任务，给出 3-6 条具体、可打分的成功标准。它是阶段 D 中挑选者的工具。候选只看得到任务本身。
3. 选择执行者。使用 `pstack-models.md` 里的 `arena runners` 行。如果该表或该行缺失，就按 [Models](#models) 中的默认值各跑一个。这一行或交叉评判行里出现 `auto` 或 `inherit-parent`，表示使用父模型，此时省略 `model`。如果 `Agent` 工具拒绝某个已配置的条目，就让该席位改跑其所属家族的默认模型，并说明这一点。家族按模型名划分，例如 Opus、Fable、Sonnet。没有对得上的家族时，使用 [Models](#models) 中的单角色默认值。如果它拒绝某个默认值，就从报错信息里取同家族最近的合法 slug。竞技场要覆盖多个设计方向时，多起几个执行者。当工作受限于生成能力而不是判断力时，用同一个模型跑 N 次。按 **poteto-mode** 技能的 Subagents 一节，为每个子代理的活挑选推理档位。
4. 按 **separate-before-serializing-shared-state** principle skill 的要求，让每个候选有自己的输出位置（尽量用 git worktree，否则用 `/tmp/arena-<slug>/candidate-<n>/`）。

## 阶段 B：铺开

在一条消息里用 `run_in_background: true` 启动全部 N 个子代理，每个都带上任务、共享背景材料的路径、自己的输出路径，并要求它产出产物，外加一段简短说明：它考虑过哪些替代方案、否决了什么。

如果某个候选没有产出，就用 N-1 继续，并记录这次掉队。

## 阶段 C：交叉评判

等阶段 B 的候选全部完成后，从 `pstack-models.md` 的 `arena cross-judge pool` 行里选一个模型。如果该表或该行缺失，就从 [Models](#models) 中的执行者默认值里选。优先选与父模型不同的模型家族。在该模型上起一个只读的评判子代理。它拿到评分表和按标签指代的候选，逐条打分，并给出推荐的基底及理由。它与你阶段 D 的阅读并行进行。候选还在写的时候不要起它。

## 阶段 D：选定基底

选之前把每个候选从头读到尾。按评分表逐条打分，不要凭整体感觉，再和交叉评判的结果对照。基底一致，说明这个选择站得住。不一致，说明你们之中有一个带偏见，或者评分表本身含糊。做决定前把两边的理由都读完。

选那个后续维护者最容易扩展、又不会破坏不变量的基底。两边难分高下时，按 Laziness Protocol 选边界更干净或 API 更小的那个。

## 阶段 E：嫁接

把每个落选候选再过一遍，挑出值得移植进基底的东西，通常每个候选一两处，而不是大半。按 **redesign-from-first-principles** principle skill 的要求，逐处手工并入，让结果在同一套心智模型下保持自洽。不要机械粘贴。

如果几个候选收敛到同一种形态，那是很强的共识。记录它，直接交付共识形态，不需要嫁接。如果它们分歧极大，说明阶段 A 定义得不够。重新定框再跑一遍，而不是把分歧平均掉。

## 阶段 F：验证

按 **prove-it-works** principle skill 的要求，像对待任何产出一样验证合成后的产物。如果验证发现了竞技场漏掉的问题，要么阶段 A 错了（重新定框再跑），要么某个候选抓到过而你漏了嫁接（回到阶段 E）。不要把它糊过去。

## 产出

一件合成后的产物，旁边配一份简短的综合作注：基底是哪个、为什么，交叉评判的结论，每一处嫁接及其来源候选，否决了什么、为什么，有哪些收敛或掉队，以及验证结果。

## Models

Role defaults, stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in the `pstack-models.md` override sheet overrides each at runtime; `/setup-pstack` writes it and lists its path per runtime.

- arena runners: `opus`, `fable`, `sonnet`, `haiku`
- arena cross-judge pool: `opus`, `fable`, `sonnet`, `haiku`

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
