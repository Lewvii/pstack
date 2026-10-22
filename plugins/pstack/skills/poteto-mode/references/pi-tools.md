# Pi tool mapping

pstack 技能用 Claude Code 的工具语言写成（the `Skill` tool、the `Agent` tool、`AskUserQuestion`、Claude 模型名）。在 Pi 上技能文件完全相同，从 pstack 包加载；包内的 Pi 扩展注册了以 Claude 工具命名的工具，因此大多数名称一一对应。当 pstack 技能提到某个 Claude 工具、驱动或内置技能、或 Claude 模型时，阅读本文件。本文件仅适用于 Pi。

## Tool actions

| pstack / Claude action | Pi 对应 |
|------------------------|---------------|
| Read a file | `read` |
| Create a file | `write` |
| Edit a file | `edit`（精确文本替换） |
| Delete a file | `bash`（`rm`） |
| Run a shell command | `bash` |
| Search file contents / find files | 启用时用 `grep`、`find` 和 `ls`，否则用 `bash`（`rg`、`grep`、`find`、`ls`）。Pi 默认只启用 `read`、`bash`、`edit` 和 `write`。 |
| Fetch a URL | `bash` 配合 `curl` / `wget` |
| Search the web | Pi 没有网页搜索工具。使用提供该功能的 MCP 服务器。 |
| Invoke a skill (the `Skill` tool, `/command`) | 技能原生加载。当技能描述匹配时模型会读取它；`/skill:<name>` 可强制加载某个技能。 |
| Dispatch a subagent (the `Agent`/`Task` tool) | `agent` |
| Dispatch N parallel subagents in one turn | 在一次响应中发出 N 个带 `run_in_background: true` 的 `agent` 调用 |
| Wait for a subagent result | 前台 `agent` 调用返回最终文本。后台 agent 完成时会以消息形式送达，包含其 id、状态和最终文本；它在你当前的工具调用之后到达，或在你空闲时作为新回合到达，因此不要轮询。 |
| Continue or steer a subagent (`SendMessage`) | `send_message`，按 agent id 或描述寻址 |
| List subagents | `list_agents`。其 `sessionFile` 字段是该 agent 的 transcript，在第一条消息写入后生成。要让用户观察运行中的 agent，给他们 `tail -f <sessionFile>`；已完成的 agent 可用 `pi --export <sessionFile>` 渲染为 HTML。运行中 agent 的 `activity` 字段给出它当前或最近一次工具调用以及最新消息。用户可以运行 `/pstack-agents` 查看同样的信息。 |
| Stop a subagent | `stop_agent` |
| Track tasks (the todolist; `TaskCreate` / `TaskUpdate`, or `TodoWrite` on Claude Code) | Pi 没有任务跟踪工具。按 poteto-mode 针对这种情况的描述，维护 `todo.md` 清单。 |
| Ask the human a fixed-choice question (`AskUserQuestion`) | `ask_user_question`，形式相同。没有交互式 UI 时（print 或 JSON 模式）以及在子 agent 中，它会返回错误，此时用纯文本提问。 |
| Schedule a self-paced re-invocation (`ScheduleWakeup`) | `schedule_wakeup`，形式相同。到期的唤醒会一直挂起，直到会话空闲，手动压缩期间也是如此。`noop: true` 只给唤醒打标签，唤醒仍会被调度；`stop: true` 取消挂起的唤醒，且不需要其他字段。在 print 和 JSON 模式下以及在子 agent 中，调度会返回错误，因为运行结束时 Pi 就会退出，唤醒永远无法触发。某次运行期间若 `/loop` 命令结束或替换了循环，该运行在结束前的每次唤醒都会被拒绝，无论 prompt 是什么，Pi 会告知用户。其他唤醒不会被拒绝。 |
| Read this workspace's session transcripts (Claude Code's `~/.claude/projects/<encoded-cwd>/`) | Pi 会话位于 `~/.pi/agent/sessions/--<cwd>--/`（设置了 `$PI_CODING_AGENT_DIR` 时位于其下），其中 `<cwd>` 是去掉开头 `/`、并把每个 `/` 换成 `-` 的工作目录。扩展启动的 subagent 会话位于 `<agent dir>/pstack/<parent session id>/agents/`。每个文件以一行 `session` 头开始；之后每一行是一个带 `id` 和 `parentId` 的条目，消息条目为 `type: "message"`，其 `message.role` 为 `user`、`assistant` 或 `toolResult`。会话会把被放弃的分支保存在同一个文件中，因此要从最后一个条目沿 `parentId` 回溯，得到实际发生的对话。同样的隐私规则适用：只读取本工作区的目录，绝不对 `sessions/` 使用通配。 |

