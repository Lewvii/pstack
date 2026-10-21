# pstack 的 GitHub Copilot 工具映射

pstack 技能用 Claude Code 的工具语言写成（the `Skill` tool、the `Agent` tool、`AskUserQuestion`、Claude 模型名、`~/.claude/` 路径）。在 GitHub Copilot 上技能文件完全相同，只是工具名、模型 ID 和路径的解析方式不同。当 pstack 技能提到某个 Claude 工具、驱动或内置技能、Claude 模型或 Claude Code 路径时，阅读本文件。它同时适用于 Copilot CLI 和 Copilot app，两者加载同一个插件。本文件仅适用于 Copilot；其他运行时使用各自的对应项。

## Tool actions

| pstack / Claude action | GitHub Copilot 对应 |
|------------------------|---------------------------|
| Read a file (`Read`) | `view` |
| Create a file | `create` |
| Edit a file | `edit` |
| Run a shell command | `bash` |
| Search file contents / find files | `grep` / `glob` |
| Fetch a URL | `web_fetch` |
| Search the web | `web_search` |
| Invoke a skill (the `Skill` tool, `/command`, `pstack:<skill>`) | 使用 `skill` 工具并传入裸技能名（`poteto-mode`，而不是 `pstack:poteto-mode`）。用户在 CLI 提示符中输入 `/pstack:<skill>`；在 CLI 1.0.89 上，对插件技能使用裸 `/<skill>` 会报告未知命令。Copilot 的提示词只列出大型插件的部分技能（字符预算会截掉按字母排序的末尾部分）；不在该列表中的技能仍可按名称加载。 |
| Dispatch a subagent (the `Agent`/`Task` tool) | `task` 工具：`agent_type`、`model`、`mode`、`name`、`prompt`。 |
| Dispatch N parallel subagents in one turn | 在一次响应中发出 N 个 `task` 调用。 |
| Background subagent (`run_in_background: true`) | `task` 配合 `mode: "background"`；完成时你会收到通知。 |
| Read a subagent's result / message a running one | `read_agent` / `write_agent`；`list_agents` 可找回丢失的 ID。 |
| Track tasks (the todolist; `TaskCreate` / `TaskUpdate`, or `TodoWrite` on Claude Code) | 用 `sql` 工具操作会话数据库内置的 `todos` 表（排序依赖用 `todo_deps`）。 |
| Ask the human a fixed-choice question (`AskUserQuestion`) | `ask_user` 配合 `choices` 列表，每次调用一个问题。若 `ask_user` 改为接受 elicitation 表单（`message` 和 `requestedSchema`，如 Copilot CLI 1.0.89），则用一个 `oneOf` 字符串属性作为选项列表。只把它当作单选，因为 Copilot app 的 `ask_user` 不支持多选，且 UI 会自行添加一个自由文本选项。多选用连续提问来模拟。 |
| Search Claude Code transcripts under `~/.claude/projects/` | Copilot 会话状态，限定在当前工作区；见下文 Transcripts。 |

## Subagent policy

poteto-mode 的 Subagents 一节设定了 Claude 专属默认值（`subagent_type: "pstack:poteto-agent"`、`run_in_background: true`）。在 GitHub Copilot 上：

