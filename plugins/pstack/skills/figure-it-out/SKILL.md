---
name: figure-it-out
description: "在没有更具体的 playbook 适用时，设计一份可审计的 playbook：大型迁移、有野心的多部分改动，或需要人类离开一段时间后回来审阅的工作。按任务调整严谨度，运行假设循环，并记录决策。用于 /figure-it-out 或 'figure it out'。"
---

# Figure it out

当任务匹配不到任何 playbook 时，就设计一份。写任何代码之前的交付物是工作流本身：一系列阶段，按任务调整严谨度，运用科学方法，并留下一条人类离开后回来也能审计的决策记录。

## 开始

打开一个 todolist，第一项是阅读 the **poteto-mode** skill 的 Principles 段。然后把下面的阶段作为 todo 加进去。

## 阶段 A：界定

先打底，再承诺。在你能说清以下内容之前，不要开始执行：

- 完成的定义，写成一个可证伪的判定条件（**prove-it-works** principle skill）。
- 范围，量化表述：大致的单元数和工作量，以及打底过程中暴露出的阻碍。
- 严谨度等级，宁高勿低。单向门和影响面大的工作要更严谨，可逆的低风险步骤可以放宽。严谨度体现在关卡和产物上，而不是"更努力一点"。

在投入一次长时间运行之前，先展示界定结果和取舍。可逆的工作直接推进（**never-block-on-the-human** principle skill），但一次长达数小时的运行值得设一个检查点。

## 阶段 B：设计工作流

拆分成原子的、可独立落地的单元。先排风险最大的未知项。脚手架和验证先于功能（**foundational-thinking** principle skill）。

- 在动手之前先搭好验证设施，并从改动前的状态采集基线，让检查读起来是"旧值对比新值"。
- 对于单向门的设计决策，运行 the **architect** skill（它会运行 **arena**）。对于形状已经明确的机械性工作，跳过它。对已经定下来的设计再跑一次 arena 属于过度设计（**laziness-protocol** principle skill）。
- 决定哪些部分要并行展开。只沿着接缝并行，并给每个 worker 自己的 worktree 或分支（**separate-before-serializing-shared-state** principle skill）。不要过度展开。按任务挑选每个子代理的推理等级，参见 the **poteto-mode** skill 的 Subagents 段。
- 把设计好的阶段清单写下来。这份清单就是人类要审阅的内容。

然后执行这个设计。把其中的步骤作为具体条目加进 todolist，放在阶段 C 和阶段 D 两项之间。每一步都按阶段 C 的循环纪律执行，并把阶段 D 的日志贯穿其中，每落地一步记一行，而不是把整条记录留到最后再写。

## 阶段 C：运行循环

每个单元都是一次实验。陈述假设，做最小的改动，在真实产物上对照判定条件测量，有进展就保留，没有就回退。运用 **sequence-verifiable-units** principle skill，每个单元验证完再开始下一个，而不是把检查攒到最后一起做。

- 通过检查产物来验证，绝不依赖自我报告。当某项通过得太容易时，先怀疑观察方法，再怀疑系统。
- 委派出去的工作要配一个评判者。如果 worker 钻了关卡的空子，就重置并加固约定。如果关卡本身错了，用一个独立的改动修正关卡，而不是绕开它。
- 结论只有 VERIFIED、NOT VERIFIED 或 INCONCLUSIVE 三种。INCONCLUSIVE 不等于通过。不要隐藏负面结果。

## 阶段 D：保留审计记录

通过 the **show-me-your-work** skill 记录这次运行。figure-it-out 的工作通常足够有野心，值得把记录提交进去，让审阅者能在 PR 里读到。记录加上 diff，就是让人类回来后能信任这份工作的依据。

## 阶段 E：验证并交还

在真实产品上，而不仅仅是在验证设施上，对照阶段 A 的判定条件检查整体结果。把任何反复出现的纠正固化为一个关卡、一条 lint 规则、一项检查或一个脚本（**encode-lessons-in-structure** principle skill）。

**回复：** 你设计的 playbook，严谨度等级及原因，决策记录的路径，哪些已对照判定条件验证，以及哪些仍未解决。
