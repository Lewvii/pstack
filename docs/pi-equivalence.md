# Pi 等价性

本表列出 pstack 插件依赖的每一个 Claude Code 机制，做法是扫描 `plugins/pstack/` 中的工具名、agent 字段、hook、路径、内置技能、CLI 和模型名。每一行说明 Pi 1.1 如何提供同样的行为，以及由什么来证明。当某行缺少合法状态、某个 `extension` 或 `script` 行点名了不存在的测试，或某个 `difference` 行没有给出原因时，`tests/pi-equivalence.test.mjs` 会失败。

状态含义：

- `native`：Pi 无需 pstack 代码即可完成。
- `extension`：[pstack Pi 扩展](../plugins/pstack/pi/index.ts)完成，由点名的离线测试证明。
- `script`：自带的脚本读取 Pi 的布局，由点名的测试证明。
- `mapping`：技能已在正文中描述了该行为，[pi-tools.md](../plugins/pstack/skills/poteto-mode/references/pi-tools.md)点名了 Pi 的等价物。
- `difference`：Pi 的行为不同，原因一栏说明为什么没有技能依赖它得出正确结果。
- `n/a`：插件提到了它，但没有行为依赖它。

Live 一栏记录在真实 Pi 上以 `PSTACK_PI_LIVE=1` 运行的端到端结果（`tests/pi/live.test.mjs`）。Live 值为 `n/a` 表示实跑没有覆盖该行。执行实跑的机器只在 `openai` 提供方上有 ChatGPT 登录，所以族名通过自带的 `openai` 表解析（X01b）。`anthropic/*` ID（X01a）和 `openai-codex/*` ID（X01c）从未实跑调用；`tests/pi/catalog.test.mjs` 对照已安装 Pi 的模型目录检查这两张表。

