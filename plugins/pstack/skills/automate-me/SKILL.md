---
name: automate-me
description: "用于 “automate me”、“create/update/refresh my -mode skill”、“turn/capture my preferences or working style into a skill”，或希望代理按用户的工作方式行事时。通过 plugin-dev:skill-development + unslop 起草或修订个人 -mode 技能，可选地从近期会话记录中提取新证据。"
---

# Automate me

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

一条引导式流程，把用户的工作惯例变成代理会遵循的技能。产出是一个为其定制的 `-mode` 技能（例如 `jay-mode`、`priya-mode`）。

本技能编排另外三个环节：一次内联的挖掘（见第 1 步）、**plugin-dev:skill-development** 技能（撰写）和 **unslop** 技能（文字纪律）。它只负责排序，不替代它们。

## 流程

### 0. 检查是否已有技能

递归查找与用户 handle 匹配的 `.claude/skills/**/*-mode/SKILL.md` 和 `~/.claude/skills/*-mode/SKILL.md`。mode 技能也可以放在个人分类目录里（`.claude/skills/<handle>/`），不只在顶层。如果找到了一个，用 `AskUserQuestion` 确认意图（除非用户已经说过“update my skill”之类的话）：

- 更新已有技能（重复运行时的默认选项）
- 从头重做（少见，做之前先问清原因）

更新模式会改变后续流程：
- 第 1 步只挖掘该技能上次编辑之后的历史（`git log -1 --format=%cI <path>`）。
- 第 2 步问的是哪些地方变了或缺失，而不是从零开始要收集什么。
- 第 4 步原地编辑已有文件。保留用户没有否定过的章节。有新的证据就修订相应章节。只对真正新的规则新增章节。

### 1. 挖掘用户的历史

在铺开之前，先定位当前工作区的会话记录。Claude Code 把它们存在 `~/.claude/projects/<encoded-cwd>/*.jsonl`，其中 `<encoded-cwd>` 是工作区的工作目录，`/` 替换为 `-`。只用这个路径。不要跨 `~/.claude/projects/` 做 glob。那会越过工作区边界，读到无关项目的私密对话。

在该范围内梳理近期的代理对话，找反复出现的模式。按历史切片运行多个并行子代理（例如最近 2-4 周，切成 3 片，让每片都有足够的材料）。按 **poteto-mode** 技能的 Subagents 一节，为每个子代理的活挑选推理档位。每个切片挖掘子代理读取父代理给出的、限定在工作区范围内的路径下的会话记录，寻找下面这些信号，并返回一份简短的结构化清单，列出它看到的模式及证据指向。默认值得找的信号：

- 回复偏好（长度、语气、格式、“说简单点”式的纠正）
- 委派习惯（子代理、模型、专门流程、并行度）
- 验证姿态（“完成”意味着什么、单元测试还是实机复现、评审者）
- 代码与文字纪律（风格、引用的原则、lint/format 工具）
- 流程惯例（worktree、提交、PR、评审/合并工具）
- 元偏好（中途修技能、提议新技能）

提升某个信号之前先跨切片交叉核对。在两个及以上切片里出现的模式是高置信的。孤立的信号很弱，通常直接丢掉。

### 2. 直接问用户

挖掘会漏掉还没出现过的意图。用 `AskUserQuestion` 工具（结构化多选），而不是让用户从零开始打字。

形态：一到两个问题，每个 4-6 个选项，分类问题用 `allow_multiple: true`。先问宽泛的（“哪些方面最重要？”），再针对选中的方面用具体选项追问。结构化几轮之后，用一个自由文本的问题兜住选项没覆盖到的部分。

别一口气甩 20 个问题。

### 3. 归类发现

把合并后的信号分组到章节。常见的几类（只取适用的）：

- **回复风格**：长度、语气、格式。
- **自主度**：多少事可以不问就做、MCP 工具的使用。
- **先理解**：界定或调查改动时该用哪些技能。
- **子代理**：默认用法、并行度、模型与任务的匹配、专门流程。
- **文字 / 代码纪律**：原则、lint 工具、风格指南。
- **评审与验证**：复现姿态、验证技能、实机测试工具。
- **流程**：git worktree、提交、PR、评审/合并工具。
- **技能**：技能撰写习惯、先修技能、提议新技能。

**poteto-mode** 技能展示了应有的形态。去读它了解颗粒度。不要抄它的内容。用户的规则不等于 poteto-mode 的规则。

### 4. 起草技能

用 **plugin-dev:skill-development** 技能来撰写。放置位置：

- 路径：沿用已有 mode 技能的分类。新建 mode 时，如果仓库已经为该 handle 建了个人分类，用 `.claude/skills/<handle>/<handle>-mode/SKILL.md`。否则默认放在项目的 `.claude/skills/<handle>-mode/SKILL.md`（用户更想要个人技能时用 `~/.claude/skills/<handle>-mode/`）。
- Handle：用户的名字或自选标识。
- frontmatter 的 `description`：以他们的名字、`/<handle>-mode` 和“按他们的风格工作”作为触发条件，而不是 “write code” 或 “review PR” 这类泛化关键词。
- frontmatter 格式：遵循 `plugin-dev:skill-development` 的 YAML 规则。`description` 保持为单个 YAML 标量。标点或折行需要时，给它加引号，或用 `description: >-` 加缩进的续行。
- frontmatter 默认写 `disable-model-invocation: true`。只有用户明确希望自己的 mode 在每一轮都生效时才去掉。

### 5. 打磨文字

对每一行都应用 **unslop** 技能和 `plugin-dev:skill-development` 的写作准则。

把草稿给用户看，收集反馈。预计要改几轮。下手要狠。mode 技能不是说明书。

### 6. 落地

在基于 main 的 worktree 里作业。提交并开 PR。不要直接推到 main。

## 护栏

- **不要过拟合单次对话。** 说过一次、另一次又被推翻的偏好是噪声。要多次出现才把它写成规则。
- **别耍聪明。** 复述其他技能的内容、生造比喻、给代理读者写“诗意”文字，都是有成本无收益。保持可操作。
- **引用，不要内联。** 用户依赖的其他技能应以路径引用出现，不要贴片段。他们在别处维护的原则文档同理。
- **章节要少。** 只有当用户在该方面有具体且非默认的规则时才加章节。“沟通要清楚”不成其为章节。“段落要短。比较选项时用表格。只有条目真正并列时才用列表。”才是。
- **称谓保持通用。** 祈使句里用“用户”或“人”，不要用作者的名字。
- **不要强求对称。** 用户没有值得写下来的流程规则，就整个跳过流程章节。

## 评估

`-mode` 技能是主观产出。`plugin-dev:skill-development` 那种测试/迭代的基准循环在这里没用。和用户凭感觉核对一下：读起来像他们吗？漏了什么吗？然后交付。

只有当技能的触发准确率在实践中成了问题时，才跑 description 优化循环。

## 何时不用

- 用户想要针对具体任务的技能（而非工作惯例）：只用 `plugin-dev:skill-development`，不需要挖掘。
- 用户想固化一个很窄的流程（例如“我怎么写提交信息”）。那是普通技能，不是 mode 技能。
