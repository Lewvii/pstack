# pstack 的 Codex 工具映射

pstack 技能用 Claude Code 的工具语言写成（the `Skill` tool、the `Agent` tool、`AskUserQuestion`、Claude 模型名）。在 Codex 上技能文件完全相同，只是工具名解析方式不同。当 pstack 技能提到某个 Claude 工具、驱动或内置技能、或 Claude 模型时，阅读本文件。本文件仅适用于 Codex。Gemini CLI、opencode、Prime Agent 及其他运行时必须使用各自具体的工具、模型名和配置路径。

## Tool actions

| pstack / Claude action | Codex 对应 |
|------------------------|------------------|
| Read a file | `shell`（`cat`、`head`、`tail`） |
| Create / edit / delete a file | `apply_patch` |
| Run a shell command | `shell` |
| Search file contents / find files | `shell`（`rg`、`grep`、`find`、`ls`） |
| Fetch a URL | `shell` 配合 `curl` / `wget` |
| Search the web | `web_search` |
| Invoke a skill (the `Skill` tool, `/command`) | 技能原生加载，按呈现的指令执行。 |
| Dispatch a subagent (the `Agent`/`Task` tool) | `spawn_agent` |
| Dispatch N parallel subagents in one turn | 在一次响应中发出 N 个 `spawn_agent` 调用 |
| Wait for a subagent result | `wait_agent` |
| Free a finished subagent slot | 会话提供 `close_agent` 时用它。部分 Codex 宿主不提供。 |
| Track tasks (the todolist; `TaskCreate` / `TaskUpdate`, or `TodoWrite` on Claude Code) | `update_plan` |
| Ask the human a fixed-choice question (`AskUserQuestion`) | 用纯文本提问，让用户作答。Codex 没有结构化选项工具。 |

派发 subagent 需要启用 `multi_agent`。在 `~/.codex/config.toml` 中加入：

```toml
[features]
multi_agent = true
```

