# 参与贡献

感谢你的帮助。本仓库是一个**移植**，不是原创作品：`skills/` 树跟踪[上游 pstack](https://github.com/cursor/plugins/tree/main/pstack)，并定期向前同步。这一点决定了后文的绝大部分内容。

[CONTEXT.md](CONTEXT.md) 是下列术语的词汇表。

<a id="the-sync-boundary"></a>
## 同步边界

上游拥有技能内容。本移植拥有 Cursor 到 Claude Code 的翻译。它还携带具名分叉，每一处都在 [`tools/forks.json`](tools/forks.json) 中声明为 `port-feature`（上游无法承载的 Claude Code、Codex、Pi 或 GitHub Copilot 机制）或 `policy`（移植选择保留的工作流改动）。

两类 `SKILL.md` 改动都落在本仓库。上游很少合并来自其团队之外的 PR，所以本移植不要求你先在那里落地改动。在改动 `SKILL.md` 之前，先判断你的改动属于哪一类：

- **修复移植。** 某个 Cursor 原语在 Claude Code 上解析错误、交叉引用断裂、模型 slug 过期。开 PR。
- **改变技能的行为。** 新增步骤、不同的工作流、重新措辞的指导。同样在这里开 PR。移植会把改动保留为 `policy` 分叉，之后每次同步都会拿它与上游新文本做三方合并。把改动控制得小且集中，因为大范围重写会在上游每次改动相同行时冲突，届时有人得在时间压力下重新裁定。

在同一个 PR 中，把你改动所分叉的每个文件登记进 `tools/forks.json`，这样下次同步就知道这是刻意的，而非漂移。把条目的 `upstream` 字段设为 `not-proposed`，或者你在上游开了 PR/issue 的话，填它的 URL。

[`tools/substitutions.json`](tools/substitutions.json) 中的每条替换规则都有一个 `rationale` 字段说明理由。如果你新增规则，请在同一个 PR 中填写它的 `rationale`。

<a id="running-a-sync"></a>
### 运行一次同步

```shell
bun tools/sync.mjs pstack <new-upstream-sha>
```

`tools/upstream.json` 按组件固定当前上游 SHA；`tools/substitutions.json` 存放机械化的 Cursor 到 Claude 重写规则，以及一份 denylist——其中的 Cursor 腔调需要重写整句，而不是替换 token。一条重写规则匹配字面量 `pattern` 或 `regex`，可选地只作用于上游相对路径匹配 `files` 的文件，并携带字面量 `replacement` 与 `rationale`；denylist 条目则是字面量 `token` 或 `regex`。规则按顺序执行，每条作用于前几条的输出，所以一个 pattern 要放在它包含的任何更短 pattern 之上。工具会在规则顺序颠倒、出现未知字段、或规则不恰好有一个匹配器时失败。匹配一种形态（例如反引号包裹的模型 slug），而不是它的某个具体版本，这样上游更新模型时无需新增规则。工具会拉取两个上游修订，把每个上游文件派生为移植形态：先替换，再走 `tools/generate.mjs` 中的 `deriveSkill`。它把技能或插件 agent 的 `name` 设为它的目录名或文件名，丢弃 Cursor 独有的 `mode`、`icon`、`color`、`reminder` 和 `is_background` 键，丢弃 `disable-model-invocation`（在 `principle-*` 叶子上换成 `user-invocable: false`）。随后它应用生成器的盖章，在上游没有 `## Models` 段时追加该段。它写入那些本地差异完全来自上游的文件（含新文件），删除上游已删除、而移植从未在文本或模式上改动过的文件，对剩下的运行 `git merge-file`：移植分叉过而上游未动的文件（文本或仅模式）计为分叉并保持原样；不重叠的编辑合并后写入；真正的冲突以 git 的 `<<<<<<< local`、`=======`、`>>>>>>> upstream` 标记写入，并按 hunk 数列出。写入的文件采用上游的文件模式，但合并或冲突的文件在上游未改动模式时保留移植的模式。这里的模式指 git 记录的可执行位与否。写入不会授予文件自身权限位或你的 umask 所不允许的权限。新文件按 umask 创建。既有文件只在模式必须改变时才动，且只动可执行位：文件可读时置位，否则清除。仅上传改可执行位的上游改动也会写入。符号链接（上游的或在移植侧位于上游路径或其上级目录的）、以及上游删除而移植改动过的文件，都以各自的原因进入同一个冲突列表。移植改动过的被删文件会保留，并在固定摘要中作为 now port-only 打印，以便 CHANGES.md 条目记录该决定。工具从不跟随或透过任一侧的符号链接写入。上游链接的目标可能位于克隆之外，而透过移植侧链接写入会改动或创建它的目标。三方都不相同的文件在上游或移植副本为二进制时无法携带标记，因此运行会以该路径失败：用上游版本替换它，或把它加入 `exclude`，然后重跑。移植侧路径挡住上游要写的文件时，运行也会在任何写入之前以该路径失败：上游是目录而这里是文件、上游是文件而这里是目录，或以另一种拼写指向同一文件系统名称的条目（默认 macOS 文件系统在大小写和 Unicode 规范化上就会如此）。改名或删除它，然后重跑。写入文件中的任何 denylist token（含冲突标记）都会让运行以文件、行号和提示失败——新增替换规则或重写该句，然后重跑。失败的运行不写入任何内容。固定 SHA 只在成功时前进，而带冲突的运行也算成功：生成器会在 `plugins/pstack` 下任何冲突标记行上失败，所以提交前要解掉每个 hunk。根据打印的报告写 CHANGES.md 条目，然后照常运行生成器和测试。

`--dry-run` 只报告不写入。工具接受组件、SHA 和 `--dry-run`，任何其他参数都以退出码 2 报出它自身，所以拼错的标志绝不会跑一次真实同步。在已固定的 SHA 上使用 `--dry-run` 会打印所有权地图。分叉计数下的每个文件都是移植分叉过的，不写入也不冲突，其余文件干净同步。每个分叉文件都会打印它相对于派生上游文本的改动行数，从大到小排列，下方给出总数；仅在文件模式上不同的分叉打印 `mode`。`port-only` 列表列出组件本地路径下、在其固定版本上没有任何组件在上游携带的文件（排除路径除外）。

`tools/forks.json` 在各自组件下声明每一处分叉，以仓库相对路径为键。每个条目有一个 `kind`（`port-feature` 或 `policy`）、一句点名机制的 `why`、分叉首次随哪个版本发布的 `since`，以及 `upstream` 状态：`not-proposed`，或上游 PR/issue 的 URL。所有权地图会在分叉计数旁打印它的 kind。分叉、合并或冲突的路径若没有条目，运行会失败（含 dry run），真实运行随后不写入任何内容。在使该文件分叉的那个 PR 中加入条目。路径并未分叉或已不存在的条目，会在已固定 SHA 的运行中失败，这就是 CI 检查，运行不写入任何内容。上游文件被移植删除却未加入 `exclude` 时同样如此，因为同步到新 SHA 会把它写回来。同步到新 SHA 时，同样的条目只发出警告，因为上游在该次运行中吸收了分叉；在该次同步的 PR 中删除条目。

当一次干净合并的结果与新上游文本和模式都一致时，同步将其报告为 `updated`。同一轮运行中，任何既有的分叉声明都会变得过期，删除该声明不会阻止更新。仅保留移植侧的模式改动，即使文本与上游一致，仍计为分叉。

<a id="before-you-open-a-pr"></a>
## 打开 PR 之前

先安装根依赖，然后运行生成器和测试：

```shell
bun install --frozen-lockfile
bun tools/generate.mjs
bun test tests/
```

根 `package.json` 把 `typebox` 和可选的 `@earendil-works/pi-coding-agent` 声明为 peer 依赖，因为 Pi 在运行时把两者提供给扩展。根 `bun.lock` 为生成器固定 `typebox`（生成器据此检查 hook 文件），也为 `tests/pi/` 下的测试固定它（这些测试在 Pi 之外加载扩展）。

生成器把 `VERSION` 写进全部四个插件清单和 Pi 的 `package.json`，并从 `plugins/pstack/models.json` 盖上模型默认值。它还拥有部分文件首个标题下的行：在 `skills/poteto-mode/references/codex-tools.md` 的 Per-skill notes 表中有行的每个技能，会得到 Codex preamble；`DRIVER_PLAYBOOKS` 中的 playbook 会得到 driver-skill 那一行。这两类行出现在其他任何地方都会让生成器失败，所以技能是通过获得 notes 行来拿到 preamble 的。GitHub Copilot 的 preamble 同样来自 `copilot-tools.md` 的 Per-skill notes 表，并跟在 Codex preamble 之后。没有任何技能文件携带 Pi 行：`poteto-mode/SKILL.md` 只指向一次 `pi-tools.md`，技能的 Pi 注记是该文件中的一行。对 Copilot，它还从 `models.json` 盖上 `skills/setup-pstack/scripts/sheet.awk` 里的角色列表、`setup-pstack/copilot.md` 里的角色到档位表，以及 `hooks/session-start-copilot.md` 里的角色模型技能列表。它校验每个技能的 `name` 和 `description`，然后用[斜杠命令表](docs/reference.md#slash-commands)为每个公开技能生成 Codex prompt。

它还把 `PORTABLE_ASSETS` 中的四个文件复制进纯技能安装，并删除过期的生成文件。`NOTICE-skills.md` 提供随这些技能一起分发的声明。

生成器会拒绝缺失的 Markdown 链接、指向技能树之外的链接，以及要求打开不可达文件的指令。它检查游离的模型名，要求存在匹配的 `CHANGES.md` 标题，并校验 Codex marketplace 与 Claude hook 路径。它还强制执行这些规则：

- 不得有 `commands/` 目录。
- 技能上不得有 `disable-model-invocation`。
- 每个 `principle-*` 叶子都设置 `user-invocable: false`。
- 插件 agent 使用其带命名空间的 `pstack:<name>` 名称。

CI 运行 `bun tools/generate.mjs --check`，它不写入任何内容，并在每个过期、缺失或孤儿的生成文件上失败并点名，所以要提交生成器的输出。本地运行同一命令即可预览 CI。它检查工作树而非提交，所以生成目录中被 gitignore 的文件（如 `.DS_Store`）只会在本地失败，而未提交的重新生成输出也只会在本地通过。在克隆中运行过 `prek install`（或 `pre-commit install`）之后，`.pre-commit-config.yaml` 会在每次提交前运行同一条命令。

新增技能时，在 frontmatter 中包含 `name` 和 `description`。公开技能还需要斜杠命令表中的一行。该行提供 Codex 菜单描述与排序。生成器会报告任何缺少行的技能，或任何没有对应技能的行。

模型默认值在 `models.json` 中修改，绝不要在技能正文中改。角色从 `tiers` 中指名一个档位（`default`、`strongest` 或 `panel`），所以移动档位只需改一处；`codex` 块给出每个档位的 Codex 示例；`pi` 块按 Pi 提供方各映射一次，把每个可用族名映射到 Pi 模型 ID，并用一个 `fallback` 提供方承接其他提供方的会话。生成器在加载时检查配置结构（`tools/generate.mjs` 中的 `parseModels`），并在失败时点名违规的角色、档位或 slug。`tests/models.test.mjs` 证明每种畸形都被拒绝，并检查技能点名了它们用到的每个角色。完整 `claude-*` ID，或生成区域之外反引号包裹的可用名（例如 `` `fable` ``），会让生成器带文件和行号失败。

`bun test tests/` 覆盖生成器、同步工具、链接校验器，以及 `tests/invariants.test.mjs`——后者构造必须触发每条布局不变量的 fixture 树。有一项检查是行为性的，位于 `tests/skill-collision-repro.sh`：它需要 `claude` CLI 和 API 访问，并发起一次 haiku 调用，以证明在没有 `commands/` 的情况下，用户手打的 `/plugin:name` 仍能触达技能。CI 无法运行它，所以至少在发布前本地跑一次。

如果你改动了 `plugins/pstack/pi/` 或 `pi-tools.md`，还要对一个已登录的 `pi` 1.1 跑 Pi 检查。`bun tools/typecheck-pi.mjs` 在 strict 模式下，对照已安装 Pi 包自身的类型来类型检查扩展；它优先用 `PI_PACKAGE_DIR`，否则依次查找全局 npm root、Homebrew 的 `pi-coding-agent` formula（`$(brew --prefix)/opt/pi-coding-agent/libexec/lib/node_modules`），以及 bun 的全局目录（`BUN_INSTALL_GLOBAL_DIR`、`$BUN_INSTALL/install/global`、`$XDG_CACHE_HOME/.bun/install/global`、`~/.bun/install/global` 和 `~/.cache/.bun/install/global`），`tests/pi/catalog.test.mjs` 与 `tests/pi/runtime.test.mjs` 通过 `tools/pi-package.mjs` 也这样查找。CI 的 `Pi extension types` job 把固定版本装进全局 npm root。`PSTACK_PI_LIVE=1 bun test tests/pi/live.test.mjs` 用真实 `pi` 进程驱动扩展，约需五分钟。它会发起真实模型调用，默认用自带的 `openai` 模型。设 `PSTACK_PI_LIVE_PROVIDER=anthropic` 改调 Claude 模型，或设 `PSTACK_PI_LIVE_MODELS` 为一行 `pi models:` 来替换个别模型。

如果你改动了 `plugins/pstack/hooks/`、`copilot-tools.md` 或 `setup-pstack`，请对一个已登录的 `copilot` CLI 运行 `tests/copilot-smoke.sh`。它把检出安装进一次性的 `HOME` 和 `COPILOT_HOME`，消耗约二十次 premium 请求，在缺少 `copilot` 时跳过。设 `SMOKE_GITHUB=<owner>/<repo>@<ref>` 可在已推送 ref 的 GitHub marketplace 安装上重跑插件文件检查。

如果你改动了 `skills/poteto-mode/scripts/`：

```shell
cd plugins/pstack/skills/poteto-mode/scripts
bun install --frozen-lockfile
bun run typecheck
bun test orch watch-pr check-plan.test.ts
bunx prettier@3.6.2 --check .
```

脚本使用上游在 `.prettierrc.json` 中的 Prettier 3.6.2 设置，CI 运行同一项检查，`.prettierignore` 列出检查跳过的每个文件及原因。

`ship-pr` 有一项 CI 无法运行的检查。`watch-pr/live-merge-safety.mjs` 在你的 `gh` 账号上创建一个私有仓库，针对过期的 head、移动过的 base 和改过目标的 base 驱动 `ship-pr inspect` 与 `cancel-pending`，确认 GitHub 会拒绝在过期 SHA 上合并，合并当前 head，然后删除仓库。删除需要 `gh` token 的 `delete_repo` 权限；失败时脚本会打印手动删除仓库的命令。在改动 `watch-pr/` 的发布前，从同一目录运行：

```shell
bun watch-pr/live-merge-safety.mjs --live-disposable
```

如果你改动了工作流，推送前先审计：

```shell
uvx zizmor@1.29.0 --persona pedantic --min-severity low --collect all -- .
```

`--collect all` 与 CI 扫描的范围一致。只把 zizmor 指向 `.github/workflows/` 会跳过 `dependabot.yml`，于是本地运行对 CI 会失败的发现给出干净结果。

<a id="things-that-will-fail-ci"></a>
## 会让 CI 失败的事

- **一个 `plugins/pstack/commands/` 目录。** Claude Code 把命令和用户可调用的技能渲染进同一个斜杠菜单，所以与技能配对的跳板会让每个 `/pstack:<name>` 行重复（[#22](https://github.com/michael-denyer/pstack-claude/issues/22)）。Codex 存根位于 `plugins/pstack/.codex-plugin/prompts/`。上游同步会试图重新引入 `commands/`；把任何新存根移过去。
- **技能 frontmatter 中的 `disable-model-invocation`。** 在技能上，它会让 Skill 工具直接拒绝调用，从而破坏 SessionStart 指令。`principle-*` 叶子改用 `user-invocable: false`。
- **过期的生成输出。** `Generated files current` job 运行 `bun tools/generate.mjs --check`，并在每个待写入的生成文件或待删除的孤儿上失败并点名。改了 `VERSION` 却没重新生成、手改清单的 `version` 字段、或升版本却没有匹配的 `CHANGES.md` 标题，都会落在这里。同一次运行还校验 `hooks/hooks.json`、`.codex-plugin/plugin.json` 点名的 Codex hooks 文件，以及 `.github/plugin/plugin.json` 点名的 Copilot hooks 文件：命令点名的每个 `${CLAUDE_PLUGIN_ROOT}` 路径，或 Copilot 文件中的 `${COPILOT_PLUGIN_ROOT}` 路径，都必须存在于插件中。
- **未解决的同步冲突。** `bun tools/generate.mjs` 会在 `plugins/pstack` 下任何以七个 `<`、`|`、`=` 或 `>` 开头、后接空格或行尾的行上失败，并点名文件和行号。同步会为每个文本冲突写入 `<`、`=` 和 `>` 标记。`|` 行来自在 `diff3` 冲突风格下手工执行的合并。
- **缺失或逃逸的本地 Markdown 链接。** `tools/validate-skills.mjs` 相对于其 Markdown 文件解析裸路径、`./`、`../` 以及引用式目标。每个本地目标都必须存在于 `plugins/pstack/skills` 内。
- **正文点名了安装不携带的插件文件。** 同一工具还会相对其 Markdown 文件和插件根解析每个反引号包裹的相对路径。落在 `plugins/pstack/skills` 之外真实文件或目录上的 token（`agents/comment-sicko.md`、`../../hooks/hooks.json`）会失败。解析不到任何东西的 token（占位符、斜杠命令、`plugins/pstack/models.json` 这类维护者注记）通过。Markdown 链接由链接检查捕获；这一条覆盖的不是链接的反引号形式。
- **未通过 shellcheck 的 shell 脚本。** `node_modules` 之外的每个 `.sh` 文件都按 warning 级别 lint。
- **固定在 tag 上的 action。** 使用完整的 40 字符 commit SHA 并附版本注释。可变 tag 可能被强推进我们的 runner。
- **未通过 `actionlint` 的工作流文件。** 非法 YAML、畸形表达式、未知 runner 标签，或指向不存在 job 的 `needs:`。从仓库根运行 `actionlint`。
- **Markdown 正确性错误。** 反写的链接语法、空链接目标、缺失的图片 alt、指向不存在标题的片段链接，或未定义/未使用的引用定义。规则集刻意只含正确性规则，位于 `.markdownlint-cli2.jsonc`。运行 `npx --yes markdownlint-cli2@0.18.1 '**/*.md'`。
- **任何 Markdown 文件中的相对链接断裂。** 链接 job 离线解析文件与片段目标，绝不访问远程 URL，所以外部链接腐坏不会让你的 PR 失败。运行 `lychee --offline --include-fragments --exclude-path node_modules --exclude-path .git '**/*.md'`。

<a id="dependency-updates"></a>
## 依赖更新

Dependabot 保持固定的 action SHA 最新。vendored 脚本唯一的运行时依赖（`commander`）跟随上游的固定版本，随 `tools/sync.mjs` 移动；`osv-scanner` 每周扫描每个 `bun.lock`，所以 CVE 仍会浮现。根 `bun.lock` 固定 `typebox`，生成器和 Pi 扩展测试都会导入它。如果你手动升级它，请运行 `bun install` 并在同一个改动中提交生成的 `bun.lock`。

<a id="releasing"></a>
## 发布

插件自动更新**按版本号**安装，而不是跟踪 `main`。一个没有升版本就合并的技能修复，在每个已安装副本上都是惰性的，因为更新器看到的还是它已有的版本，什么也不做。

所以：任何改变技能行为的 PR，要么自己升版本，要么后面跟一个升版本的发布 PR。升版本分三步：编辑根 `VERSION` 文件，在 `## <version>` 标题下加一条 `CHANGES.md` 记录说明改了什么以及为什么，然后运行 `bun tools/generate.mjs` 给清单盖章。三步漏任何一步都会让 CI 失败。合并前运行完整的不变量脚本（含行为那一段）。

<a id="commit-and-pr-style"></a>
## 提交与 PR 风格

- 解释改了什么以及为什么。diff 已经显示了怎么改的。
- 一个 PR 只做一件事。同一个 diff 里的行为修复和重构是两次审查，放在一起审查意味着两次都没审好。
- 只声称你验证过的，并点名检查。"52/52 bun tests pass" 胜过 "tests pass"；"did not run the behavioral leg" 胜过沉默。

<a id="reporting-bugs"></a>
## 报告缺陷

请包含 pstack 版本、Claude Code（或 Codex、Pi、GitHub Copilot）版本，以及复现步骤。[#22](https://github.com/michael-denyer/pstack-claude/issues/22) 是值得照抄的范本：它点名了版本，给出了编号步骤，并包含隔离出原因的实验。

在提交之前，先在最新发行版上复现缺陷，因为只有该版本会收到修复。[缺陷报告表单](.github/ISSUE_TEMPLATE/bug_report.yml)会询问其中每个字段。通过 `gh` 或 API 提交的 issue 会跳过表单，所以请按同样的顺序给出同样的字段。