| ID | Claude Code 机制 | 在 Pi 上 | Status | Evidence | Live |
| --- | --- | --- | --- | --- | --- |
| T01 | `Agent` 工具派发 | `agent` 工具运行一个子 `pi --mode rpc` 进程，并把 prompt 作为 RPC 命令写入其 stdin | extension | `foreground waits, returns the final text and agentId, and runs the exact child command` | VERIFIED |
| T02 | `subagent_type` | 解析为插件的 agent 文件；未知类型会列出全部合法类型 | extension | `unknown subagent_type is an error listing every valid type` | VERIFIED |
| T03 | 逐调用的 `model` | 族名通过 `models.json` 和覆盖表的 `pi models:` 行解析 | extension | `alias, sheet override, inherit-parent, auto, pass-through, and no parent model` | VERIFIED |
| T04 | `run_in_background` | 立即返回 agent id；进程退出后，完成通知在下一个工具边界并入运行中的回合，或在其空闲时开启一个回合 | extension | `background returns at once and the completion notice comes only after the process exits` | VERIFIED |
| T05 | `readonly`（Cursor 时代的措辞，不是 Claude Code 参数） | `readonly: true` 让子进程在没有 `edit` 和 `write` 工具的情况下运行 | extension | `readonly runs the child without the edit and write tools` | n/a |
| T06 | 每个写者一个 worktree | `isolation: "worktree"` 在 `.claude/worktrees/` 下创建 worktree，并且只在 agent 没有留下任何东西时才连同其分支一起删除：一个改动、它写下的 gitignore 文件，或它分支上的一个 commit 或它的 HEAD 曾访问过的任何 commit（reflog）都会保留它；保留的路径只在恢复时、且它仍是同一仓库的链接 worktree（自身即 Git 顶层、共享仓库的公共 Git 目录、并被 `git worktree list` 列出）时才被复用，该路径上的其他任何东西都会让启动失败而不删除它；手工 `git worktree` 通过 `bash` 工作 | extension | `a commit made on a detached HEAD and left behind by checking the branch out again is kept, through the reflog` | VERIFIED |
| T07 | 给 agent 发消息（`SendMessage`） | 给运行中 agent 的 `send_message` 是子进程 stdin 上的 RPC `steer`，agent 在当前工具调用之后于同一次运行中读取，因此只报告一次；给已完成、或已沉降并正在退出的 agent 的消息，会在进程退出后恢复同一个 Pi 会话、模型与思考档位 | extension | `a message to a running agent reaches that run before it exits, and the agent reports once` | VERIFIED |
| T08 | 停止 agent 并确认已停止 | `stop_agent` 结束子 pi，后者会杀掉它正在运行的 bash 命令，并且只在退出之后报告 `stopped`。已结束的 bash 命令留在后台的进程运行在自己的进程组中，会活得比 agent 更久。 | extension | `ends the child and the bash command it has running, not a process a finished command left behind, and reports stopped once the child has exited` | VERIFIED |
| T09 | agent 列表与完成通知 | `list_agents`；状态跟随进程，所以 `completed` 始终意味着已退出 | extension | `list_agents survives a reload through the persisted entries` | VERIFIED |
| T10 | 延迟工具加载（`ToolSearch`） | 每个 pstack 工具在启动时以 `model-only` 暴露注册，所以 Pi 从第一个回合起就向模型声明它，在任何 `codemode.mode` 下都是如此，`tool_search` 永远不需要加载它 | extension | `every pstack tool is model-only, so Pi declares it to the model in every codemode mode and never defers it` | VERIFIED |
| T11 | `Skill` 工具与 `/command` | Pi 在系统提示中列出技能；`/skill:<name>` 强制调用其中一个 | native | Pi docs/skills.md | n/a |
| T12 | `pstack:` 技能命名空间 | 技能以裸名加载，并以 `/skill:<name>` 运行；agent 类型保留 `pstack:` | difference | 只是命令拼写。技能在正文中以裸名互相点名，agent 类型带前缀解析。 | n/a |
| T13 | `AskUserQuestion` | 形状相同的 `ask_user_question`；没有 UI 时，以及在子 agent 中，它会失败并让模型改用纯文本提问 | extension | `a single-select question returns the chosen label` | n/a |
| T14 | `TaskCreate`/`TaskUpdate`/`TodoWrite` | poteto-mode 自带的 `todo.md` 回退 | mapping | poteto-mode Platform Adaptation；pi-tools.md Tool actions | n/a |
| T15 | `Read`/`Edit`/`Write`/`Bash`/`Grep`/`Glob` | `read`、`edit`、`write`、`bash`；启用时用 `grep`、`find`、`ls` | native | pi-tools.md Tool actions | n/a |
| T16 | `WebFetch`/`WebSearch` | 通过 `bash` 用 `curl`；网络搜索通过 MCP 服务器 | difference | 没有任何技能点名这两个工具。抓取通过 `bash` 即可工作。 | n/a |
| T17 | `ScheduleWakeup`（其他具名工具从未被引用） | 形状相同的 `schedule_wakeup` | extension | `clamps to 60 s and fires the prompt as a follow-up user message` | VERIFIED |
| T18 | MCP 工具发现 | Pi 内置的 MCP；从会话的工具或 `pi mcp list` 列出服务器 | mapping | pi-tools.md `why` note | n/a |
| T19 | 系统提示中点名的事实 | Pi 的系统提示列出工具与技能；会话记录目录在 pi-tools.md 中给出 | mapping | pi-tools.md `reflect` note | n/a |
| T20 | 图像生成工具 | 没有，与 Claude Code 相同；用 Mermaid 或文字绘制 | difference | Claude Code 也没有这类工具，所以 `teach` 本就有回退方案。 | n/a |
| T21 | 嵌套 agent，最多低于主对话三层 | 第三层的子进程在没有 `agent` 工具的情况下运行 | extension | `agents nest at most three layers below the main session, as on Claude Code` | n/a |
| A01 | `pstack:poteto-agent` | 从 `agents/poteto-agent.md` 解析 | extension | `every pstack agent type reaches its child with its own agent file as the system prompt` | VERIFIED |
| A02 | 推理档位 agent | `effort:` frontmatter 成为子进程的 `--thinking`；没有档位则继承父级 | extension | `an effort agent passes its body as a 0600 system prompt file and its effort as --thinking; others run at the parent's level` | VERIFIED |
| A03 | `pstack:comment-sicko` | 从 `agents/comment-sicko.md` 解析 | extension | `every pstack agent type reaches its child with its own agent file as the system prompt` | n/a |
| A04 | 清单 `agents[]` 注册 | 扩展在运行时读取 `agents/` 和 `effort-agents/` | extension | `every pstack agent type reaches its child with its own agent file as the system prompt` | n/a |
| A05 | `general-purpose` agent 类型 | 没有 agent 文件的子进程 | extension | `foreground waits, returns the final text and agentId, and runs the exact child command` | n/a |
| A06 | agent 文件正文作为子进程的系统提示 | 用 `--append-system-prompt` 追加，使 Pi 的技能列表对子进程保持可见 | extension | `an effort agent passes its body as a 0600 system prompt file and its effort as --thinking; others run at the parent's level` | VERIFIED |
| H01 | `SessionStart` hook | 每次 agent 启动时都加入路由指令，因此它挺过恢复、清空和压缩 | extension | `injects the mandate and the full sheet on every agent start` | VERIFIED |
| H02 | `CLAUDE_PLUGIN_ROOT` | 扩展从自身文件路径定位插件 | extension | `injects the mandate when there is no sheet` | n/a |
| H03 | `~/.claude` 配置目录 | `$PI_CODING_AGENT_DIR` 或 `~/.pi/agent` | mapping | setup-pstack Other runtimes | n/a |
| H04 | 覆盖表加 `@` include | 扩展把整张表加入系统提示，子进程也一样 | extension | `a child pi gets the sheet but not the mandate` | VERIFIED |
| H05 | `CLAUDE.md` 指令文件 | Pi 按目录加载 `AGENTS.md` 或 `CLAUDE.md`，外加 `~/.pi/agent/AGENTS.md` | native | pi-tools.md Instructions file | n/a |
| H06 | `.claude/settings.local.json` 为 todo 工具设置的 env | Pi 没有 todo 工具；适用 `todo.md` 回退 | n/a | poteto-mode Platform Adaptation | n/a |
| F01 | `~/.claude/projects` 会话记录 | `find-transcript.mjs` 读取 Pi 会话并跟随活跃分支 | script | `a Pi session's opening prompt is the first user message on the branch that ends at the last entry` | VERIFIED |
| F07 | 自己的会话记录作为证据 | 同一个读取器；`worktree-audit.mjs` 扫描 Pi 会话根目录 | script | `a Pi session in a second transcripts root marks the worktree it ran in as a recent chat` | n/a |
| F02 | `.claude/skills/` 项目与用户技能 | `.pi/skills/` 或 `.agents/skills/` | mapping | pi-tools.md `create-verification-skill` note | n/a |
| F03 | `~/.claude/plugins/` 安装路径 | 技能文件从 `pi list` 显示的 Pi 包目录加载 | mapping | pi-tools.md `reflect` note | n/a |
| F04 | `~/.claude/orchestrate/<slug>/` 存储 | 一个普通目录，原样使用 | native | orchestrate playbook | n/a |
| F05 | `~/.claude/shell-snapshots/` 清理建议 | 仅清理建议 | n/a | worktree-cleanup playbook | n/a |
| F06 | `.claude/worktrees` | 扩展使用同一路径 | extension | `keeps a worktree with changes and reports its path and branch` | n/a |
| S01 | 内置 `run` 技能 | 通过 `bash` 运行应用 | mapping | pi-tools.md Driver and bundled skills | n/a |
| S02 | `verify` 技能 | 按路径读取项目技能，或把它加入 Pi 的 `skills` 设置 | mapping | pi-tools.md Driver and bundled skills | n/a |
| S03 | `loop` 技能 | `/loop [interval] <prompt>` 与自定节奏的 `schedule_wakeup` | extension | `a fixed interval runs now and then on every interval until /loop stop` | VERIFIED |
| S04 | `plugin-dev:skill-development` | Pi 的技能文档与 Agent Skills 规范 | mapping | pi-tools.md Driver and bundled skills | n/a |
| S05 | 内置 `babysit` 的消歧 | Pi 没有内置 `babysit`，所以只有 pstack 的那一个 | n/a | poteto-mode Non-negotiables | n/a |
| S06 | `superpowers`、`code-review`、`simplify`、`security-review` | Pi 上不存在 | difference | 仅提及。hook 文本让位于其他指令，没有工作流要求这些技能。 | n/a |
| S07a | frontmatter `user-invocable: false` | Pi 忽略它，所以 23 个 `principle-*` 技能会出现在 `/skill:` 补全中 | difference | 仅菜单可见性。模型仍按名字加载每个原则技能，与 Claude Code 相同。 | n/a |
| S07b | frontmatter `paths:` 自动加载 | 扩展在第一次匹配的文件读取或编辑时注记该技能 | extension | `the first matching file gets a one-line note naming the skill, once per session` | n/a |
| S07c | frontmatter `disable-model-invocation` | 被 Pi 尊重；没有 pstack 技能设置它 | native | Pi docs/skills.md | n/a |
| C01 | `gh` | 通过 `bash` 使用同一个 CLI | native | pi-tools.md Vendored scripts | n/a |
| C02 | `origin` | 通过 `bash` 使用同一个 CLI | native | pi-tools.md Vendored scripts | n/a |
| C04 | `claude`/`codex` CLI | 仅正文提及（`claude mcp list`），在 `why` 注记中映射 | mapping | pi-tools.md `why` note | n/a |
| C05 | `bun` 脚本 | 通过 `bash` 相同 | native | pi-tools.md Vendored scripts | n/a |
| C06 | `node` 脚本 | 通过 `bash` 相同 | native | pi-tools.md Vendored scripts | n/a |
| X01a | Anthropic 会话或任何没有自有表的提供方上的族名 `opus`、`fable`、`sonnet`、`haiku` | `models.json` 的 `anthropic` 表把它们映射到 `anthropic/*` ID | extension | `a family name resolves in the table of the parent's provider, and in the fallback table on any other` | n/a |
| X01b | ChatGPT 登录（`openai`，Pi 的 Sign in with ChatGPT）上的族名 | `openai` 表把它们映射到与 Codex 默认值同档位的 GPT-6 模型 | extension | `a family name resolves in the table of the parent's provider, and in the fallback table on any other` | VERIFIED |
| X01c | 旧 OpenAI Codex 登录（`openai-codex`）上的族名 | `openai-codex` 表把它们映射到同样的 GPT-6 模型 | extension | `a family name resolves in the table of the parent's provider, and in the fallback table on any other` | n/a |
| X02 | 推理档位与 `@level` | 同样的五个档位，以 `--thinking` 传入 | extension | `an effort agent passes its body as a 0600 system prompt file and its effort as --thinking; others run at the parent's level` | VERIFIED |
| X03 | `inherit-parent` / `auto` | 子进程在父级当前模型上运行 | extension | `alias, sheet override, inherit-parent, auto, pass-through, and no parent model` | n/a |
| X04 | 压缩与 `/clear` | 每次 agent 启动都注入，从而保住路由指令 | extension | `injects the mandate and the full sheet on every agent start` | VERIFIED |
| X05 | 会话暂存区 | Pi 没有；技能只警告不要在那里保存持久状态 | difference | 唯一的引用（orchestrate）告诉 agent 不要使用暂存区。 | n/a |