未启用时 `spawn_agent` 不可用，扇出类技能（`interrogate`、`why`、`how`、`arena`、`reflect`）退化为单次顺序执行。必需的独立评审不会退化，而是保持阻塞，见 poteto-mode 的 [Subagents](../SKILL.md#subagents) 一节。

启用 `multi_agent` 并不会解除宿主的线程上限。`spawn_agent` 返回的线程上限错误是容量错误，不是模型 slug 被拒，因此技能中回退到其他模型的规则不适用。第一次遇到容量拒绝后，停止继续派生。重试前先用 `close_agent` 关闭已完成的 agent；会话没有 `close_agent` 时，不要假定已完成 agent 的槽位已释放。

## Subagent policy

poteto-mode 的 Subagents 一节设定了 Claude 专属默认值（`subagent_type: "pstack:poteto-agent"`、`run_in_background: true`）。在 Codex 上：

- 没有 `poteto-agent` 这一 subagent 类型。要让临时 subagent 遵循 poteto-mode 的风格，就派发一个 `spawn_agent`，在指令中要求它先完整阅读 `poteto-mode` 技能。
- `spawn_agent` 调用本来就与你的回合并发运行，所以 `run_in_background: true` 没有单独的标志。发出派发后继续工作即可。
- 没有 `pstack:effort-<level>` 或 `pstack:poteto-agent-<level>` 类型。当角色值带有 `@<level>`，或 `default effort` 行指定了级别时，把该级别作为 `spawn_agent` 的 `reasoning_effort` 传入，派发的其他部分保持不变。`session` 不传 `reasoning_effort`。
- 也没有 `comment-sicko` 这一 subagent 类型。the **no-comments** skill 在 Claude Code 上派生它；在 Codex 上派发一个 `spawn_agent`，在指令中要求它先完整阅读 `poteto-mode/references/agents/comment-sicko.md`。
- 没有 `thermo-nuclear-review-subagent` 或 `thermo-nuclear-code-quality-review-subagent` 类型。the **thermos** skill 在 Claude Code 上派生这两者；在 Codex 上派发两个 `spawn_agent` 调用，指令分别要求各自先完整阅读 `thermo-nuclear-review` 或 `thermo-nuclear-code-quality-review` 技能，并且只评审其 prompt 中给出的 diff 和文件内容。
- Claude Code 在本机运行所有 subagent，因此 the **swarm** skill 的 worker 和扇出类 playbook（`orchestrate`、`autopilot-full`、`autopilot-stack`）用 worktree 隔离写入者。Codex 上同样如此。
- 策略的其余部分保持不变。传递文件指针而不是内联上下文，写入的 worker 各自使用独立的 worktree 或分支，并亲自评审每个 subagent 的 diff。

@@MODEL_NAMES@@
## Session routing hook

原生 pstack 插件捆绑了与 Claude Code 插件相同的 `SessionStart` 路由指令，在 macOS 和 Linux 上通过 `session-start.sh`，在 Windows 上通过 `session-start.ps1`。用户通过 `/hooks` 信任该 hook 后，Codex 会在启动、恢复、清空和压缩时运行它。hook 从 Codex 配置表读取 `session hook`，路径见 [setup-pstack 的运行时表](../../setup-pstack/SKILL.md#other-runtimes)；`session hook: off` 关闭注入。

仅安装技能（skills-only）时不包含插件 hook。这种情况下请显式请求 `poteto-mode`，或在 `AGENTS.md` 中添加常驻指令。

## Driver and bundled skills pstack references

[驱动策略](../SKILL.md#non-negotiables) 决定使用哪个应用驱动。对于这些工作流提到的技能和驱动，使用以下 Codex 对应项：

| pstack 中提到的技能或驱动 | 在 Codex 上 |
|---------------------------------|----------|
| `run`（驱动 CLI/TUI 以观察变更生效） | 通过 `shell` 自行运行应用并观察真实输出。 |
| 项目 UI 驱动 | 用你手头的任何自动化手段驱动 UI，或交给用户一项具体的手动检查。未观察到产物前不要声称完成。 |
| `plugin-dev:skill-development`（Claude 的 SKILL.md 编写指南） | 遵循你所在平台的技能编写指南；若有 `writing-skills` 技能则使用它。保留 `name` + `description` frontmatter 和渐进式披露。 |
| `loop`（循环或自定节奏的重复调用，`babysit` 使用） | Codex 没有 `loop` 技能。按固定节奏自行重跑该步骤，或在可用时使用 Codex 定时任务。 |

## Per-skill notes

受影响的技能入口和可选的 Codex 斜杠存根都指向这里。大多数技能只需要上面的表格。以下技能还需要一项额外映射：

| Skill | On Codex |
|-------|----------|
| `interrogate` | `subagent_type`/`model`/`readonly` 派发字段映射到 `spawn_agent`；替换为你配置的 Codex 模型，并保持评审小组的模型多样性。 |
| `setup-pstack` | 该技能的 Other runtimes 表给出 Codex 配置表路径及其加载方式；slug 是你的 Codex 模型（见上文 Model names）。角色行完全相同。 |
| `no-comments` | 没有 `comment-sicko` subagent 类型；见上文 Subagent policy。 |
| `thermos` | 没有 thermo-nuclear subagent 类型；见上文 Subagent policy。 |
| `teach` | 并行运行 `how` 和 `why` 映射为 `spawn_agent` 扇出；图像生成使用已配置的 Codex 对应工具。 |
| `create-verification-skill` | 在 Claude Code 上生成的技能位于 `.claude/skills/verify/`；改为写到 Codex 的项目技能位置。驱动应用的测试框架与平台无关。 |
| `maintain-verification-skill` | 按功能并行读取源码的 reader 映射为 `spawn_agent` 扇出；项目本地技能位于 Codex 的技能位置，而不是 `.claude/skills/`。 |
| `create-formal-verification-skill` | 把生成的技能写到 Codex 的项目技能位置，并以它验证的领域命名。验证器及其运行命令与平台无关。 |
| `maintain-formal-verification-skill` | 项目技能（包括形式化验证技能）位于 Codex 的项目技能位置。 |
| `babysit` | `loop` 和 `AskUserQuestion` 通过上面的表格解析。 |
| `automate-me` | `plugin-dev:skill-development` 通过上面的技能表解析。 |
| `architect` | runner 小组经由 the **arena** skill 运行，因此它的 `spawn_agent` 扇出和模型替换同样适用。 |
| `arena` | 并行候选和交叉评审映射为 `spawn_agent`；runner 和交叉评审池替换为你配置的 Codex 模型（见上文 Model names）。 |
| `how` | 并行探索者和讲解者映射为 `spawn_agent` 扇出；替换为你配置的 Codex 模型。 |
| `reflect` | 三个评审者和综合者映射为 `spawn_agent`；替换为你配置的 Codex 模型。transcript 查找器读取的是 Claude Code 在 `~/.claude/projects/` 下的布局，因此改为传入第 1 步允许的会话摘要。 |
| `swarm` | 每个 worker 是一次使用你配置的 Codex 模型的 `spawn_agent` 调用，这些调用本身就并发运行；给每个写入的 worker 分配独立的 worktree 或输出目录（见上文 Subagent policy）。 |
| `why` | 并行调查者和综合者映射为 `spawn_agent`；替换为你配置的 Codex 模型。从 Codex 向会话暴露的工具中列出 MCP 服务器，而不是从 `.mcp.json` 或 `claude mcp list`。 |

## Vendored scripts

`skills/poteto-mode/scripts/` 附带 `watch-pr` PR 监视器、`orch` 存储 CLI 和 `worktree-audit.mjs`。`watch-pr/ship-pr` 命令负责待合并状态的检查与取消；`resume.mjs` 负责 [Resume storage](resume-storage.md) 中描述的共享检查点定位器。这些脚本使用 bun 和 Node.js，在 Codex 上运行方式相同；通过 `shell` 调用。它们需要 `bun` 和 `gh`，forge 为 Origin 时需要 `origin`。按其 shebang 用 `node` 运行 `worktree-audit.mjs`。审计会用 `stat` 跟随 worktree 各祖先目录中的每个链接，并按链接最终指向的目录匹配，bun 和 node 下行为一致。bun 无法解析自身路径的 worktree 会丢失最近一次对话，归入 `review`。悬空或循环的链接会被忽略。其他任何 `stat` 无法跟随的链接，会把该链接所在目录下的所有 worktree 归入 `review`，并给出指明该链接的警告。`worktree-audit.mjs` 扫描 `$CODEX_HOME/sessions` 和 `$CODEX_HOME/archived_sessions`（默认 `~/.codex`）下的 Codex 会话，以及存在的任何 Claude Code 或 Pi transcript 目录。它从 `skills/reflect/scripts/find-transcript.mjs` 导入 transcript 遍历器，因此请让 `reflect` 技能与 `poteto-mode` 一起安装。

## Instructions file

pstack 技能提到 "your instructions file" 时，在 Codex 上指 `AGENTS.md`（项目根目录，加上全局的 `~/.codex/AGENTS.md`）。在 Claude Code 上指 `CLAUDE.md`。
