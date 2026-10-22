# pstack 参考

安装与第一个任务请先看 [README](../README.md)。

<a id="slash-commands"></a>
## 斜杠命令

The package includes 65 skill directories: 41 public skills and 24 `principle-*` references. Claude Code 使用 `/pstack:<name>`，Pi 使用 `/skill:<name>`。在 Codex 中，按名字请求技能，或安装[可选快捷方式](#codex)以使用下表中的 `/name` 形式。

每个技能的说明都在[技能树](../plugins/pstack/skills/)中。

| command | use it when |
| --- | --- |
| `/poteto-mode` | 任何非平凡任务的默认入口 |
| `/how` | 走查一个子系统如何工作 |
| `/why` | 调查某处为什么这样构建（并行多 MCP 证据） |
| `/architect` | 在写跨函数边界的代码之前，先定下类型与模块形状 |
| `/arena` | 同一任务并行跑 N 次尝试，挑出最好的部分 |
| `/interrogate` | 让三个不同的模型尝试击破一个 diff |
| `/automate-me` | 从近期会话记录起草你自己的个人 -mode 技能 |
| `/reflect` | 把一次长任务的教训固化为技能修改 |
| `/correct` | 找出 agent 在仓库里反复犯的错误，并以可行的最高层级让每一个不可能再犯 |
| `/tdd` | 先写失败的测试，再修复缺陷 |
| `/benchmark-checklist` | 在报告或依据某个实测加速/回退之前先做校验（限制因素、调优、误差、重复运行、端到端相关性） |
| `/typescript-best-practices` | 用 TypeScript 语法落实类型系统纪律 |
| `/teach` | 组合 how + why，把一个子系统讲清楚 |
| `/swarm` | 跨切片或竞速并行铺开 N 个 worker，然后返回一份汇总报告 |
| `/technical-writing` | 按统一的分层标准写文档、RFC、readme、PR 描述和提交信息 |
| `/bro` | 用平实的人类语言复述上一条消息，不用行话 |
| `/in-your-own-words` | 在 agent 开始之前，把你的目标与问题复述回来 |
| `/sharpen` | 像顶尖专家那样，把任务瞄向你真正想要的东西，同时不过度规定 |
| `/figure-it-out` | 为没有现成 playbook 可用的任务，设计一份严谨、可审计的 playbook |
| `/show-me-your-work` | 把决策记录到一份可审查的 tsv 决策轨迹中 |
| `/blast-radius` | 找出改动在 diff 之外可能破坏什么，并通过运行代码来证明安全 |
| `/recall` | 从聊天历史、实时状态和共享记录中补上近期工作上下文 |
| `/ketchup` | 补上这个对话自你上一条消息以来的所有内容，并为每个待办事项给出执行摘要 |
| `/setup-pstack` | 配置 pstack 的逐角色模型选择 |
| `/poteto-help` | 回答关于 pstack 的问题，并给出该发送的 prompt 和答案所出自的文件 |
| `/unslop` | 清除 AI 腔调，把文字清理干净 |
| `/no-comments` | 审查前剥掉注释，修复被接受的发现，把声称的约束编码进去 |
| `/create-verification-skill` | 生成一个项目本地的验证技能与功能地图 |
| `/maintain-verification-skill` | 重新同步一个已漂移的验证技能及其功能地图 |
| `/create-formal-verification-skill` | 生成一个项目本地技能，对某一项高风险性质做形式验证 |
| `/maintain-formal-verification-skill` | 在所覆盖的代码变化之后，重新同步形式验证 |
| `/deslop` | 提交前对 diff 做 deslop |
| `/babysit` | 盯住一个进行中的 PR，修 CI/评论，保持 merge-ready |
| `/thermo-nuclear-review` | 审计一个分支的改动中的缺陷、破坏、安全问题、开发者体验回退和功能开关泄漏 |
| `/thermo-nuclear-code-quality-review` | 极其严格的可维护性审计 |
| `/thermos` | 把两个 thermo-nuclear 审查作为并行 subagent 运行，并综合它们的发现 |
| `/make-pr-easy-to-review` | 审查前清理噪声历史并改进 PR 描述 |
| `/fix-ci` | 找出失败的 PR 检查，查看日志，施加聚焦的修复 |
| `/fix-merge-conflicts` | 非交互地解决合并冲突，验证并收尾 |
| `/get-pr-comments` | 抓取并汇总当前 PR 的审查评论 |
| `/what-did-i-get-done` | 汇总用户选定时间段内你提交的 commit |

<a id="runtime-support"></a>
## 运行时支持

所有运行时共享[一棵技能树](../plugins/pstack/skills/)。纯技能安装包括技能、脚本、agent 引用和许可声明。Claude Code、Codex 和 GitHub Copilot 插件还会安装自动路由 hook，Pi 包则增加一个扩展，注入同样的路由并提供 subagent 工具。Codex 的命令快捷方式是独立的一步。

| 运行时 | 配置与已记录的验证 |
| --- | --- |
| Claude Code | 安装 marketplace 插件。技能使用 Claude 的工具名和模型默认值；插件安装自动路由。 |
| Codex | 通过仓库的 marketplace 安装原生插件，并通过 `/hooks` 信任它的 hook。[Codex 映射](../plugins/pstack/skills/poteto-mode/references/codex-tools.md)负责翻译 Claude 的工具与模型名。共享技能符号链接也在一次真实会话中被检测到。 |
| Pi | 用 `pi install` 把仓库安装为 Pi 包。[Pi 扩展](../plugins/pstack/pi/index.ts)注册 subagent、提问和唤醒工具以及 `/loop`，[Pi 映射](../plugins/pstack/skills/poteto-mode/references/pi-tools.md)翻译 Claude 的工具与模型名。[等价表](pi-equivalence.md)记录每个 Claude Code 机制以及它在 Pi 1.1 上的验证方式。 |
| GitHub Copilot | 通过仓库的 marketplace 安装插件；CLI 与 GitHub Copilot 应用共用它。[Copilot 映射](../plugins/pstack/skills/poteto-mode/references/copilot-tools.md)翻译 Claude 的工具、路径与模型角色，并把仅限应用的工具有对应地映射到 CLI 回退方案。安装、路由、agent 派发与首次运行的 setup 都在 CLI 上做过冒烟测试；见 [GitHub Copilot](#github-copilot)。 |
| Prime Agent | 其文档描述了共享目录发现；尚未在真实会话中测试。通过 Prime 的配置选择工具与模型。 |
| opencode | 在 2.0.30 版本上验证过：CLI 与桌面应用中的发现和技能加载，以及 CLI 中通过其 `subagent` 工具进行的委派。在 `opencode.json` 中配置 agent、命令与权限。它的选择器也会列出原则技能。 |
| Gemini CLI | 其文档描述了共享目录发现；尚未在真实会话中测试。用 `/skills list` 检查发现情况，改动后用 `/skills reload`。 |

这些检查覆盖技能发现。委派在 Prime Agent 和 Gemini CLI 上仍未验证，多模型工作流在三个运行时上也都未验证。在这些运行时上，agent 必须自行适配 Claude 特有的工具、模型和配置。每份映射只适用于它对应的运行时。

<a id="automatic-routing"></a>
### 自动路由

Claude Code、Codex 和 GitHub Copilot 插件在启动、恢复、清空和压缩时加载同一段简短的[路由指令](../plugins/pstack/hooks/session-start-context.md)。[POSIX hook](../plugins/pstack/hooks/session-start.sh)在 Claude Code、POSIX Codex 和 GitHub Copilot 上运行，把该指令作为 JSON `additionalContext` 打印，Copilot 还附带一段补充说明。在 Windows 上，Codex 通过其 `commandWindows` 覆盖项使用一个 [PowerShell 适配器](../plugins/pstack/hooks/session-start.ps1)，不需要 Bash。该适配器以 `-ExecutionPolicy Bypass` 运行，而由组策略设置的机器或用户执行策略会覆盖它；在这类机器上 hook 会失败，不加载任何指令。Codex 要求用户通过 `/hooks` 信任插件 hook。在 Pi 上，[扩展](../plugins/pstack/pi/prompt.ts)在每次 agent 启动时把同一条指令加入系统提示，因此它能挺过压缩。当任务满足以下任一条件时，该指令会调用 `poteto-mode`：

- 它触及不止一个文件，或改变了其他文件调用的签名。
- 它涉及设计或架构选择。
- 它涉及原因不明的缺陷，或性能问题。

更小的任务直接进行。完整技能在被调用时加载，用户明确的指令优先。

要关闭路由，运行 `setup-pstack` 并关掉 session hook。在 Claude Code 中，用 `/pstack:setup-pstack`。你也可以在该运行时的覆盖表中写入 `session hook: off`，路径见 [setup-pstack 的运行时表](../plugins/pstack/skills/setup-pstack/SKILL.md#other-runtimes)。hook 在注入指令之前会读取该设置。没有该设置时，路由保持开启。

纯技能安装和其他运行时不含 hook 或 Pi 扩展。请显式请求 `poteto-mode`，或往该运行时的指令文件中加入一条常驻指令。

<a id="shared-skills-installation"></a>
### 共享技能安装

需要 Prime Agent、opencode、Gemini CLI，或纯技能的 Codex 安装时，走这条路径。运行 README 中[其他 harness](../README.md#other-harnesses)一节的脚本，它会克隆仓库并把技能链接进 `~/.agents/skills/`。

保留所有技能目录，包括原则参考。链接安装期间，把克隆留在该路径。

<a id="manage-linked-skills"></a>
### 管理已链接的技能

如果目标已存在，替换前先检查它。安装会跳过已存在的文件、目录和链接。

要更新，先在做链接的那个克隆里运行 `git pull`，然后重跑安装循环。`pull` 会更新已链接的技能，但不会链接新增的技能目录。要卸载某个已链接的技能，删除它在 `~/.agents/skills/<name>` 的链接。这会把它从所有使用该目录的运行时中移除。

<a id="install-with-the-skills-cli"></a>
### 用 skills CLI 安装

不想保留本地克隆时：

```shell
npx skills add https://github.com/michael-denyer/pstack-claude/tree/main/plugins/pstack/skills --skill "*" --agent "*" --yes
```

[CI 安装检查](../.github/workflows/ci.yml)用 skills CLI 复制检出的技能树，并把安装后的文件与源文件比对。

<a id="codex"></a>
### Codex

[原生插件清单](../plugins/pstack/.codex-plugin/plugin.json)指向共享技能目录和 Codex 的 [SessionStart hook](../plugins/pstack/hooks/codex-hooks.json)。[marketplace 目录](../.agents/plugins/marketplace.json)在 `pstack-claude` marketplace 中列出 `pstack`。通过 `/hooks` 审阅并信任该 hook；当它的定义变化时 Codex 会再次询问。

[README 安装](../README.md#codex)一节用 `codex plugin marketplace add` 注册该目录，再用 `codex plugin add` 安装插件。这些命令与 `codex-cli 0.154.0-alpha.6.2` 的帮助输出一致。本次文档改动没有测试全新的原生安装。

OpenAI 记录了[marketplace 注册与插件格式](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli)。如果你的 CLI 没有 `plugin add`，注册 marketplace 后使用插件浏览器，或改用[纯技能安装](#shared-skills-installation)。

按名字请求 `poteto-mode`，或选择它的条目，例如 `pstack:poteto-mode`。要启用并行 subagent：

```toml
[features]
multi_agent = true
```

如果 subagent 被禁用，把该设置加入 `~/.codex/config.toml`。`arena`、`interrogate` 和 `architect` 这类技能会使用并行 agent。[映射](../plugins/pstack/skills/poteto-mode/references/codex-tools.md)描述了顺序回退方案，并翻译 Claude 的工具名、模型默认值与验证指令。

需要可选的斜杠命令快捷方式时，在克隆根目录运行：

```shell
mkdir -p ~/.codex/prompts
for c in plugins/pstack/.codex-plugin/prompts/*.md; do
  target=~/.codex/prompts/"$(basename "$c")"
  test -e "$target" || test -L "$target" || ln -s "$(pwd)/$c" "$target"
done
```

每个快捷方式调用它对应的技能。这些命令会跳过已存在的文件和链接。要移除快捷方式，删除它在 `~/.codex/prompts/<name>.md` 的链接。原生插件与纯技能安装在没有这些快捷方式时都能工作。

<a id="pi"></a>
### Pi

仓库根是一个 [Pi 包](https://pi.dev/packages)：它的 [`package.json`](../package.json)列出共享技能目录和 [pstack Pi 扩展](../plugins/pstack/pi/index.ts)。用 `pi install git:github.com/michael-denyer/pstack-claude` 安装，或对本地检出用 `pi install <clone path>`。

扩展在 Claude Code 的名称下提供 Pi 原生缺失的东西，技能正是用这些名称书写：

- `agent` 以技能给出的 `subagent_type`、模型和推理档位派发一个子 `pi --mode rpc` 进程，可前台或后台运行，并可选择使用它自己的 git worktree。后台 agent 完成时，会在其当前工具调用之后并入正在运行的回合；若会话空闲则开启一个回合。
- `send_message`、`list_agents` 和 `stop_agent` 负责向这些 agent 发消息、列出和停止。给运行中 agent 的消息是子进程 stdin 上的 RPC `steer`，agent 在其当前工具调用之后读取，然后在同一次运行中继续。已完成的 agent 会带着该消息恢复。agent 的状态跟随它的进程。`list_agents`、前台 agent 的结果以及每条完成通知都会点名子进程的会话记录 `<agent dir>/pstack/<parent session id>/agents/` 下的 `<timestamp>_<session id>.jsonl`，因此用户可以用 `tail -f` 跟踪运行中的 agent，或用 `pi --export` 渲染已完成的 agent。运行中 agent 的 `list_agents` 条目还带有 `activity`：它当前或最后一次工具调用及最新消息，从子进程的 RPC 事件读取并保存在内存中，所以模型只能在 `list_agents` 结果中看到它。顶层会话在 `pstack-agents` 页脚行显示数量与最近活跃的 agent，用户可以运行 `/pstack-agents` 查看逐 agent 视图，该视图不发送给模型。
- `ask_user_question` 和 `schedule_wakeup` 对应 `AskUserQuestion` 和 `ScheduleWakeup`，`/loop` 对应 `loop` 技能。在子 agent 中，提问与安排唤醒都会返回错误。
- 每次 agent 启动时，它把路由指令、覆盖表 `~/.pi/agent/pstack-models.md`，以及指向 Pi 映射的一行加入系统提示。

`opus` 这类族名通过 [`models.json`](../plugins/pstack/models.json)的 `pi` 块，解析为 Pi 会话所用提供方的模型 ID。ChatGPT 登录（`openai`，或旧的 `openai-codex`）得到 OpenAI 模型，Anthropic 或任何其他提供方得到 Claude 模型。Pi 会警告：即使你有 Claude 订阅，Anthropic 也会把通过 Pi 使用的 Claude 按 token 计为额外用量，而 pstack 启动的每个 subagent 都会增加这笔账单。要在 Claude 计划的额度内使用，请在 Claude Code 中运行 pstack。覆盖表中的 `pi models:` 行可以重映射任何族名。[Pi 映射](../plugins/pstack/skills/poteto-mode/references/pi-tools.md)列出每一处翻译，[等价表](pi-equivalence.md)记录了什么被验证以及如何验证。

<a id="github-copilot"></a>
### GitHub Copilot

Copilot CLI 读取 Claude Code 的 [marketplace](../.claude-plugin/marketplace.json)。对插件，它会先查找 `.github/plugin/plugin.json`，再查找 `.claude-plugin/plugin.json`，所以 pstack 提供了一份 [Copilot 清单](../plugins/pstack/.github/plugin/plugin.json)，其 `hooks` 字段点名 [`hooks/copilot-hooks.json`](../plugins/pstack/hooks/copilot-hooks.json)。这样 Copilot 会跳过 Claude Code 的 `hooks/hooks.json`，每个 hook 只跑一次，而 Claude Code 继续读它自己的清单。GitHub Copilot 应用加载安装进 `~/.copilot` 的插件；2026-09-25 对路由、首次运行 setup、agent 派发、`session hook: off` 和仅限应用的工具做过一次手工应用侧检查，全部通过。要试本地检出，把它的绝对路径传给 `copilot plugin marketplace add`。技能通过 `skill` 工具按裸名加载，用户在 CLI 提示符中输入 `/pstack:<skill>`；agent 保留前缀，即 `pstack:poteto-agent` 和 `pstack:comment-sicko`。

Copilot 的 hook 文件以 `copilot` 参数运行 [SessionStart hook](../plugins/pstack/hooks/session-start.sh)。它打印 JSON `additionalContext`：路由指令加上一段 [Copilot 补充说明](../plugins/pstack/hooks/session-start-copilot.md)。它读取位于 Copilot 路径沙箱之外的模型覆盖表，并用与 `setup-pstack` 写完表后经 `check-sheet.sh` 运行的[同一个 POSIX `awk` 校验器](../plugins/pstack/skills/setup-pstack/scripts/sheet.awk)检查它。合法的表会按已知角色各加一行，内容由已检查的值重建，因此表中其他文字不会进入上下文。非法的表会加上 `sheet invalid:` 与每个问题，缺失的表会加一条注记；两者都告诉会话先运行 `setup-pstack`。第二个 hook，[`pre-tool-use.sh`](../plugins/pstack/hooks/pre-tool-use.sh) 配 `view|task|bash` 匹配器，在 Copilot 上做三件事。它批准插件目录内的读取，让 playbook 和参考文件无需路径访问提示即可加载。它批准带有声明参数契约且解释器匹配的已注册脚本，使用不含 shell 元字符的严格命令形式。它把路径操作数与散文字段分开检查；未知脚本和参数形式走正常的权限流程。setup、SessionStart 和 PreToolUse 共用一个表解码器，包括 UTF-16LE 支持。表合法期间，它会拒绝 `model` 不在已保存选项之内的 `pstack:*` `task` 调用，并在原因中列出已保存的 ID。其他情况保持沉默并总是以 0 退出，因为 hook 失败时 Copilot 会拒绝工具调用；`awk` 错误仍会到达 stderr。Copilot 在每次会话启动时都运行插件 hook，所以 Claude 的匹配器不适用。该 hook 惰性运行：在第一条消息提交之后、第一个模型回合之前，包括该消息是 `/pstack:arena` 这类斜杠命令时。恢复的会话会以 `resume` 来源再运行一次，模型仍然只看到一个路由块。在 CLI 1.0.87 到 1.0.92 上，多个插件的 session-start hook 上下文会合并（[github/copilot-cli#3589](https://github.com/github/copilot-cli/issues/3589)曾报告只有最后一个存活）。如果某个版本不再合并，请使用 `setup-pstack` 提供的常驻指令。在 1.0.92 上，当 CLI 缓存的实验分组开启 computer use 时，`-p` 会话不加载任何插件技能，而交互式会话仍然加载；冒烟测试在每次 `-p` 探测前清除该缓存。

pstack 不自带 Copilot 的默认模型，因为账号能触达的模型取决于它的计划与策略。第一个按角色模型派发的技能会运行 `setup-pstack`。它按档位和 panel 槽位各问一个 `ask_user` 问题，每个都是来自 `task` 工具模型列表的简短选项，并按厂商分组，不推荐任何模型。然后它写入 `${COPILOT_HOME:-~/.copilot}/pstack-models.md` 并运行 `check-sheet.sh`，之后的会话通过 hook 收到这些选择。panel 模型请从不同厂商中选择。角色值上的 `@<level>` 后缀与表中的 `default effort` 行映射到 `task` 工具的 `reasoning_effort`。

Copilot 在其提示中只列出大型插件的部分技能，所以有些 pstack 技能不会出现在那里；每个技能仍可按名字加载。如果另一个 hook 或纯技能安装挤掉了路由指令，`setup-pstack` 会提供一个写进 `~/.copilot/copilot-instructions.md` 的常驻指令。

[`tests/copilot-smoke.sh`](../tests/copilot-smoke.sh)把检出安装进一次性的 `COPILOT_HOME`，并从每个会话的 `events.jsonl` 与写出的表中检查：安装、路由、`session hook: off`、带显式模型的 agent 派发、首次运行 setup、setup 在没有用户回答时不写表而有了回答时只写所提供的 ID，以及已保存的选择和插件文件在无 `--allow-all-paths`、无任何路径访问请求的情况下抵达会话。它对 `SMOKE_SETUP_MODELS` 中的每个模型各跑一次 setup 检查。它还检查 `PreToolUse` 的拒绝、一个 vendored 脚本与 `check-sheet.sh` 在无权限请求下运行、注入的表行不会进入上下文、非法表产出 `sheet invalid`、每个 hook 只触发一次、与第二个插件 session-start 上下文的合并、`-p` 恢复，以及通过 [`tests/copilot-tui.py`](../tests/copilot-tui.py)在伪终端上的一条斜杠命令首条消息与一次交互式恢复。它需要已登录的 `copilot` CLI，消耗约二十次 premium 请求，缺少 `copilot` 时跳过。CI 不运行它。

<a id="configuration-and-dependencies"></a>
## 配置与依赖

调用 [setup-pstack](../plugins/pstack/skills/setup-pstack/SKILL.md)为每个角色选择模型。它会检测可用模型、确认选择并写出覆盖表。它的[运行时表](../plugins/pstack/skills/setup-pstack/SKILL.md#other-runtimes)为每个运行时给出覆盖表路径与加载机制。默认值位于 [models.json](../plugins/pstack/models.json)。

做设计比较与审查时，请选择你的运行时可用且彼此不同的模型。默认 panel 使用不同的 Claude 模型。

按你使用的工作流安装依赖：

| 依赖 | 何时需要 |
| --- | --- |
| GitHub CLI，`gh` | PR 监控、交付，以及 `orch` 的 stack frontier。用 `gh auth login` 认证。当 Origin 的 CLI `origin` 能解析仓库时，playbook 改用它。 |
| Bun | 自带的 `watch-pr` 与 `orch` 脚本。它们的 bootstrap 在首次运行时安装脚本依赖。 |
| `plugin-dev` | `automate-me`、`reflect` 和 `poteto-mode` 使用的 Claude Code 技能编写指导。 |

用以下命令安装 Claude Code 技能编写配套插件：

```text
/plugin marketplace add anthropics/claude-plugins-official
/plugin install plugin-dev@claude-plugins-official
```

这些编写类工作流需要 `plugin-dev` 提供指导；其他工作流不需要。Codex 和 GitHub Copilot 使用各自映射中点名的等价物：[Codex](../plugins/pstack/skills/poteto-mode/references/codex-tools.md#driver-and-bundled-skills-pstack-references) 和 [Copilot](../plugins/pstack/skills/poteto-mode/references/copilot-tools.md#driver-and-bundled-skills-pstack-references)。

Playbook 使用运行时的任务跟踪工具，或一份未提交的 `todo.md` 清单。对 Claude Code，仓库记录了 `CLAUDE_CODE_ENABLE_TODO_TOOLS=1`；见[平台适配](../plugins/pstack/skills/poteto-mode/SKILL.md#platform-adaptation)。

用 [create-verification-skill](../plugins/pstack/skills/create-verification-skill/SKILL.md)记录 agent 应如何运行和检查你的项目，遵循 [driver 策略](../plugins/pstack/skills/poteto-mode/SKILL.md#non-negotiables)。

<a id="maintenance"></a>
## 维护

<a id="repository-layout"></a>
### 仓库布局

```text
.claude-plugin/marketplace.json    Claude Code marketplace
.agents/plugins/marketplace.json  Codex marketplace
package.json                      Pi package manifest
plugins/pstack/
  .claude-plugin/plugin.json      Claude Code plugin manifest
  .codex-plugin/                  Codex manifest and generated prompt stubs
  .github/plugin/plugin.json      GitHub Copilot plugin manifest
  pi/                            Pi extension (subagent, question, and wake-up tools)
  skills/                        Shared skills, references, and scripts
  agents/                        Claude Code subagent definitions
  hooks/                         Startup routing for Claude Code, Codex, and Copilot; Copilot tool checks
tools/                           Generation, validation, and upstream sync
tests/                           Repository checks
```

纯技能安装使用 `plugins/pstack/skills/`。agent 引用与许可文件包含在 `poteto-mode/references/` 下。

<a id="generated-files-and-checks"></a>
### 生成文件与检查

[生成器](../tools/generate.mjs)更新版本、模型默认值、Codex prompt、Copilot preamble 与角色列表，以及可移植的参考文件，并校验 Pi 包清单。[斜杠命令表](#slash-commands)提供 Codex prompt 的描述与顺序。改菜单描述时编辑该表，然后重新生成。为每个公开技能保留一行，`poteto-mode` 放在最前。

[文档事实测试](../tests/readme-facts.test.mjs)检查技能计数与上游 pin。表解析器要求表头为 `| command | use it when |`。

用 Bun 运行生成器与仓库测试：

```shell
bun install --frozen-lockfile
bun tools/generate.mjs
bun test tests/
```

安装步骤读取根 `bun.lock` 并拉取 `typebox`，Pi 扩展及其测试会导入它。

CI 还会检查 shell 脚本、工作流、Markdown、相对链接，以及自带的 Bun 工具。命令见[本地检查](../CONTRIBUTING.md#things-that-will-fail-ci)，版本管理与真实 Claude Code 命令检查见[发布说明](../CONTRIBUTING.md#releasing)。

<a id="port-scope-and-attribution"></a>
### 移植范围与署名

技能树 synced against upstream `7b353f7`（v0.15.27 及之后）。thermos 技能与 agent synced against `118e73b`（thermos 1.0.0）。

本仓库把 Lauren Tan 的 pstack 从 Cursor 移植到 Claude Code，并与其他运行时共享技能。它包含六个 cursor-team-kit 技能、三个 thermos 技能及其两个审查 subagent，以及一个独立编写的 `babysit` 技能。移植提供 Claude Code 插件注册与路由、Codex 清单与快捷方式、Codex 工具映射、Pi 包、扩展与工具映射，以及 GitHub Copilot 的 hook 与工具映射。

Cursor 特有的 automation、sticky-mode 元数据、Grok Bot UI 工作流和 Cursor UI 教程被排除。[tools/upstream.json](../tools/upstream.json)记录修订与排除项，[tools/substitutions.json](../tools/substitutions.json)存放 Cursor 到 Claude 的重写规则，[CHANGES.md](../CHANGES.md)记录每个版本。自带的 `thermo-nuclear-code-quality-review` 在工作流需要时提供可维护性审查，`thermo-nuclear-review` 提供正确性与安全审计，`thermos` 把两者作为并行 subagent 运行。

技能改动请遵循[同步边界](../CONTRIBUTING.md#the-sync-boundary)。运行时适配与工作流改动都落在这里，工作流改动要声明为分叉。

许可与完整插件署名见[许可摘要](../README.md#license)。[NOTICE-skills.md](../NOTICE-skills.md)是纯技能安装的声明。
