---
name: poteto-help
description: 引导用户完成 pstack 的设置、了解 /poteto-mode，并为某个任务挑选合适的技能、playbook 或原则。输入 /poteto-help 并附上问题。
---

# Poteto help

回答用户关于 pstack 的问题，给他们一段可以直接发送的提示，并链接答案所依据的文件。对于求助问题，不要直接开始干活。用户问的是怎么做，而 pstack 运行会消耗真实的 token，所以把提示交给他们自己发送。

像 "use pstack to fix this bug" 这样要求做事的消息，不是求助问题。阅读 [`poteto-mode`](../poteto-mode/SKILL.md)，并在其下完成工作。

本文件把问题映射到已安装的技能、playbook 和平台映射，它们才是答案的归属地。回答之前先读对应的来源。来源中的说明优先于本映射和任何上游指南。上游指南描述的是 Cursor，可能与这个移植版本不同。本地链接指向已安装的插件，因此给用户的是文件的公开副本：`https://github.com/michael-denyer/pstack-claude/blob/main/plugins/pstack/`，后面接它相对于插件根目录的路径。

## 弄清他们需要什么

从消息和对话中推断需求。已经说清情境的，例如 "which skill reviews a PR?"，直接跳到对应小节。如果需求仍不明确，就用下面这些选项问一个多选题，然后只回答他们选中的那一节：

- Get set up
- 用 `/poteto-mode` 启动一个任务
- 为某个情境挑选技能
- 修复一次出错的运行
- 按我的习惯定制 pstack

检查会改变答案的状态，只在它确实改变答案时才提及：

