---
name: reflect
description: 在当前对话记录上并行派生三个评审子代理，提炼出经验，并把每条经验落到对某个现有技能的具体修改上。在用户说 reflect 时使用。
---

# Reflect

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

从当前对话中挖掘持久的经验，然后把它们转成技能修改。

在用户说 "reflect" 或 "/reflect" 时调用。如果对话很琐碎、离题，或者已经被某个现有技能覆盖且父代理正确遵循了它，就跳过。一次性的事情不算经验。

## 1. 定位当前对话记录

在并行展开之前，先在 Claude Code 的按项目对话记录目录 `~/.claude/projects/<encoded-cwd>/` 中找到本次对话的记录。不要跨 `~/.claude/projects/` 做 glob。那会越过工作区边界，读取无关项目的私密对话。

运行已安装插件下位于 `skills/reflect/scripts/find-transcript.mjs` 的查找脚本，传入项目目录和本次对话开头用户提示的一个片段：

```bash
node <plugin>/skills/reflect/scripts/find-transcript.mjs ~/.claude/projects/<encoded-cwd> "<opening prompt fragment>"
```

它覆盖三种布局（扁平的 `<id>.jsonl`、嵌套的 `<id>/<id>.jsonl`、子代理的 `<parent>/subagents/<child>.jsonl`），按从新到旧排列，并打印第一个开头键入提示包含该片段的路径。不要手工重新实现这套扫描：对话记录的第一行是会话元数据而不是消息；以 `/clear` 或 `!` shell 命令开头的会话，会在提示之前把该命令的包装和输出记录为 `user` 记录；文件可达数 MB，所以查找脚本以流式方式读取每个候选，并在遇到第一条键入的 `user` 记录时停下。如果它以 1 退出，就为本次会话写一份紧凑的摘要并传入它。摘要按轮次顺序，每项一行，列出用户的提示和纠正、工具调用及其结果、以及改动过的文件。It states no diagnosis, verdict, or cause.（摘要不给出任何诊断、结论或原因。）评审者评判的是会话本身，而不是你对它的结论。

## 2. 并行派生三个评审者

一条消息，三次 `Agent` 调用，`subagent_type: "general-purpose"`。评审者需要 MCP 访问权限来查找对话记录中引用的工单、聊天线程和可观测性追踪，所以要选一个保留 MCP 访问权限的 subagent_type。提示禁止写文件。修改由父代理来应用。

每个评审者和综合者都对应 `pstack-models.md` 中的一个角色行，以及 [Models](#models) 中的一个默认值。把 `model` 设为该行的值，若表格或该行缺失则用默认值。当值为 `auto` 或 `inherit-parent` 时不设置 `model`。如果 `Agent` 工具拒绝某个 slug，使用默认值并说明这一点。如果它拒绝默认值，根据其错误信息选择同族中最接近的合法 slug。按任务挑选每个子代理的推理等级，参见 the **poteto-mode** skill 的 Subagents 段。

| 视角 | 角色行 | 提示模板 |
|---|---|---|
| Judgment | `reflect judgment, divergent, synthesizer` | `references/judgment-reviewer.md` |
| Tooling | `reflect tooling` | `references/tooling-reviewer.md` |
| Divergent | `reflect judgment, divergent, synthesizer` | `references/divergent-reviewer.md` |

逐字传入每个模板，只在标记处替换为对话记录路径或摘要。Add nothing beside a transcript path, not even a focus note for a long transcript.（除对话记录路径外什么都不要加，即使对话记录很长，也不要加关注点说明。）评审者在 `Agent` 响应正文中返回发现。

## 3. 综合

一次 `Agent` 调用，`subagent_type: "general-purpose"`，`model` 取自 `reflect judgment, divergent, synthesizer` 行（默认值见 [Models](#models)）。它会通过 MCP 抽查引用，所以要选一个保留 MCP 访问权限的 subagent_type。逐字传入 `references/synthesizer.md`，并在标记处内联每个评审者的完整输出。它返回一份 Accepted / Rejected / Backlog 列表。

## 4. 结构化约束检查

把任何用 lint 规则、脚本、元数据标志或运行时检查来约束会更可靠的 Accepted 条目移到 Backlog。参见 **encode-lessons-in-structure** principle skill。

## 5. 应用

展示综合者完整的 Accepted / Rejected / Backlog 输出，并在应用任何 Accepted 修改之前等待明确批准。用户挑选子集，也可以改变分流去向。技能的改动会影响组织里今后的每一个代理。不要自动应用。

把每个 Backlog 条目直接提交到团队的 devex 或 backlog 跟踪器，不必等待。只有 Accepted 列表需要等待批准。

严格按照每个已批准行的 Routing 执行：

- 对现有技能的小修改（一条单行要点、一句收紧的表述、一个过时事实的更正）：父代理直接完成。
- 对现有技能的实质性修改（新增一段、一张新的模式表、超过约 10 行）：交给 the **plugin-dev:skill-development** skill，运行它的起草 / 测试 / 迭代循环。
- `tune description: <skill path>`（技能存在，但在应该触发时没有触发）：交给 `plugin-dev:skill-development`，运行它的描述优化循环。
- `new skill via plugin-dev:skill-development: <kebab-name>`：把创建工作交给 `plugin-dev:skill-development`。不要临时自创技能形式。

如果你的环境附带 SKILL.md 校验器，在宣布完成之前对每个改动过的技能运行它。

## 6. 向用户总结

简短列表，不要开场白：

- 已应用的修改：`<skill path>`。改了什么，每条一行。
- 新建的技能：`<skill path>`。每条一行（少见）。
- 提交到 devex 跟踪器的 Backlog：`<issue title>`（`<tags>`）。每条一行。
- 已舍弃：每个被拒绝的发现一行，附综合者给出的理由。

## Models

Role defaults, stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in the `pstack-models.md` override sheet overrides each at runtime; `/setup-pstack` writes it and lists its path per runtime.

- reflect tooling: `opus`
- reflect judgment, divergent, synthesizer: `opus`

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
