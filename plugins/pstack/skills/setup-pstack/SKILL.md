---
name: setup-pstack
description: 配置 pstack 为每个角色使用哪些模型。检测可用模型，并写出当前运行时的覆盖表。用于 /setup-pstack、"configure pstack models"、更改 pstack 的模型选择，或开启、关闭 SessionStart hook。
---

# 设置 pstack

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, follow [the Copilot setup](copilot.md) in place of steps 1, 3, 4, and 7 and step 6's header.

若使用其他运行时，请阅读下文的[其他运行时](#other-runtimes)一节，了解覆盖表放在哪里、如何加载；步骤是一样的。

写出当前运行时的按角色模型覆盖表，路径见[其他运行时](#other-runtimes)。每个 pstack 技能都在正文中写明默认模型；覆盖表负责把这些默认值适配成你实际可用的模型。

Claude Code 没有 Cursor 的 `.mdc` 那样自动生效的 "rules" 机制。当 `$CLAUDE_CONFIG_DIR` 已设置时，Claude Code 的配置目录就是它，否则为 `~/.claude`。本技能称它为 `<config>`。引用是显式的：用户在 `<config>/CLAUDE.md`（或项目的 `CLAUDE.md`）中加入一行，例如：

```text
@<config>/pstack-models.md
```

其中 `<config>` 写成解析后的路径，这样该文件会在每个会话中作为上下文加载。

当 `$CODEX_HOME` 已设置时，Codex 的主目录就是它，否则为 `~/.codex`。本技能称它为 `<codex-home>`。

## Steps

### 1. 检测可用模型

枚举本会话中 `Agent` 工具的 `model` 参数所接受的模型名。这是可靠的来源。在 Claude Code 上，它们就是下文 [Models](#models) 列出的家族名，每个都指向该家族当前的模型，而完整模型 ID 会被拒绝。默认面板也在那里列出。该面板是按跨家族多样性挑选的。请用户确认，或粘贴他们希望额外启用的 slug。绝不要写入尚未确认可用的真实 slug。别名 `inherit-parent` 和 `auto` 始终有效，即使它们不是检测到的 slug。两者都表示该角色运行在父会话的模型上，`Agent` 调用通过省略 `model` 来表达这一点。

### 2. 载入当前状态

默认的角色到模型映射就是下文「写出覆盖表」步骤中展示的规则形状。如果当前运行时的覆盖表已存在，读取它，把其中的值当作当前选择；否则从这些默认值开始。若某行的角色不在这套形状里（例如 `how critics`），说明它来自已废弃的角色，直接丢弃。较旧的覆盖表可能写着以 `claude-` 开头的完整模型 ID，而 `Agent` 工具会拒绝这种写法。把它们分别替换成家族名，即 `claude-` 后面的那个词。

### 3. 映射并确认

列出每个角色及其当前模型，把不在检测集合中的真实 slug 标记为需要选择。同时列出步骤 2 丢弃或改写的每一行。询问用户是原样接受，还是修改特定角色，并把检测到的模型加上 `inherit-parent` 和 `auto` 作为选项提供。优先用 `AskUserQuestion`，而不是自由文本。对于面板类角色（arena runners、architect runners、interrogate reviewers），其值是一个列表，每个条目运行一个 subagent，别名条目也算，因此列表长度决定数量。`arena cross-judge pool` 也是一个列表，但 Arena 会从中挑一个值，在可能的情况下选模型家族与父会话不同的那个。`swarm workers` 是每个 worker 的默认模型，除非某场竞赛或比较为某个分支单独指定了另一个模型。

接着询问默认推理强度，即 `default effort` 行。它的值是 `session`，表示沿用父会话的强度，或者是 [Models](#models) 中的某个级别。从那里列出的默认值开始。每个不带后缀的角色值都按它运行。然后询问是否有角色需要以另一级别运行。在 Claude Code 上，角色值可以在 slug 后带一个级别，例如 `<slug> @xhigh`；面板条目各自携带自己的级别，例如 `<slug> @xhigh, <slug> @max`。每个级别都通过插件中该级别的 effort agent 派发，因此这一选择无需改动会话即可生效。默认强度请省略后缀。

### 4. 选择是否由 session hook 路由任务

在 Claude Code 和 Codex 上，插件的 `SessionStart` hook 会在启动、恢复、清空和压缩时注入 poteto-mode 的指令。Codex 会要求用户先通过 `/hooks` 信任插件 hook，然后才会运行它。在 Pi 上，pstack 扩展会在每次 agent 启动时把同样的指令加入系统提示。询问是否保留该 hook。默认开启。答案就是当前运行时覆盖表中的 `session hook` 行：`on` 或 `off`。没有覆盖表或没有这一行时，hook 会注入。在其他运行时上，这一行不起作用。

### 5. 校验

写入的每个真实 slug 都必须属于检测集合。`inherit-parent` 和 `auto` 始终通过。校验 slug 时去掉任何 `@<level>` 后缀，并对照 [Models](#models) 中的强度级别校验该级别。`default effort` 的值是这些级别之一或 `session`。在 Codex 上，级别改为你的 Codex 模型所接受的 `reasoning_effort` 值。如果选中的真实 slug 或级别不可用，停下来重新询问。

### 6. 写出覆盖表

按下面的形状写出当前运行时的覆盖表。整份文件覆盖写入，这样重复运行保持幂等。

```markdown
# pstack model configuration

Per-role model overrides for pstack skills. Each pstack SKILL.md names its defaults in a Models section; the values here override those defaults. Delete a line to fall back to the skill default. A value of `inherit-parent` or `auto` runs that role on the parent session's model (the `Agent` call omits `model`); an alias entry in a panel list still counts toward that panel's fan-out. A model may carry a reasoning effort, as in `opus @xhigh` (levels: low, medium, high, xhigh, max); the role then runs through the pstack effort agent of that level, each entry of a panel list on its own. `default effort` sets the level for a value without one; `session` keeps the parent session's effort. `session hook: off` stops the Claude Code or Codex SessionStart hook, or the pstack Pi extension, from injecting the poteto-mode mandate; any other value, or no line, leaves it on.

feature, refactoring: sonnet
bug-fix: opus
perf-issue: opus
hillclimb: opus
judgment and prose: opus
strongest judgment: fable
how explorer: opus
how explainer: opus
why investigators: opus
why synthesizer: opus
reflect tooling: opus
reflect judgment, divergent, synthesizer: opus
arena runners: opus, fable, sonnet, haiku
arena cross-judge pool: opus, fable, sonnet, haiku
swarm workers: opus
architect runners: opus, fable, sonnet, haiku
interrogate reviewers: opus, fable, sonnet, haiku

default effort: session
session hook: on
```

### 7. 接入配置

在 Claude Code 上，如果 `<config>/CLAUDE.md` 尚未包含 `<config>/pstack-models.md`，就追加一行 `@` 引用，指向覆盖表解析后的路径，例如 `@~/.claude/pstack-models.md`，这样模型行会在每个会话中加载。如果用户偏好项目范围，就把该引用加到项目的 `CLAUDE.md`。

在 Codex 上，把模型行和 `default effort` 行粘贴进 `<codex-home>/AGENTS.md`；Codex 没有 `@` 引用机制。不要把 `session hook` 行复制过去：插件 hook 会直接从 `<codex-home>/pstack-models.md` 读取它。

### 8. 确认

告诉用户覆盖表写到了哪里、其中的模型行如何加载，以及插件 hook 是否开启。重新运行本技能会更新覆盖表。

### 9. 提供验证技能（可选）

检查项目是否已有驱动真实应用来取证的方式（项目里的 `verify` 或 `verify-*` 技能，或现有的 harness）。如果没有，询问一次："want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." 若同意，调用 [`/create-verification-skill`](../create-verification-skill/SKILL.md)。若不同意，继续往下，不要劝。

### 10. 提供形式化验证技能（可选）

检查项目是否已有形式化验证技能（一个对代码运行模型检查器、证明或其他验证器的项目技能）。如果没有，询问一次："want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." 若同意，调用 [`/create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md)。若不同意，继续往下，不要劝。

<a id="other-runtimes"></a>

## 其他运行时

角色行在所有运行时上都一样。不同的是覆盖表路径、运行时的加载方式，以及如何列出模型。用运行时自己的工具检测模型，绝不要写入你没见它列出的 slug。即使某个运行时的 subagent 调用没有 model 参数，它仍然会有覆盖表，作为用户选择的记录，并在力所能及处应用。`session hook` 行适用于 Claude Code、Codex 和 GitHub Copilot 插件以及 pstack Pi 扩展。

| 运行时 | 覆盖表 | 加载方式 | 列出模型 | 状态 |
| --- | --- | --- | --- | --- |
| Claude Code | `<config>/pstack-models.md` | `<config>/CLAUDE.md` 中的 `@<config>/pstack-models.md` | `Agent` 工具的 model 参数 | 已实测验证 |
| Codex | `<codex-home>/pstack-models.md` | 模型行：粘贴到 `<codex-home>/AGENTS.md`；hook 设置：由插件读取 | 你配置的 Codex 模型，见 [codex-tools.md](../poteto-mode/references/codex-tools.md#model-names) | hook 契约已测试；模型发现已验证 |
| Pi | Pi agent 目录中的 `pstack-models.md`，即 `$PI_CODING_AGENT_DIR` 或 `~/.pi/agent` | 由 pstack Pi 扩展读取，模型行和 hook 设置都读；没有 include 行 | `pi --list-models`，见 [pi-tools.md](../poteto-mode/references/pi-tools.md#model-names) 及其 `setup-pstack` 说明 | 扩展契约已离线测试；实测结果见仓库的 `docs/pi-equivalence.md` |
| GitHub Copilot（CLI 与 app） | `${COPILOT_HOME:-~/.copilot}/pstack-models.md` | 插件 hook 会检查它，并在会话开始时注入其中的角色行；仅安装技能时用 `view` 读取 | `task` 工具的 `model` 枚举，见 [copilot-tools.md](../poteto-mode/references/copilot-tools.md#model-names) | hook 契约已测试；CLI 安装已做冒烟测试 |
| opencode | `~/.config/opencode/pstack-models.md` | 把该路径加入 `opencode.json` 的 `instructions` 数组 | 会话内的 `models` 斜杠命令 | 依据公开文档，未做实机会话 |
| Gemini CLI | `~/.gemini/pstack-models.md` | `~/.gemini/GEMINI.md` 中的 `@~/.gemini/pstack-models.md` | 会话内的 `model` 斜杠命令 | 依据公开文档，未做实机会话 |
| Prime Agent | 没有文档记录的覆盖表路径；由 Prime 的配置选择模型 | | | 未做实机会话 |

## Models

Stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`).

- Available Claude models: `opus`, `fable`, `sonnet`, `haiku`
- Default panel: `opus`, `fable`, `sonnet`, `haiku`
- Reasoning effort levels: `low`, `medium`, `high`, `xhigh`, `max`
- Default reasoning effort: `session`
- Single-role default: `opus`