- 插件的 agent 以带命名空间的名称作为 `task` agent 类型加载：`pstack:poteto-agent`、`pstack:comment-sicko`、`pstack:thermo-nuclear-review-subagent` 和 `pstack:thermo-nuclear-code-quality-review-subagent`。把技能的 `subagent_type` 原样作为 `agent_type` 传入。
- `subagent_type: "general-purpose"` 映射为 `agent_type: "general-purpose"`。技能未指定类型时，使用 `pstack:poteto-agent`。
- `readonly: true`（评审者、探索者、批评者）映射为只读 agent 类型：读代码用 `explore`，评审 diff 用 `code-review`。当只读角色必须运行命令或访问 MCP 工具时，使用 `general-purpose` 并在 prompt 中告诉它不要写文件；无论如何都要评审它的 diff。
- `readonly: false`，或需要 MCP 工具的角色，映射为 `general-purpose` 或 `pstack:poteto-agent`。
- 推理强度映射为 `task` 的 `reasoning_effort`。传入角色值的 `@<level>` 后缀，否则传入配置表的 `default effort` 级别，作为 `reasoning_effort`；`session` 不传。不要派发 Claude Code 用于 effort 的 `pstack:effort-<level>` 或 `pstack:poteto-agent-<level>` agent 类型，改用 `general-purpose` 或 `pstack:poteto-agent` 并配合 `reasoning_effort`。
- `run_in_background: true` 映射为 `mode: "background"`。`mode: "sync"` 会阻塞你的回合；只在下一步需要其结果时使用。
- 技能为某个角色指定的模型（其 Models 一节、`Agent` 调用的 `model`，或用户的配置表行）是明确的模型指令，因此作为 `task` 的 `model` 传入。没有配置模型的角色省略 `model`。Claude 默认值如何解析见下文 Model names。
- 在 CLI 上写入的 worker 共享本机，因此 the **swarm** skill 的 worker 和扇出类 playbook（`orchestrate`、`autopilot-full`、`autopilot-stack`）与 Claude Code 一样用 worktree 隔离写入者。在 Copilot app 中，改用 `create_session` 为每个写入者提供各自的 worktree 会话；见下文 App and CLI。
- 策略的其余部分保持不变。传递文件指针而不是内联上下文，写入的 worker 各自使用独立的 worktree 或分支，并亲自评审每个 subagent 的 diff。

@@MODEL_NAMES@@
## Session routing hook

插件的 `SessionStart` hook 在 Copilot CLI 和 Copilot app 中运行。Copilot 通过它自己的 Copilot manifest 和 hook 文件加载插件，而不是 Claude Code 的那些。hook 把路由指令作为 Copilot 注入的 `additionalContext` JSON 输出。它读取 `${COPILOT_HOME:-~/.copilot}/pstack-models.md`，而会话本身读取该文件需要路径访问授权。`session hook: off` 关闭注入。当配置表通过 `check-sheet.sh` 所做的检查时，注入的上下文末尾按角色各附一行，由检查过的值重建；配置表的其他文本不会进入上下文。检查失败时，上下文会写明 `sheet invalid` 并列出每个问题。没有配置表时也会注明。无论哪种情况，它都会告诉会话在第一个按角色模型派发的技能之前运行 `setup-pstack`。指令以 `pstack:<skill>` 的形式提及技能；在 Copilot 上按裸名称加载它们。

如果会话中缺少该指令（仅安装技能、Copilot 版本丢弃了插件 hook 上下文，或其他插件的 hook 替换了它），把 `setup-pstack` 描述的常驻指令添加到 `~/.copilot/copilot-instructions.md`，或显式请求 `poteto-mode`。

## Plugin file access

Copilot 把文件访问限制在工作区和系统临时目录内，因此从已安装插件中读取 playbook、参考文档或脚本，每个会话都会请求路径访问，而不带 `--allow-all-paths` 的 `-p` 运行会直接拒绝。插件的 `PreToolUse` hook 会批准两类调用。

- 对 `COPILOT_PLUGIN_ROOT` 内、不含 `.` 或 `..` 路径段的路径执行 `view`。
- 以绝对路径运行 `COPILOT_PLUGIN_ROOT/skills/` 下已登记脚本、并使用其文档规定参数的 `bash`。已登记的脚本有 `log.sh`（用 `bash` 运行）、`check-sheet.sh`（用 `sh` 或 `bash` 运行），以及 `find-transcript.mjs`、`resume.mjs`、`check-plan.mjs`、`check-playbooks.mjs` 和 `worktree-audit.mjs`（用 `node` 运行）。也接受直接执行。hook 会检查每个脚本的参数约定：例如 `log.sh` 的六个参数中只有第一个是文件名，其余是自由文本。Resume 先接受操作名，然后视情况接受 `--project`、`--note` 和可重复的 `--artifact` 路径，可写作 `--option=value` 或 `--option value`。

