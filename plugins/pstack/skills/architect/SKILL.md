---
name: architect
description: "在写代码之前先勾勒类型、签名和模块结构，实现填充期间持续参与。用于 /architect、「架构一下这个」「设计这个」，或直接跳到代码会锁死错误形状的非平凡工作。"
---

# Architect

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

用 `not implemented` 函数体和伪代码勾勒类型、函数签名、类形状和模块边界。综合多个模型的视角，然后针对选定的草图填充代码。

<a id="start"></a>
## 开始

开工前先开一个 todolist，每个阶段一条。

1. Ground（打底）
2. Sketch（画草图）
3. Agree（达成一致）
4. Implement（实现）
5. Scrap（推倒重来）

<a id="phase-a-ground-the-problem"></a>
## 阶段 A：为问题打底

对新代码会触及的每一个系统建立真实的心智模型。对相关子系统运行 the **how** skill。能说出文件名不算打底。产出 `how` 要求的架构说明。如果设计重新定义了归属或分层，还要对现有形状运行 the **why** skill，让背后的理由成为约束而不是猜测。

只有当工作确实是全新的绿地、没有周边系统需要集成时，才跳过阶段 A。

<a id="phase-b-sketch"></a>
## 阶段 B：画草图

带着设计草图任务和阶段 A 的打底产物运行 the **arena** skill。把 `references/runner-prompt.md` 作为每个 runner 的提示词传入。

runner 取自 `pstack-models.md` 里的 `architect runners` 行，代替 `arena runners` 行。如果表格或该行缺失，使用 [Models](#models) 中的默认值。别名和被拒绝的条目的处理遵循 **arena** 技能阶段 A 中的 runner 规则。

设计两次。综合之前至少要有两个结构上不同的候选，即使第一个看起来已经够用。这是把 **exhaust-the-design-space** principle skill 落到实处。要整体形状的备选，而不是同一种形状里的点状修补。

综合之前，用 [`references/design-red-flags.md`](references/design-red-flags.md) 逐一筛查每个候选。假设下一位贡献者是 agent，它只看得到自己打开的文件、抄最近的例子、走能编译的最短路径。优先选择那种「改动从单个文件看是对的」恰好对整个仓库也成立的设计。

按接口深度比较可行的候选。优先选择把更多复杂度藏在更小、更简单的公共表面之后的设计。丰富的接口可以通过集中能力而不是把它散落到各层，让调用链保持简短。

Arena 返回一个综合后的设计包。它的综合决策填入 rationale 的「Synthesis decision」段。

## 阶段 C：达成一致（可选）

默认：直接用综合后的设计进入实现。没有人工检查点。

当调用方明确提出时才启用检查点：「/architect with checkpoint」「实现前停下来给我看」之类。此时把综合后的设计摆出来，暂停等待确认。

无论如何，综合结果都可以作为独立提交交付，也就是 **foundational-thinking** principle skill 中的「先搭脚手架」模式。按 **outcome-oriented-execution** principle skill，填充期间有计划、有范围的破损是可以接受的。若要在实现前对设计施加对抗性压力，对综合后的草图运行 the **interrogate** skill。

如果人对形状提出异议（无论在检查点还是事后），把它当作阶段 A 的证据。重新打底并重跑阶段 B，然后再写更多代码。

## 阶段 D：按草图实现

用代码替换 `not implemented` 函数体，用逻辑替换伪代码。综合后的草图就是契约。偏离它是需要摆到台面上的信号，而不是可以默默吞下的摩擦。如果某个函数需要一个草图没预料到的参数，问清楚是草图错了、需求漏了，还是实现越界了。

在关闭每个实现单元、或把它的契约交给下一个 worker 之前，把已接受的偏离与保存的行为契约、接口和责任划分对照一遍。一旦相应的负责人或评审流程接受了某项改动，就更新现有草图和 rationale 中受影响的部分，并在那里记录接受来源。从抛出错误改为返回拒绝，必须在下一个单元开始前更新结果契约。

对于不破坏共享契约的局部偏离，记录决定以及设计为何仍然成立；改一个局部变量名不必重写架构。让未被接受的改动和尚未解决的分歧保持可见。当下一个 worker 会收到自相矛盾的指令时，不要声称该单元已经对齐；也不要仅仅为了替已实现的东西辩护而改规格。使用已有的设计产物，而不是每个单元各记一份平行记录。反复出现的结构性偏离仍然会触发阶段 E。

## 阶段 E：架构不对就推倒

如果实现持续产生草图无法吸收的摩擦，就把草图扔掉。不要在一个错误的设计上打补丁，参见 **redesign-from-first-principles** 和 **fix-root-causes** principle skill。

信号是*模式*，不是单个实例。迹象包括：

- 同一种形状的变通反复出现在互不相关的代码里。
- 多个互不相关的边界情况都需要专门的特例分支。
- 类型需要逃生舱（`any`、类型断言、实际上总会赋值的可选字段）才能编译。
- 草图说状态并不共享，你却条件反射地觉得「我们需要一把锁」。
- 调用者必须了解抽象的内部规则才能使用它。
- 实现中出现两处或更多形状相同的独立阶段 D 偏离。

用判断力。几个边界情况不足以否定一个架构。有些问题本来就复杂。数据里的复杂度不是设计里的复杂度。

推倒时：

1. 对已经建好的部分重新运行 the **how** skill。
2. 按 redesign-from-first-principles，假设新约束是第一天就存在的假设来重新设计。
3. 按 the **subtract-before-you-add** principle skill，先减后加。新草图在长大之前应当比旧草图更小。
4. 回到阶段 B，重跑 arena。

## 产物

先写调用方的用法，再由它导出类型草图。小改动是一个文件，含新类型和签名。较大的工作给模块图加类型定义。rationale 一并交付，按 `references/rationale-template.md` 组织，包含用法草图和综合决策。

## Models

Role defaults, stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in the `pstack-models.md` override sheet overrides each at runtime; `/setup-pstack` writes it and lists its path per runtime.

- architect runners: `opus`, `fable`, `sonnet`, `haiku`

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
