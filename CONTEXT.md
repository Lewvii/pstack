# CONTEXT：领域术语表

这些是本仓库的设计讨论反复援引的概念的名称。架构评审使用这些术语；如果某次评审需要一个这里没有的概念，请把它加进来。

- **Build**：插件发布的运行时之一：Claude Code 构建（`.claude-plugin/`）、Codex 构建（`.codex-plugin/`）、Pi 构建（仓库根目录的 `package.json` 以及 `plugins/pstack/pi/` 中的扩展）、GitHub Copilot 构建（由 Copilot CLI 和应用读取的 Claude Code 清单，带有自己的 hook 上下文、`PreToolUse` 检查与映射），以及 Prime Agent、opencode 和 Gemini CLI 的原生 Agent Skills 路径（`~/.agents/skills/` 符号链接）。所有构建共享同一棵 `skills/` 树。技能保留上游的 Claude 原生措辞，Codex、Pi 和 Copilot 构建则通过 `skills/poteto-mode/references/codex-tools.md`、`pi-tools.md` 和 `copilot-tools.md` 解析工具名，因为用工具中立的语言重写它们会让每个上游技能分叉。运行时专属的执行适配器，例如 Pi 扩展，与发现机制保持分离。
- **Agent Skill**：`plugins/pstack/skills/` 下的一个目录，带有可移植的 `name` 与 `description` frontmatter。Codex、Prime Agent、opencode 和 Gemini CLI 会原生发现这棵共享树。运行时专属的键只在该运行时理解它们的地方影响可见性。
- **Model policy**：角色到模型 slug 的映射。默认值位于 `plugins/pstack/models.json`，并被写入每个技能的 Models 小节；用户的覆盖表（`pstack-models.md`，由 `setup-pstack` 写到各运行时的路径下）是在运行时适配它们的那一层。Codex 和 GitHub Copilot 保留各自的表。Copilot 不随附默认 slug，因此在技能按角色模型分派之前，必须先有它的表。
- **Role**：一个具名的委派工作单元，带有自己的模型选择（`arena runners`、`how critics`、`swarm workers`）。角色词汇位于 `models.json`，并在 `setup-pstack` 的表中呈现。
- **Panel**：多模型技能（`arena`、`architect`、`interrogate`、`how`）默认运行的三模型多样化面板。它在 `models.json` 中只是一条事实；生成器把它写入每一份副本，遇到残留副本就失败。
- **Prompt stub**：可选的 Codex 斜杠命令，用于调用某个技能。生成器依据该技能在[斜杠命令参考](docs/reference.md#slash-commands)中的那一行，把它写到 `.codex-plugin/prompts/` 下。没有它，原生技能发现依然可用。
- **Menu description**：公开技能在[斜杠命令参考](docs/reference.md#slash-commands)中对应那一行的简短描述。Codex 会在斜杠菜单里展示它。技能 frontmatter 里的 `description` 则告诉 agent 何时调用它。
- **Generator**：`tools/generate.mjs`。它把事实从其唯一来源写入每一份已提交的副本，并校验跨文件契约。CI 以 `--check` 运行它，任何过期的副本都会导致失败。
- **Generator-owned copy**：由生成器写入的已提交取值，例如清单版本、prompt stub 或 Models 小节。重新生成会覆盖手工修改，任何差异都会被 CI 抓到。
- **VERSION**：位于仓库根目录、保存插件规范版本的文件。发版时编辑它，加上对应的 `CHANGES.md` 标题，然后重新生成；插件自动更新按这个数字安装。
- **Sync boundary**：上游 pstack 所拥有的内容（技能内容）与此移植版所拥有的内容（Cursor 到 Claude Code 的转换，以及 `tools/forks.json` 中声明具名的分叉）之间的分界。定义见 [CONTRIBUTING.md](CONTRIBUTING.md)；由 `tools/sync.mjs` 强制执行，其替换表和拒绝清单位于 `tools/substitutions.json`，任何 `tools/forks.json` 未声明的分叉都会让它失败。
- **Upstream pin**：`tools/upstream.json` 中每个组件对应的上游 SHA，移植版同步到该版本。只有当一次同步完成且没有命中拒绝清单、也没有三方二进制冲突时，`sync.mjs` 才会推进它。文本冲突不会拖住它：冲突会连同 git 的冲突标记一起写入工作树，而生成器会拒绝这种状态。
- **Invariants**：在 `tools/generate.mjs` 内部强制执行的布局与 flag 规则（`agentSkills`、`validatePluginLayout`）；`tests/invariants.test.mjs` 证明每一条都能在夹具树上失败。`tests/skill-collision-repro.sh` 是唯一的行为检查，需要 `claude` CLI。