每个参数要么是由字母、数字和 `_ . / : = @ % + , -` 组成的普通单词，要么是内部不含引号的单引号字符串。路径操作数不得用 `..` 向上跳出，并且在跟随目录符号链接后必须解析到工作区内；位于插件内的路径走正常的权限流程，因此任何获批的脚本都不能改写每个会话加载的上下文。文件符号链接和悬空符号链接走正常的权限流程。命令中不得包含 `; | & $`、反引号、`< > ( ) \`、双引号或换行。每次调用只运行一条命令，不使用管道或重定向。未知脚本、无法识别的参数形式以及不匹配的解释器都走正常的权限流程。

其他所有调用，包括以任何其他形式运行脚本、以及在插件目录树中执行 `grep` 或 `glob`，都走正常的权限流程。请求绕过沙箱的命令也是如此，因为无论 hook 如何回答，Copilot 都会就绕过请求进行提示。当插件位于工作区内或工作区位于插件内时，hook 不会批准脚本，因为会话可能先编辑脚本再运行它。在该 hook 不运行的情况下（仅安装技能，或禁用了 hook），用 `--add-dir <plugin directory>` 启动 CLI，或在会话中运行 `/add-dir <plugin directory>`；从 GitHub marketplace 安装的插件位于 `${COPILOT_HOME:-~/.copilot}/installed-plugins/<marketplace>/pstack`。

## App and CLI

Copilot app 加载与 CLI 相同的插件，并增加了会话级工具。工具列表中有 app 原语时使用它；否则使用 CLI 的替代方案。

| pstack 需求 | Copilot app | Copilot CLI |
|-------------|-------------|-------------|
| 隔离的写入者（swarm worker、stack 层、orchestrate worker） | `create_session`，每个写入者一个 worktree 会话 | 每个写入者执行一次 `git worktree add`，然后把 `task` worker 指向它 |
| 周期性唤醒（`loop`、`/loop` 审计节拍、babysit 的节奏） | 在本会话上使用 `save_session_automation` | 每个回合自行重跑该步骤，或请用户重新调用；在决策记录中写明节奏 |
| 协调多个会话或仓库（`orchestrate`、`autopilot-full`） | app 的 `orchestrate` 技能加上 `send_session_message` | 配合 worktree 的 `task` 扇出；并行 subagent 用 `/fleet` |
| 堆叠 PR（`autopilot-stack`、发布一个 stack） | app 的 `pr-stack` 技能，每层一个子会话 | `gh` 或 `origin`，每层一个 worktree |
| 监视 PR 直到合并（`babysit`、`watch-pr`） | 会话上的 Agent merge，或 `babysit` 配合 `save_session_automation` | `babysit` 配合内置的 `watch-pr` 脚本和 `gh` |
| 驱动 UI | `browser` 画布，或你的浏览器自动化 | 你的浏览器自动化，或交给用户一项具体的手动检查 |

## Transcripts

Claude Code 在 `~/.claude/projects/<encoded-cwd>/` 下为每个项目保留一个目录。Copilot 在 `${COPILOT_HOME:-~/.copilot}/session-state/<session-id>/` 下为每个会话保留一个目录，所有项目放在一起：

- `events.jsonl` 是 transcript。第一条记录是带 `data.context.cwd` 的 `session.start`；用户回合为 `{"type":"user.message","data":{"content":...}}`。
- `workspace.yaml` 写明会话的 `cwd`。每次 transcript 搜索都要限定在当前工作区：只保留 `cwd` 为工作区路径（或其 worktree）的会话目录，然后读取它们的 `events.jsonl`。绝不读取其他项目的会话。
- 技能提到 "this session's transcript" 时，指 `events.jsonl` 中包含开场 prompt 的、最新的匹配会话目录。

在 poteto-mode 的 playbook 中，`eval` 从 Copilot 会话状态读取候选 transcript，`session-pickup` 和 `worktree-cleanup` 在限定于工作区的范围内搜索它。`orchestrate` 和 `multi-phase-plan` 把存储放在 `${COPILOT_HOME:-~/.copilot}/orchestrate/` 下，而不是 `~/.claude/orchestrate/`。

## Driver and bundled skills pstack references

[驱动策略](../SKILL.md#non-negotiables) 决定使用哪个应用驱动。对于这些工作流提到的技能和驱动，使用以下 Copilot 对应项：

| pstack 中提到的技能或驱动 | 在 GitHub Copilot 上 |
|---------------------------------|-------------------|
| `run`（驱动 CLI/TUI 以观察变更生效） | 通过 `bash` 自行运行应用并观察真实输出。 |
| 项目 UI 驱动 | 用你手头的任何自动化手段驱动 UI（app 的 `browser` 画布、浏览器技能），或交给用户一项具体的手动检查。未观察到产物前不要声称完成。 |
| `plugin-dev:skill-development`（Claude 的 SKILL.md 编写指南） | 遵循 Agent Skills 约定：带 `name` + `description` frontmatter 的 `SKILL.md`、渐进式披露到 `references/`，以及 poteto-mode 的 [authoring-a-skill playbook](../playbooks/authoring-a-skill.md)。 |
| `loop`（循环或自定节奏的重复调用，`babysit` 使用） | Copilot 没有 `loop` 技能。在 app 中使用 `save_session_automation`；在 CLI 上按固定节奏自行重跑该步骤。 |

## Per-skill notes

受影响的技能入口指向这里。大多数技能只需要上面的表格。以下技能还需要一项额外映射：

| Skill | On GitHub Copilot |
|-------|-------------------|
| `interrogate` | 每个评审者是一次 `task` 调用：`subagent_type` 变为 `agent_type`，`readonly: true` 变为只读 agent 类型（见 Subagent policy），`model` 取自配置表。保持小组成员来自不同厂商。 |
| `setup-pstack` | 该技能的 Other runtimes 表给出 Copilot 配置表路径及其加载方式。其 [Copilot setup questions](../../setup-pstack/copilot.md) 从 `task` 工具的 `model` 枚举中列出模型，按档位和小组席位每次用一个 `ask_user` 问题提问，按厂商分组，不推荐任何模型；第一个问题允许只有一个模型的方案，或不想逐角色选择的人，让所有角色都使用会话的模型。角色行完全相同；`session hook` 行同样控制 Copilot hook。 |
| `architect` | runner 小组经由 the **arena** skill 运行：每个 runner 一个 `task`，各自使用配置表 `architect runners` 行中的一个模型。 |
| `how` | 探索者和讲解者是使用配置表中 `how` 角色模型的 `task` 调用；批评者使用只读 agent 类型（见 Subagent policy）。 |
| `arena` | runner 和交叉评审池取自配置表的小组行；交叉评审要选与被评候选不同厂商的模型。 |
| `no-comments` | 派发 `task`，使用 `agent_type: "pstack:comment-sicko"`。 |
| `thermos` | 派发两个 `mode: "background"` 的 `task` 调用，分别使用 `agent_type: "pstack:thermo-nuclear-review-subagent"` 和 `agent_type: "pstack:thermo-nuclear-code-quality-review-subagent"`，各自的 prompt 中带上相同的 diff 和文件内容。 |
| `swarm` | worker 是 `mode: "background"` 的 `task` 调用，各自在自己的 worktree 上；在 app 中，每个 worker 一个 `create_session`。 |
| `teach` | 并行运行 `how` 和 `why` 映射为 `task` 扇出；图像生成使用你手头的任何图像工具，或者跳过并说明。 |
| `why` | 可用的 MCP 服务器出现在你的工具列表中；没有可供扫描的 `mcp__` 前缀约定。 |
| `reflect` | 运行 `find-transcript.mjs <session-state directory> '<opening prompt fragment>' <workspace directory>`，传入 `${COPILOT_HOME:-~/.copilot}` 下会话状态的绝对路径。省略工作区时默认为当前目录。查找器在读取 Copilot 的 `user.message` 记录前，会先用会话头中的 `cwd` 与该工作区比对，因此会跳过其他项目中匹配的 prompt。若对话位于相关 worktree 中，显式传入该 worktree 的目录。评审者参考中提到的 `.claude/skills/` 和 `~/.claude/` 路径，指下文的 Copilot 技能位置。 |
| `recall` | 搜索限定于当前工作区的 Copilot 会话状态（见上文 Transcripts），而不是 `~/.claude/projects/`。 |
| `show-me-your-work` | 运行结束时的检查读取本会话的 `events.jsonl`（见上文 Transcripts）。 |
| `automate-me` | 模式技能位于下文的 Copilot 技能位置；transcript 来自上文 Transcripts；`plugin-dev:skill-development` 通过技能表解析。 |
| `create-verification-skill` | 在 Claude Code 上生成的技能位于 `.claude/skills/verify/`；在 Copilot 上写到 `.github/skills/verify/`。驱动应用的测试框架与平台无关。 |
| `maintain-verification-skill` | 按功能并行读取源码的 reader 映射为 `task` 扇出；项目本地技能位于 `.github/skills/`，而不是 `.claude/skills/`。 |
| `create-formal-verification-skill` | 在 Copilot 上把生成的技能写到 `.github/skills/<area>/`，并以它验证的领域命名。验证器及其运行命令与平台无关。 |
| `maintain-formal-verification-skill` | 项目技能（包括形式化验证技能）位于 `.github/skills/`。 |
| `babysit` | `loop` 和 `AskUserQuestion` 通过上面的表格解析；在 app 中，节奏由 `save_session_automation` 提供。 |

## Vendored scripts

`skills/poteto-mode/scripts/` 附带 `watch-pr` PR 监视器、`orch` 存储 CLI、`resume.mjs` 和 `worktree-audit.mjs`。它们使用 bun 和 Node.js，在 Copilot 上运行方式相同；按上文 Plugin file access 描述的形式通过 `bash` 调用。它们需要 `bun` 和 `gh`，forge 为 Origin 时需要 `origin`。`worktree-audit.mjs` 在 `~/.claude/projects/` 下的 Claude Code transcript、Codex 和 Pi 会话，以及 `${COPILOT_HOME:-~/.copilot}/session-state` 下的 Copilot 会话中查找每个 worktree 的最近一次对话。要扫描其他目录，把 transcript 目录放在仓库路径之后传入。它从 `skills/reflect/scripts/find-transcript.mjs` 导入 transcript 遍历器，因此请让 `reflect` 技能与 `poteto-mode` 一起安装。

## Instructions file

pstack 技能提到 "your instructions file" 或 `CLAUDE.md` 时，在 Copilot 上指仓库中的 `AGENTS.md` 或 `.github/copilot-instructions.md`，以及对所有仓库生效的 `~/.copilot/copilot-instructions.md`。Copilot 也会读取仓库的 `CLAUDE.md`。Copilot 不会展开用户级指令中的 `@~/` include，因此插件 hook 改为注入模型配置表的角色行，如上文 Model names 所述。

## Skill locations

pstack 技能提到 `.claude/skills/` 时，在 Copilot 上项目技能位于 `.github/skills/`（Copilot 也读取 `.claude/skills/` 和 `.agents/skills/`），个人技能位于 `~/.copilot/skills/` 或 `~/.agents/skills/`。
