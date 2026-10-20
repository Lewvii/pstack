# pstack

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-support-yellow?logo=buymeacoffee&logoColor=white)](https://buymeacoffee.com/codenyer)

Lauren Tan 的 [pstack](https://github.com/cursor/plugins/tree/main/pstack) 是一套有明确主张的 Cursor 技能栈，用来提升 agent 的产出质量。本仓库是它在 Claude Code、Codex、Pi、GitHub Copilot 以及其他 agent harness 上的移植。它跟踪上游，同时携带具名的策略分叉，每一处分叉都在 [`tools/forks.json`](tools/forks.json) 中声明。

把目标告诉 `poteto-mode`，它会为这项任务调用正确的工作流。它让代码保持简洁、简单、经过验证。

对于测试触及不到的并发缺陷和不变量，`/create-formal-verification-skill` 会挑出代码中风险最高的性质，用合适的工具验证，并留下一个可供下一个 agent 重跑的项目技能。独立的 [agent-formal-verify](https://github.com/michael-denyer/agent-formal-verify) 插件在方法上走得更深，提供引导式 TLA+ 模型检查、Lean 证明，以及面向 Rust 的 Kani。

## 安装

<details>
<summary><strong>Claude Code</strong></summary>

在 Claude Code 中运行：

```text
/plugin marketplace add michael-denyer/pstack-claude
/plugin install pstack@pstack-claude
```

</details>

<a id="codex"></a>
<details>
<summary><strong>Codex</strong></summary>

在终端中运行：

```shell
codex plugin marketplace add michael-denyer/pstack-claude
codex plugin add pstack@pstack-claude
```

</details>

<details>
<summary><strong>Pi</strong></summary>

在终端中运行：

```shell
pi install git:github.com/michael-denyer/pstack-claude
```

这个包会加载技能和 pstack Pi 扩展。该扩展补上了技能所用的 subagent、提问和唤醒工具，外加 `/loop` 和路由指令。用 `/skill:<name>` 调用技能。

</details>

<details>
<summary><strong>GitHub Copilot</strong></summary>

在终端中运行：

```shell
copilot plugin marketplace add michael-denyer/pstack-claude
copilot plugin install pstack@pstack-claude
```

这会把 pstack 安装到 Copilot CLI 和 GitHub Copilot 应用，两者共用 `~/.copilot`。安装后请新开一个会话。Copilot 不自带 pstack 的默认模型，所以第一个需要模型的技能会运行 `setup-pstack`，从你账号列出的模型中选择，之后的会话复用这个选择。

Copilot 构建在 Copilot CLI 1.0.87 到 1.0.92 上测过。在这些版本上，路由 hook 的上下文与其他插件的 SessionStart 上下文一起抵达会话。如果之后的版本只保留一个插件的上下文，`setup-pstack` 会提供一个写进 `~/.copilot/copilot-instructions.md` 的[常驻指令](plugins/pstack/skills/setup-pstack/copilot.md#wire-it-in)。在 1.0.92 上，一旦 CLI 缓存了它的 computer-use 实验分组，`copilot -p` 会话就不会列出任何插件技能，`skill` 调用会返回 "Skill not found"。交互式会话、hook 和 agent 都不受影响。

</details>

<a id="other-harnesses"></a>
<details>
<summary><strong>其他 harness</strong></summary>

OpenCode（CLI 与桌面应用）、Gemini CLI、Prime Agent，以及纯技能的 Codex 安装都从 `~/.agents/skills/` 读取技能。在终端中运行：

```shell
git clone https://github.com/michael-denyer/pstack-claude
cd pstack-claude
mkdir -p ~/.agents/skills
for s in plugins/pstack/skills/*/; do
  target=~/.agents/skills/"$(basename "$s")"
  test -e "$target" || test -L "$target" || ln -s "$(pwd)/$s" "$target"
done
```

要更新，先在克隆目录里运行 `git pull`，再重跑这个 `for` 循环，把上次运行之后新增的技能链接进来。这类安装没有路由 hook，所以要按名字请求 `poteto-mode`，或者往该 harness 的指令文件里加一条常驻指令。各 harness 的说明与卸载方式见[共享安装](docs/reference.md#共享技能安装)。

</details>

### 配置

pstack 会把任务自动路由到 `poteto-mode`。会话启动时，一个 hook 会注入指向它的指令：在 Claude Code、Codex 和 GitHub Copilot 上是插件的 hook，在 Pi 上是 pstack 扩展。Codex 只在你通过 `/hooks` 信任该 hook 之后才运行它。其他 harness 没有 hook。

运行 `setup-pstack` 可以开关自动路由、修改默认模型，或为每个角色设置推理档位（例如 `arena runners: opus @xhigh, fable @max`）。

## 快速开始

```text
Use poteto-mode to fix the search filter resetting when I change pages.
```

以缺陷为例，它会复现失败，用 `how` 和 `why` 做调查，把修复委派出去，然后重跑失败的用例。如果修复跨过函数边界，它会在动手实现之前引入 `architect`。你拿到的是修复本身，以及失败与通过的证据。

[其他 playbook](plugins/pstack/skills/poteto-mode/SKILL.md#playbooks) 覆盖规划、功能、重构、性能问题、调查、原型、PR 维护、交付，以及更长期的项目。

![poteto-mode 在 Claude Code、Codex 和 Pi 上把一次请求变成经过验证的工作。选择 playbook，用 architect、arena 或 swarm 规划并委派，再用 interrogate、测试和度量来审查与验证。项目 playbook 可以定制工作流，setup-pstack 按角色配置模型与推理档位。配套技能包括 how、why 和 unslop。](assets/pstack-overview.png)

## 详情

- [技能与斜杠命令](docs/reference.md#slash-commands)
- [运行时配置](docs/reference.md#runtime-support)
- [模型与依赖](docs/reference.md#configuration-and-dependencies)
- [维护与移植范围](docs/reference.md#maintenance)

## 数据处理

pstack 没有服务端，也没有遥测。技能要求你的 agent 读取的任何内容，包括会话记录，都会发往你的模型提供方。脚本在本地运行，PR 工具使用你的 GitHub CLI 登录。

## 参与贡献

感谢你帮助改进这个移植。缺陷报告、文档修正和运行时改进都欢迎。检查项与改动该放在哪里，见 [CONTRIBUTING.md](CONTRIBUTING.md)。漏洞请按 [SECURITY.md](SECURITY.md) 所述私下报告。

如果想支持这个移植的维护，可以[请维护者喝杯咖啡](https://buymeacoffee.com/codenyer)。

<a id="license"></a>
## 许可证

本移植，包括它的修改与新增，同样以 [MIT 许可](LICENSE)发布，© 2026 Michael Denyer。原始 pstack © 2026 Lauren Tan；引入的 cursor-team-kit 与 thermos 技能 © 2026 Cursor。见 [LICENSE-cursor-team-kit](LICENSE-cursor-team-kit)、[LICENSE-thermos](LICENSE-thermos) 和 [NOTICE.md](NOTICE.md)。