`agent`、`send_message`、`list_agents`、`stop_agent`、`ask_user_question` 和 `schedule_wakeup` 来自 pstack Pi 扩展，通过 `pi install` 安装 pstack 包时会加载。仅安装技能时这些工具都没有，扇出类技能（`interrogate`、`why`、`how`、`arena`、`reflect`）退化为单次顺序执行。必需的独立评审不会退化，而是保持阻塞，见 poteto-mode 的 [Subagents](../SKILL.md#subagents) 一节。

## Subagent policy

poteto-mode 的 Subagents 一节在 Pi 上通过 `agent` 工具生效：

- `subagent_type` 取值相同。`pstack:poteto-agent`、`pstack:comment-sicko`、`pstack:thermo-nuclear-review-subagent`、`pstack:thermo-nuclear-code-quality-review-subagent`、`pstack:poteto-agent-<level>` 和 `pstack:effort-<level>` 解析到插件的 agent 文件。`general-purpose` 或不指定 `subagent_type` 时，运行一个没有 agent 文件的子进程。Claude Code 内置的 `Explore`、`Plan` 等类型在 Pi 上不存在，因此使用 `general-purpose` 并把约束写进 prompt。未知类型会报错并列出有效类型。
- `readonly: true`（`how` 和 `interrogate` 小组会设置）运行的子 agent 没有 `edit` 和 `write` 工具。它保留 `bash`，因此评审者仍可运行 `git diff`；在 prompt 中说明它不得修改文件。
- 每个子 agent 都是你工作目录下独立的 `pi --mode rpc` 进程，扩展通过它的 stdin 发送 prompt。它加载磁盘上保存的包和设置，所以在 pstack 通过 `pi install` 安装时能看到 pstack 技能。它不继承父进程的命令行参数，例如 `-e`、`--api-key` 或一次性的 `-a` 项目信任，因此只通过 `-e` 加载的 pstack 不会给子 agent 提供扩展。
- 子 agent 可以启动自己的 agent，最多到主会话以下三层，与 Claude Code 相同。第三层的子 agent 运行时没有 `agent` 工具。
- `isolation: "worktree"` 让子 agent 在 `.claude/worktrees/` 下自己的 git worktree 中运行。agent 结束时，如果它在 worktree 中没有留下任何东西，worktree 会被删除；若有改动、它写入的被 gitignore 的文件、或其分支上或其 HEAD 访问过的任何提交上有 commit，则保留。
- `run_in_background: true` 立即返回 agent id。完成通知在你当前的工具调用结束后加入对话，与 Claude Code 相同；你空闲时则开启一个回合。在 print 和 JSON 模式（`pi -p`）下，Pi 会在运行结束后退出，扩展会在后台 agent 运行期间推迟结束，因此每个完成通知仍以回合形式到达，进程在最后一个之后结束。子 agent 在自己的后台 agent 运行期间也以同样方式推迟结束，因为其父进程在它结束后会关闭它的 stdin；来自父进程的消息会结束这种推迟。交互式和 RPC 主会话照常结束，并在完成通知到达时接收。
- 后台 agent 运行期间，扩展会拦截这样的 `bash` 命令行：单词 `sleep` 后紧跟两秒或以上的字面时长，出现在行内任何位置都算，包括轮询循环：`sleep 30`、`npm test; sleep 30`、`until gh pr checks 1; do sleep 30; done`、`sleep infinity`。更短的 sleep、以 `sleep 30 &` 放到后台的 sleep，以及位于引号字符串、注释或 heredoc 中的 `sleep` 都可以运行。该规则宁可误拦，因此 `timeout 5 sleep 30` 和后台命令组中的 sleep 也会被拦截。完成通知会自行到达，所以继续其他工作或结束你的回合。
- agent 的状态跟随其进程。`completed` 表示子进程已退出，`stop_agent` 只有在子进程退出后才报告 `stopped`，因此 Claude Code 中关于 `completed` 的 agent 仍在运行的注意事项不适用于 agent 本身。停止 agent（以及停止所有 agent 的会话关闭）会结束子 pi 进程及其仍在运行的 `bash` 命令，因为 Pi 在收到 SIGTERM 时会杀掉该命令的进程树。已结束的 `bash` 命令留在后台的进程（`cmd &`、`nohup cmd`）运行在自己的进程组中，会比 agent 活得更久；需要你自己停止它。被停止的 agent 的通知加入对话但不开启回合，因为 `stop_agent` 已经返回。
- agent 属于启动它的会话。退出、重新加载，以及启动、恢复或分叉会话，都会停止所有运行中的 agent；父进程退出时会通知其 agent 停止。恢复时，仍在其启动 Pi 进程下运行的 agent 不受影响。进程身份与保存记录匹配的孤儿进程会收到停止信号，在其进程退出前，`send_message` 和 `stop_agent` 都无法使用它。如果没有足够的身份信息来安全地发送信号，就不动它的进程，并在确认其退出前拒绝继续。只有到那时，恢复的 agent 才被标记为 `stopped` 并允许恢复运行。
- 对运行中 agent 的 `send_message` 是写入子进程 stdin 的 RPC `steer`。agent 在当前工具调用之后读取它，与 Claude Code 相同，并在同一次运行中继续，最终发送一个完成通知。被子进程拒绝的消息返回错误，agent 继续运行。发给已完成 agent 的消息会在后台恢复它的会话，并带有之前各次运行的上下文。在 agent 已结束但进程尚未退出时到达的消息，会在进程退出后恢复该 agent。
- 角色值的 `@<level>` 选择与 Claude Code 相同的 effort agent，扩展把该级别作为 `--thinking` 传给子进程。`session` 或没有 effort 的 agent 以父进程当前的思考级别运行子进程。
- 策略的其余部分保持不变。传递文件指针而不是内联上下文，写入的 worker 各自使用独立的 worktree，并亲自评审每个 subagent 的 diff。

@@MODEL_NAMES@@

## Model names

Skills name models by the Claude aliases in their Models sections. On Pi, pass the alias as the `agent` tool's `model`. The pstack extension resolves it in the column of the provider the session's current model comes from, and in the `anthropic` column for any other provider:

| Alias | `anthropic` | `openai` | `openai-codex` |
| --- | --- | --- | --- |
| `opus` | `anthropic/claude-opus-5-5` | `openai/gpt-6.1-sol` | `openai-codex/gpt-6.1-sol` |
| `fable` | `anthropic/claude-fable-5-1` | `openai/gpt-6-astra` | `openai-codex/gpt-6-astra` |
| `sonnet` | `anthropic/claude-sonnet-5-5` | `openai/gpt-6-sol` | `openai-codex/gpt-6-sol` |
| `haiku` | `anthropic/claude-haiku-5-5` | `openai/gpt-6-luna` | `openai-codex/gpt-6-luna` |

Pi warns that Anthropic bills Claude used through Pi per token, as extra usage, even on a Claude subscription. Pi shows that warning only in interactive mode, never for the `pi --mode rpc` children the `agent` tool runs.

A `pi models: opus=<provider/id>, sonnet=<provider/id>` line in the Pi override sheet points each alias it names at another Pi model, whatever the session's provider. Add one when the session's provider has no column above and Pi has no credentials for `anthropic`, because each alias then resolves to an `anthropic/*` ID and the `agent` call fails with `No API key found for anthropic`. The `agent` tool also takes a full `provider/id`, passed through unchanged, and `inherit-parent`, `auto`, or no `model` runs the child on the parent's current model. Diverse-model panels (`arena`, `architect`, `interrogate`, `how` critics, `reflect`) stay diverse only while their aliases resolve to distinct models. If one model family is all you can reach, vary the reasoning effort and note in the verdict that diversity was reduced.

`/setup-pstack` writes the configured model list. On Pi, keep the aliases and remap them with `pi models:`.

## Session routing

pstack Pi 扩展在每次 agent 启动时，把 poteto-mode 指令（即 Claude Code 和 Codex 的 `SessionStart` hook 注入的文本）加入系统提示词，因此该指令在压缩后依然存在。它还会加入 Pi 覆盖配置表，因为 Pi 的上下文文件没有 include 语法。配置表是 Pi agent 目录下的 `pstack-models.md`，该目录在设置了 `$PI_CODING_AGENT_DIR` 时为其值，否则为 `~/.pi/agent`。配置表中的 `session hook: off` 停止注入指令。由 `agent` 工具启动的子 agent 会得到配置表，但得不到指令，这与 Claude Code subagent 能看到 CLAUDE.md 但不运行 `SessionStart` hook 的情况一致。每个会话（包括子 agent 和关闭了 hook 的会话）还会得到指向本文件的指针，以及 Claude Code 中关于在一次响应中发出相互独立的工具调用的那句话。

没有扩展时什么也不会注入。用 `/skill:poteto-mode` 显式请求 `poteto-mode`，或在 `AGENTS.md` 中添加常驻指令。

## Driver and bundled skills pstack references

[驱动策略](../SKILL.md#non-negotiables) 决定使用哪个应用驱动。对于这些工作流提到的技能和驱动，使用以下 Pi 对应项：

| pstack 中提到的技能或驱动 | 在 Pi 上 |
|---------------------------------|-------|
| `run`（驱动 CLI/TUI 以观察变更生效） | Pi 没有 `run` 技能。通过 `bash` 自行运行应用并观察真实输出。 |
| `verify`（项目的 `.claude/skills/verify/`，或 Claude Code 内置的 `/verify`） | Pi 不会发现 `.claude/skills/`。按路径读取项目技能的 SKILL.md，或把 `../.claude/skills` 加入 `.pi/settings.json` 的 `skills` 列表。没有项目技能时，按 `run` 的方式驱动应用。 |
| 项目 UI 驱动 | 用你手头的任何自动化手段驱动 UI，或交给用户一项具体的手动检查。未观察到产物前不要声称完成。 |
| `plugin-dev:skill-development`（Claude 的 SKILL.md 编写指南） | 遵循 Pi 的技能文档和 Agent Skills 规范。保留 `name` + `description` frontmatter，目录以技能命名，并使用渐进式披露。 |
| `loop`（循环或自定节奏的重复调用，`babysit` 使用） | 扩展的 `/loop [interval] <prompt>` 命令。带间隔时按该节奏重复触发。不带间隔时 prompt 立即运行，由模型用 `schedule_wakeup` 自定节奏。`/loop stop` 取消循环，迭代运行期间也可以：该次运行之后调度的任何唤醒都不会生效。运行期间给出的 `/loop` 会在该次运行结束后开始。 |

## Per-skill notes

没有任何技能文件带有 Pi 专用行。poteto-mode 的 Platform Adaptation 一节在这里指向一次，扩展的系统提示词在每个会话中都指向这里，所以在执行某个技能前先读它对应的行。大多数技能只需要上面的表格。以下技能还需要一项额外映射：

| Skill | On Pi |
|-------|-------|
| `poteto-mode` | 任务清单回退到 `todo.md`，Subagents 默认值通过上文 Subagent policy 映射。Eval 和 Session pickup playbook 从 Pi 会话目录读取 transcript（见 Tool actions）。 |
| `interrogate` | 评审者通过 `agent` 派发，使用相同的 `subagent_type`、`model` 和 `readonly`（见 Subagent policy）。让小组保持使用不同的模型（见 Model names）。 |
| `recall` | 搜索本工作区的 Pi 会话目录（见 Tool actions）。不要列出或读取 `~/.claude/projects/`：那里存放的是 Claude Code transcript，不是本会话的历史。匹配 `type: "message"` 条目及其 `message.role`。 |
| `show-me-your-work` | 对照本次运行的 Pi 会话文件（见 Tool actions）检查日志，而不是 Claude Code transcript。 |
| `setup-pstack` | Pi 配置表是 Pi agent 目录下的 `pstack-models.md`（见 Session routing），由扩展加载，因此不需要 include 行。用 `pi --list-models` 列出模型。对于未列出模型的模型家族，提供一行 `pi models:`（见 Model names）。角色行和五个 effort 级别完全相同。`session hook` 行在 Pi 上同样适用，`off` 会让扩展停止加入指令。会话运行在 `anthropic` 上时，告诉用户 Pi 按 token 以额外用量计费 Claude，且每个多模型小组都会成倍增加这一成本。 |
| `no-comments` | `pstack:comment-sicko` 按原样通过 `agent` 解析。传给它的是范围，而不是让它运行 `/no-comments` 的请求；运行时不要用 `isolation: "worktree"`，因为 worktree 从 `HEAD` 开始，缺少该范围内未提交的改动。 |
| `teach` | 并行运行 `how` 和 `why` 映射为两个后台 `agent` 调用。Pi 没有图像生成工具，因此用 Mermaid 或纯文本绘图。 |
| `create-verification-skill` | 在 Claude Code 上生成的技能位于 `.claude/skills/verify/`。改为写到 Pi 能发现项目技能的位置，即 `.pi/skills/verify/` 或 `.agents/skills/verify/`。驱动应用的测试框架与平台无关。 |
| `maintain-verification-skill` | 按功能并行读取源码的 reader 映射为后台 `agent` 调用。项目本地技能位于 `.pi/skills/` 或 `.agents/skills/`，而不是 `.claude/skills/`。 |
| `create-formal-verification-skill` | 把生成的技能写到 Pi 能发现项目技能的位置，即 `.pi/skills/<area>/` 或 `.agents/skills/<area>/`，并以它验证的领域命名。验证器及其运行命令与平台无关。 |
| `maintain-formal-verification-skill` | 项目技能（包括形式化验证技能）位于 `.pi/skills/` 或 `.agents/skills/`。 |
| `babysit` | `loop` 和 `AskUserQuestion` 通过上面的表格解析。 |
| `automate-me` | `plugin-dev:skill-development` 通过上面的技能表解析。工作区的 transcript 就是它的 Pi 会话目录（见 Tool actions）。Pi 不会发现 `.claude/skills/`，因此改为把模式技能写到 `.pi/skills/<handle>-mode/SKILL.md` 或 `.agents/skills/<handle>-mode/SKILL.md`。 |
| `architect` | runner 小组经由 the **arena** skill 运行，因此它的 `agent` 扇出和模型别名同样适用。 |
| `arena` | 并行候选和交叉评审是使用已配置别名的后台 `agent` 调用（见 Model names）。 |
| `how` | 并行探索者和讲解者是 `agent` 调用，`readonly` 按原样适用（见 Subagent policy）。 |
| `reflect` | 三个评审者和综合者是后台 `agent` 调用。transcript 查找器也能读取 Pi 会话。传入本工作区的 Pi 会话目录（见 Tool actions）代替 Claude Code 的 projects 目录。它沿会话的活动分支回溯到开场 prompt。技能文件从 `pi list` 显示的 Pi 包目录加载，而不是 `~/.claude/plugins/`，因此把该目录下的读取视为插件技能读取。 |
| `swarm` | 每个 worker 是使用已配置别名的后台 `agent` 调用。给每个写入的 worker 设置 `isolation: "worktree"` 或独立的输出目录（见上文 Subagent policy）。 |
| `why` | 并行调查者和综合者是后台 `agent` 调用。从 Pi 向会话暴露的工具或 `pi mcp list` 中列出 MCP 服务器，而不是从 `.mcp.json` 或 `claude mcp list`。只有在每个调查者的完成通知都到达后才派生综合者，因为这些通知携带了综合者需要的发现。对调查者发 `send_message` 不会返回发现，所以要等它的通知。 |

## Vendored scripts

`skills/poteto-mode/scripts/` 附带 `watch-pr` PR 监视器、`orch` 存储 CLI 和 `worktree-audit.mjs`。这些脚本使用 bun 和 Node.js，在 Pi 上运行方式相同；通过 `bash` 调用。它们需要 `bun` 和 `gh`，forge 为 Origin 时需要 `origin`。按其 shebang 用 `node` 运行 `worktree-audit.mjs`。审计会用 `stat` 跟随 worktree 各祖先目录中的每个链接，并按链接最终指向的目录匹配，bun 和 node 下行为一致。bun 无法解析自身路径的 worktree 会丢失最近一次对话，归入 `review`。悬空或循环的链接会被忽略。其他任何 `stat` 无法跟随的链接，会把该链接所在目录下的所有 worktree 归入 `review`，并给出指明该链接的警告。`worktree-audit.mjs` 在每个存在的运行时 transcript 目录中查找每个 worktree 的最近一次对话：`$CLAUDE_CONFIG_DIR`（默认 `~/.claude`）下 Claude Code 的 projects，`$CODEX_HOME`（默认 `~/.codex`）下 Codex 的 sessions 和 archived sessions，以及 `$PI_CODING_AGENT_DIR`（默认 `~/.pi/agent`）下 Pi 的 `sessions/` 和 pstack 扩展的 subagent 会话。要扫描其他目录，把 transcript 目录放在仓库路径之后传入。它从 `skills/reflect/scripts/find-transcript.mjs` 导入 transcript 遍历器，因此请让 `reflect` 技能与 `poteto-mode` 一起安装。

## Instructions file

pstack 技能提到 "your instructions file" 时，在 Pi 上指 `AGENTS.md` 或 `CLAUDE.md`，Pi 从工作目录及其每一级父目录加载它们，另外对所有目录加载 `~/.pi/agent/AGENTS.md`。在 Claude Code 上指 `CLAUDE.md`。