- 对于模型或路由问题，先确定运行时和安装类型，然后阅读 [`setup-pstack`](../setup-pstack/SKILL.md#other-runtimes)，了解覆盖表的位置以及该运行时如何加载它。在说明适用默认值还是覆盖值之前，先检查那份配置。
- 项目里没有 `verify` 或 `verify-*` 技能、也没有其他应用 harness，意味着 agent 没有脚本化的方式驱动应用。当问题涉及证明某项改动能工作时，提及 `/create-verification-skill`。
- 项目里没有形式化验证技能，意味着没有任何性质被检查到测试覆盖之外。当问题涉及竞态、不变量或其他测试很少触及的情况时，提及 `/create-formal-verification-skill`。如果已有一个，而它覆盖的代码在其记录为最后同步的 commit 之后发生了改动，且问题涉及那段代码，就提及 `/maintain-formal-verification-skill`。

当模型覆盖表缺失且这确实重要时，询问用户是否想现在为每个角色选模型并定一个默认强度。它重要的情况是：用户是新手、问题关于设置或成本、或者答案取决于运行哪些模型。每次聊天最多问一次。如果需求也不明确，就把两个问题一起问。提供两个选项：

- 现在：把 `/setup-pstack` 交给他们输入，同时也回答他们的问题。
- 稍后：回答他们的问题，并加一句说明：在他们运行 `/setup-pstack` 之前，每个角色都保持默认模型。

<a id="get-set-up"></a>

## Get set up

1. 确定用户的运行时，以及他们想要原生插件/包还是仅安装技能。阅读 [README](https://github.com/michael-denyer/pstack-claude/blob/main/README.md#install) 或[共享安装参考](https://github.com/michael-denyer/pstack-claude/blob/main/docs/reference.md#shared-skills-installation)中对应的小节，然后给出匹配的安装命令。
2. 在解释模型选择、强度、覆盖表加载或自动路由之前，阅读 [`/setup-pstack`](../setup-pstack/SKILL.md)，包括其中的 Other runtimes 表格。对于内置的路由 hook 或扩展，使用当前运行时覆盖表里持久的 `session hook: off` 设置来关闭路由。在模型表之后，它会各提供一次机会，为项目生成验证技能和形式化验证技能，除非项目已经有其中一个。
3. 按 [`references/prompting.md`](references/prompting.md) 提供一段首个任务提示，包含一个目标和一项能通过或失败的检查。使用[斜杠命令参考](https://github.com/michael-denyer/pstack-claude/blob/main/docs/reference.md#slash-commands)中当前运行时的调用语法。

在给出运行时特定的说明或改写配方之前，若适用，先阅读 [Codex 映射](../poteto-mode/references/codex-tools.md)或 [Pi 映射](../poteto-mode/references/pi-tools.md)。它们的路由小节涵盖 hook 信任、扩展加载和仅安装技能的部署。对于其他运行时，使用[运行时支持参考](https://github.com/michael-denyer/pstack-claude/blob/main/docs/reference.md#runtime-support)。

如果担心成本，说明 token 花在哪里以及如何少花。pstack 会在 subagent 和评审面板上多花 token。重新运行 `/setup-pstack`，选择更低的强度或更便宜的模型。设为 `auto` 或 `inherit-parent` 的角色运行在聊天的模型上，当聊天本身跑在更便宜的模型上时，这样更省钱。面板列表越短，运行的 subagent 越少，每个条目一个。把 `/poteto-mode` 留给需要严谨性的工作。

## 用 `/poteto-mode` 启动一个任务

`/poteto-mode` 把任务匹配到某个 playbook，把该 playbook 的步骤复制进 todo 列表，并按步骤需要运行其他技能。它跳过的步骤会以 `skip: <reason>` 留在列表里。好的提示会说明目标以及如何判断完成。它不列出技能，因为手写的序列往往会丢掉或打乱 playbook 会保留的步骤。在帮你措辞之前，先读 [`references/prompting.md`](references/prompting.md)。

关于后续任务是否会自动路由，使用 [Get set up](#get-set-up) 下的设置与运行时来源。在承诺之前，先确认该安装已启用路由。没有自动路由时，每个新任务都要调用 `poteto-mode`。在对话中途，"new task" 会让模式匹配一份全新的 playbook。关于 subagent 默认值，阅读 [poteto-mode 的 Subagents 一节](../poteto-mode/SKILL.md#subagents)和当前运行时的映射。

## 挑选技能

默认答案是 `/poteto-mode`，它在步骤需要时会运行大多数其他技能。当用户想要比 playbook 提供的更多或更少的东西时，直接点名某个技能。推荐之前先读该技能，并给一个示例提示。

| 用户想要 | 技能 |
|---|---|
| 以严谨方式完成任何非平凡任务 | [`/poteto-mode`](../poteto-mode/SKILL.md) |
| 在开始前确认 agent 理解了目标和问题 | [`/in-your-own-words`](../in-your-own-words/SKILL.md) |
| 在开始前把任务瞄得更高：更专业、更聚焦结果、不过度规定 | [`/sharpen`](../sharpen/SKILL.md) |
| 了解代码现在如何运作，或新代码应该放在哪 | [`/how`](../how/SKILL.md) |
| 了解代码为何是这个形状，或某个数字从哪来 | [`/why`](../why/SKILL.md) |
| 以平实的方式理解一处改动或子系统 | [`/teach`](../teach/SKILL.md) |
| 跟上自己最近在某个主题上的工作 | [`/recall`](../recall/SKILL.md) |
| 用平实语言和小图示回顾这个聊天自上次发言以来的变化，列出每个等你处理的事项，且仅凭回复就能决定 | [`/ketchup`](../ketchup/SKILL.md) |
| 了解一个小 diff 可能在自身之外破坏什么 | [`/blast-radius`](../blast-radius/SKILL.md) |
| 在写跨函数边界的代码之前敲定类型和模块形状 | [`/architect`](../architect/SKILL.md) |
| 对同一份 brief 做多次尝试，合并成最好的一份 | [`/arena`](../arena/SKILL.md) |
| 对切片并行检查，或让 worker 竞赛 | [`/swarm`](../swarm/SKILL.md) |
| 让不同模型评审一个 diff 并试图击破它 | [`/interrogate`](../interrogate/SKILL.md) |
| 在有便宜的本地测试时以测试先行的方式修 bug | [`/tdd`](../tdd/SKILL.md) |
| 对 `.ts` 或 `.tsx` 工作应用 TypeScript 规则 | [`/typescript-best-practices`](../typescript-best-practices/SKILL.md) |
| 评审前剥离注释，使用没有写这些注释的 reviewer | [`/no-comments`](../no-comments/SKILL.md) |
| 清除散文中的 AI 痕迹 | [`/unslop`](../unslop/SKILL.md) |
| 按标准撰写文档、RFC、README、PR 描述或 commit message | [`/technical-writing`](../technical-writing/SKILL.md) |
| 用平实语言再听一遍上一条回复 | [`/bro`](../bro/SKILL.md) |
| 给 agent 一个脚本化方式驱动应用并证明行为 | [`/create-verification-skill`](../create-verification-skill/SKILL.md) |
| 让验证技能及其功能地图重新与应用对齐 | [`/maintain-verification-skill`](../maintain-verification-skill/SKILL.md) |
| 形式化验证竞态、不变量或其他测试很少触及的性质 | [`/create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md) |
| 在其覆盖的代码变更后让形式化验证重新对齐，并判断失败是漂移还是 bug | [`/maintain-formal-verification-skill`](../maintain-formal-verification-skill/SKILL.md) |
| 在报告或依据某个性能数字行动之前先审查它 | [`/benchmark-checklist`](../benchmark-checklist/SKILL.md) |
| 执行一项大型或跨领域的改动，或在离开一阵后回来评审的改动 | [`/figure-it-out`](../figure-it-out/SKILL.md) |
| 在一次运行中保留决策日志，并在事后复核 | [`/show-me-your-work`](../show-me-your-work/SKILL.md) |
| 为每个角色挑选模型并设定默认强度 | [`/setup-pstack`](../setup-pstack/SKILL.md) |
| 把自己的工作习惯变成个人模式技能 | [`/automate-me`](../automate-me/SKILL.md) |
| 把完成的任务教给你的东西变成技能修改 | [`/reflect`](../reflect/SKILL.md) |
| 阻止 agent 在这个仓库里重复同样的错误 | [`/correct`](../correct/SKILL.md) |
| 搞清楚 pstack 怎么用 | `/poteto-help` |

如果本文件旁边某个技能目录不在表中，读它的 frontmatter，按其 description 路由。`principle-*` 目录在下文的原则部分介绍。

容易混淆的情况：

- `/how` 解释代码做了什么。`/why` 解释背后的原因。`/teach` 运行其中之一或两者，并平实地解释结果。
- `/arena` 给每个 worker 相同的 brief 并合并最好的部分。`/swarm` 把工作切成片或让 worker 竞赛，返回一份报告。
- `/architect` 在敲定设计后立刻实现。加上 "with checkpoint" 可在写代码前先评审设计。
- `/interrogate` 评审 diff。`/blast-radius` 寻找 diff 之外的破坏，并证明那个让改动安全的事实。
- `/recall` 在最近的聊天之间重建上下文。`/ketchup` 只覆盖这个聊天自用户上条消息以来的内容。恢复某个特定聊天或分支，是 Session pickup playbook。
- `/figure-it-out` 设计一次严谨的运行。Orchestrate playbook 运行跨越数天和许多 PR 的程序。Autonomous run playbook 驱动单个任务到某个完成条件。

不在 pstack 中：

- Cursor 的 driver 技能没有移植。poteto-mode 的 Non-negotiables 中的 driver 策略列出了 Claude Code 的对应项：`run` skill 和项目的 `verify` skill。
- `/loop` 是 Claude Code 内置技能，而 the **plugin-dev:skill-development** skill 是 Claude Code 的技能编写指南。
- pstack 没有 orchestrate 技能。Orchestrate 是 `/poteto-mode` 的一个 playbook。如果斜杠菜单显示 orchestrate 命令，那是另一个插件提供的。

## Playbook 与原则

Playbook 是 `/poteto-mode` 内部的步骤列表，不是技能，因此没有斜杠命令。在 `/poteto-mode` 内部，描述任务即可选中一个，而下面这些说法会直接点名一个：

- "babysit this pr" 或 "check on pr 123" 运行 Babysit。它把 PR 推进到可合并就停下。除非用户要求合并、落地或发布，否则它不会合并。
- "land the stack" 运行 Shipping。
- "take over this branch" 运行 Session pickup。
- "pause safely" 运行 Pause safely。
- "full autopilot on this queue" 运行 [Autopilot-full](../poteto-mode/playbooks/autopilot-full.md)。"stack them, don't ship" 运行 [Autopilot-stack](../poteto-mode/playbooks/autopilot-stack.md)。在说明什么会在无人值守下运行之前，先读所选 playbook 的合并规则。
- "run the eval playbook" 运行 Eval。

不使用 `/poteto-mode` 时，像 "babysit this pr" 这样的说法可能会启动 pstack 独立的 [`/babysit`](../babysit/SKILL.md) 技能。 [Playbooks 一节](../poteto-mode/SKILL.md#playbooks)列出了每个 playbook 及其适用时机。移植版本的 PR 工作流见 [Babysit](../poteto-mode/playbooks/babysit.md) 或 [Shipping](../poteto-mode/playbooks/shipping.md)。

pstack 没有规划技能。Claude Code 的 plan mode 可与之配合使用。对于跨越多个阶段或堆叠 PR 的工作，向 `/poteto-mode` 要一份计划会运行 [Multi-phase plan playbook](../poteto-mode/playbooks/multi-phase-plan.md)，它会写出计划而不实现它。对于设计问题，Prototype playbook 或 `/architect` 会先在代码中敲定。

原则是单条规则的技能，`/poteto-mode` 会读取它们并在回复中引用。用户很少直接调用。他们改为用名字来引导，例如 "apply prove it works. show me the real output." 在 Claude Code 上，`principle-*` 叶子技能在斜杠菜单中隐藏，`/poteto-mode` 按路径读取它们；其他运行时可能会在技能选择器中列出。 [Principles 索引](../poteto-mode/SKILL.md#principles)列出了它们。

## 修复一次出错的运行

| 症状 | 修复 |
|---|---|
| 几轮之后模式不再生效 | 通过 Get set up 下的来源检查当前运行时的路由配置。没有自动路由时，每个任务都要调用 `poteto-mode`。 |
| 一个问题被当成上一个任务的下一步 | 说 "new task"，或者说这一轮不需要该模式。 |
| 新的模型选择没有起作用 | 阅读 [setup-pstack](../setup-pstack/SKILL.md#other-runtimes) 和当前运行时的映射，然后检查其覆盖表位置和加载方式。 |
| 运行成本超出预期 | 见 Get set up 下的成本段落。 |
| 某个技能没有自动加载 | 阅读它的 description 和当前运行时的技能加载说明。需要时显式调用它；`/poteto-mode` 并不会运行每一个技能。 |
| 并行 agent 互相覆盖 | 给每个 agent 各自的 worktree。 |
| 通宵运行有活动却没完成任何事 | 阅读 [Autonomous run](../poteto-mode/playbooks/autonomous-run.md) 和当前运行时的唤醒机制。对于队列，阅读上文选定的 Autopilot playbook。给这次运行一个能通过或失败的检查。 |
| 回复仅凭构建通过就声称成功 | 要求给出真实的命令、流程、存储值或 profile。这就是 prove-it-works 原则。 |

对于发生漂移的运行，[`references/prompting.md`](references/prompting.md) 有一行式的引导，[`references/recipes.md`](references/recipes.md) 有可改写的提示。

## 按我的习惯定制 pstack

- [`/automate-me`](../automate-me/SKILL.md) 从用户自己的历史中起草一个个人模式技能，与 `/poteto-mode` 配合使用。
- [`/reflect`](../reflect/SKILL.md) 在会话结束后把其中的经验变成用户批准的技能修改。
- `/poteto-mode write a skill for <workflow>` 运行编写 playbook。Eval playbook 以盲测方式测试技能改动。
- 在技能自己的 PR 中修复行为异常的技能，而不是在它出错的那项功能工作里顺手修。

## 回复

先给答案。最多给出一个写在代码块里的示例提示，合适时从 [`references/recipes.md`](references/recipes.md) 改写，然后给出该文件的链接。除非用户要完整地图，否则保持简短。
