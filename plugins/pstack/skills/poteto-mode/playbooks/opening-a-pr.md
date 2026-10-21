### Opening a PR

在其他每个 playbook 的末尾调用。

**Worktree。** 在基于 main 的 git worktree 中工作。subagent 会继承它。同一分支上的多个 `Agent` 调用要么各自使用独立的 worktree，要么在两次调用之间执行 `git fetch && git reset --hard origin/<branch>`。分支脏了且混有无关改动：导出补丁，新建 worktree，再应用补丁。worktree 乱成一团：从 main 重置，以最小改动重做。在从某个 worktree 提交、合并或部署之前，列出存活的 agent，and stop every one that holds it, including grandchildren you never launched（即停掉每一个占用它的 agent，包括你从未亲自启动的孙级 agent）。委派者的子 agent 不会继承它的 brief，所以只读指令传不到它们那里。确认每一个都已停止，然后运行 `git status`，读一遍你即将发布的代码树。

**Commits。** 放开提交。开 PR 之前，rebase 成小而有序的 commit。每个 commit 都是未来的一个 PR：可以独立合入，顺序要讲清来龙去脉。修复属于刚做的那个 commit 时用 amend；能分离时新建 commit。commit 正文不要复述标题。

**PRs。** 提交前对 diff 运行 `/deslop`，review 前运行 `/no-comments`。每个 PR 标题、描述和 commit 正文都用 `/technical-writing` 来写（除 Diátaxis 外的所有层，并应用 STE 词汇规则：一个动作只用一个词，保留冠词，能用简单动词时不用 `-ing` 形式），然后再应用 `/unslop`。

**标题。** 采用 Conventional Commits 格式 `type(scope): subject`，末尾不加句号。type 可以是 `feat`、`fix`、`docs`、`refactor`、`test`、`chore` 或 `perf`。scope 是改动所在的区域，例如 `pstack` 或 `poteto-mode`。subject 要简短、用祈使语气，并在有真实符号承载改动时写出它的名字，例如 `fix(pstack): retarget opening-a-pr babysit trigger`。

**描述。** PR 正文是一分钟的简报，不是实验记录，读者是手里已有 diff 的 reviewer。它要说明改动为什么存在、刻意不包括什么、可能弄坏什么，以及你如何证明它可行。用简短、简单的句子，少用标识符，不要大段文字。正文会成为 squash commit 的正文，所以把那个 commit 控制在大约 40 行以内。使用 `##` 标题而不是加粗引导语，按以下顺序排列，没有内容的小节直接删掉：

- `## Why`：问题和做法，1 到 3 个短句。不写 SHA、rebase 谱系，也不写 "based on main" 之类的开场白。
- `## What changed`：1 到 3 个简短要点。只有当真实符号或路径承载了改动时才写出它。重命名或改目标时，两边都要写出来。
- `## Scope`：必写。1 到 3 个简短条目，说明覆盖了什么、刻意没包括什么（后续工作、已知缺口）。不写符号、路径，也不逐文件长篇大论。
- `## Tradeoffs`：只写 reviewer 否则会追问的被否决方案。没有真正的取舍时跳过。
- `## Blast Radius`：1 到 2 句，说明它影响到谁或什么，以及为什么安全或有风险。如果 main 是红的，说明让它继续红着的代价。
- `## Verification`：1 到 3 个要点，每个都是一条真实运行路径及其结果。性能：给出一个带单位的主要数字，写成 `before → after`，其余内容链接到 arena 或 swarm 目录。不写样本量方法论、swarm 复述或指标表格。

各小节之后，在视频或截图能证明某个结论时附上它们。完整 SHA、swarm 或 arena 的 lane 复述、杠杆修正长文、逐文件清单以及 "CLEAN" 结论，放进链接的产物里，不放进正文。

**Forge。** 在第一次 PR 操作之前确定 forge，并在 create、edit、view、watch 和 merge 中一直沿用这个选择。默认使用 GitHub CLI（`gh`）。如果 `command -v origin` 成功且 Origin 能解析该仓库，优先使用 `origin pr ...`。如果没有 Origin 或它无法解析该仓库，继续使用 `gh` 并记录这次回退。不要强制要求 Graphite（`gt`）。

**内置 PR 工具。** 当运行环境提供内置 PR 工具时，创建、编辑、改目标和标记 ready 都通过它完成，不要用 forge CLI。用 CLI 创建的 PR 会缺失该工具所跟踪的信息。该工具不覆盖的操作使用已确定的 forge；运行环境没有这类工具时，所有 PR 操作都使用已确定的 forge。

**规模与堆叠。** 宁可开五个窄 PR，也不要开一个大 PR。堆叠（stack）是一条 base 分支链。根 PR 以 trunk 为目标。每个子分支 rebase 到父分支的确切 tip 上，它的 PR 以父分支为目标。父 PR 合并后，按 `playbooks/shipping.md` 第 4 步把子分支移到 trunk 上。没有内置 PR 工具时，按已确定的 forge，用 `origin pr create --status open --base <parent-branch>` 或 `gh pr create --base <parent-branch>` 创建子 PR，用 `origin pr edit <pr> --base <parent-branch>` 或 `gh pr edit <pr> --base <parent-branch>` 改目标。只有独立的工作才从 trunk 拉分支。在进行大量堆叠工作前，先 rebase 到 trunk。

**就绪状态。** 每个 PR 都以 ready 状态打开，绝不开成 draft。内置 PR 工具可能默认 draft，所以通过它的每次创建调用都要设置 `draft: false`。使用 Origin 时传 `--status open`。使用 `gh` 时不要加 `--draft`。如果 PR 仍然以 draft 打开，通过 PR 工具把它标记为 ready，或按已确定的 forge 运行 `origin pr ready <number>` 或 `gh pr ready <number>`。在提及 PR 状态之前，先运行 `origin pr view <number>` 或 `gh pr view <number>`。

**Babysit。** 开 PR 不等于开始 babysit。贴出 URL，继续构建。先完成当前阶段或整个 stack。只有当整个 stack 都已存在、且用户要求时，才单独跑一轮 babysit。为每个新 PR 都 babysit 会拖住构建，并把检查花在后续批次会重来的 commit 上。当反馈偏离意图时，要提出异议。

开 PR 的 subagent 运行 `interrogate`、`/deslop` 和 `/no-comments`，并贴出 URL。它不做 babysit，直接返回父 agent，除非它是 Autopilot-full 或 Autopilot-stack 的 owner。那种 owner 的 brief 指派了 babysit 循环，这就是 `playbooks/babysit.md` 所等待的请求。该 owner 在发出 code-ready 报告后启动循环，按其 playbook 报告 merge-ready 或 STACK-READY，并且不受本文和 `playbooks/babysit.md` 中"整个 stack 建好之前不做 babysit"规则的约束。
