### Multi-phase or multi-PR plan

Resolve the driver skill through [poteto-mode's Non-negotiables](../SKILL.md#non-negotiables).

**你负责的是计划，不是代码。计划是一份清单：owner 逐项执行，operator 依据证据审计。** 计划本身就是交付物。不要动手实现。

1. 如果改动只涉及一两个文件且做法显而易见，跳过计划。说明这一点，然后停止。
2. 动笔之前，用原型解决未决问题。每个问题跑一遍 `playbooks/prototype.md`。保留分支、SHA 和截图，供附录 A 使用。只有在任何运行都无法回答的产品或偏好问题上才询问 operator，并给出选项（the **never-block-on-the-human** principle skill）。
3. 在 subagent 中探索，使用 `subagent_type: "pstack:poteto-agent"`，并按 Subagents 一节显式指定模型（the **guard-the-context-window** principle skill）。按 Subagents 一节，为每个 subagent 选择与任务匹配的推理级别。每个 subagent 返回文件指针、约定、测试命令和入口点。不要内联大段转储。
4. 把下方骨架复制到计划文件中，填满每个占位符。除非 operator 指定路径，否则把文件写在 agent store 下（`~/.claude/orchestrate/<slug>/docs/`）。保留每个标题和每个子块，并保持所示顺序。每个 PR 一节。一个 PR 是一项带有自身证据的改动（the **sequence-verifiable-units** principle skill）。在 **How to read this** 中写明执行 playbook。按 `playbooks/autopilot-stack.md` 末尾的规则，在 `playbooks/autopilot-full.md` 和 `playbooks/autopilot-stack.md` 之间选择。常设项目使用 `playbooks/orchestrate.md`。执行 playbook 负责 base 选择、拓扑变更和合并权限。不要把它的 rebase 步骤抄进计划。
5. 先完整按 `/technical-writing` 写作，再跑 `/unslop`。正文只用一种 Diátaxis 模式，即 how-to。附录承载解释和参考。每个标题陈述任务或结论。不用长破折号。句中不用冒号。
6. 在已安装的插件中运行 `node skills/poteto-mode/scripts/check-plan.mjs <plan.md>`，修掉它打印的每一行（the **encode-lessons-in-structure** principle skill）。
7. 交回。贴出计划路径和脚本输出，然后停止。执行要等 operator 明确放行后才开始，并遵循计划所指定的执行 playbook。

**验证。** Tests alone are not sufficient verification.（仅有测试不足以构成验证。）只有当一个 PR 的 unit、live 和 perf 三类复选框全部勾选时，它才算通过验证（the **prove-it-works** principle skill）。这句话就是验证规则，每个验证块都以它开头。live 块是必需的。Ten lanes on the configured `swarm workers` model at the PR head（即在 PR head 上用配置的 `swarm workers` 模型跑十条 lane），按 the **swarm** skill，通过 driver skill 驱动真实界面。每条 lane 是一个复选框，写明具体场景、它保存的截图以及通过判据。其中一条是 **Regression lane against trunk.**（针对 trunk 的回归 lane）。它在 trunk 和 head 上运行同一个关键场景。如果 trunk 还没有该功能，这条 lane 记录这一事实，改为对 diff 新增的行为以及用户等待的最终状态设门禁，而不是编造一个 trunk 结果。性能门禁是双边的：trunk 和 head 都必须产出指定的指标。如果 trunk 缺少该功能，还要单独隔离 diff 新增的工作，并为这部分工作以及用户等待的端到端状态设定绝对预算。不要在不同场景之间声称比值。perf 块要写明指标、交错执行的探针、先测得的 trunk 基线，以及带失败阈值的规则。改变交互的 PR 需要过 review 门禁：合并前，operator 在聊天中通过截图和视频进行 review。不改变交互的 PR 写 `**Review gate.** None. <PR id> is not review-gated.`，下面不放复选框。

**Driver skill。** 通过 Non-negotiables 选定它，并在每条 live lane 的启动配方中写入解析后的 skill 路径或确切命令。原生移动端使用仓库中现有的模拟器驱动 skill。触及两个界面的 PR 在两个界面上都要有 lane。没有 driver skill 的界面要作为风险写入附录 C，它的 live 块仍需写明每条 lane 如何驱动它。

````markdown
# <Program> plan

<Under ten lines. What changes, for whom, the rule the program enforces, and the PR ids in order.>

## How to read this

One box is one unit of work. Every box names the evidence that checks it. A nested box is a sub-step of the box above it. Check a box only when its evidence exists, a file, a log line, a screenshot, a test run, or a SHA. The body is a how-to. The appendices explain and record.

The program runs `skills/poteto-mode/playbooks/<execution playbook>.md` from the installed plugin. <Who merges, and which PR ids are the operator's items that stop at merge-ready.>

Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

## Program checklist

### Arm the program

- [ ] State the protocol and this plan to the operator, then stop. Start execution only on the operator's explicit go.
- [ ] On the operator's go, write this exact text into the standing orders and restate it in your todolist. "<The plan path, the PR ids in order, the verification rule, who merges, and the done condition.>"
- [ ] Read these from the installed plugin at program start. Re-read them at every tick.
  - [ ] `skills/poteto-mode/playbooks/<execution playbook>.md`
  - [ ] `skills/swarm/SKILL.md`
  - [ ] `<driver skill path>`
  - [ ] `skills/poteto-mode/playbooks/opening-a-pr.md`
  - [ ] `skills/<each other leaf skill the program uses>/SKILL.md`
- [ ] Arm the hourly audit tick as a real `/loop` in dynamic mode, which schedules its own wake-up rather than blocking on a sleep. Never leave the cadence to memory.
- [ ] Use this tick prompt, verbatim. "Re-read the execution playbook from the installed plugin and the standing objective. Audit the operation against both and fix drift in this tick. Probe every active lane and judge progress by side effects only. Stand down a stuck lane and dispatch its replacement now. Then post a short status message to the operator in chat only when the audit found a tracked change that no earlier status message reported, such as a PR opened, a code-ready head, a round launched or closed, a verdict, a merge, a stuck agent and the action taken, a blocker added or cleared, or a decision only the operator can make. Name every such change and nothing else. Do not repeat a table, the merged list, or an unchanged blocker. If the audit found none, end the turn with no reply text. Either way, log this tick's row in your decision trail. The row names the items reported, or none."
- [ ] On the operator's hold or stand-down, send every owner a zero-writes order at once.

### Spawn owners

- [ ] Spawn one owner per PR with the full lifecycle the execution playbook names.
- [ ] Follow this dependency graph. Start dependent work only after its parent merges, or base it on the parent branch when the execution playbook stacks.
  - [ ] <PR id> and <PR id> are independent and first. Both branch from `main`.
  - [ ] <PR id> after <PR id>.
- [ ] Hold the file boundaries. <PR id or class> touches only `<glob>`.
- [ ] Hold the review gate. <PR ids> change an interaction. They wait for the operator's review in chat with screenshots and a video before merge.

### PR mechanics, for every PR

- [ ] Resolve the forge once. Default to `gh`; if `command -v origin` succeeds and Origin can resolve the repository, use `origin pr` for every PR operation. Record any fallback to `gh`. Never require `gt`.
- [ ] Open the PR ready, never draft, per **Opening a PR**. Use the run's built-in PR tool when it has one, else `origin pr create --status open --base <base-branch>` or `gh pr create --base <base-branch>` according to the resolved forge. A stack child targets its parent branch.
- [ ] Run the repo's lint and typecheck once before the PR-facing push. Push with hooks on.
- [ ] Run `/deslop` before each commit and `/no-comments` before review.
- [ ] Triage every review-bot and security-reviewer comment per `../references/bugbot-triage.md`.
- [ ] Before the code-ready report and babysit, and again after merge prep, record the base and head SHAs prepared by the topology owner under the execution playbook.

### Verdict and merge, for every PR

- [ ] A full swarm runs once, at the code-ready head. Another full swarm runs only for new behavior, a redesign, or a large new diff. A rebase, a merge-conflict resolution, or a minor fix gets a single verifier at the new head. Run each full swarm per `skills/swarm/SKILL.md`. One gates lane. The ten live lanes from the PR's **Verify, live** block. The perf lane from its **Verify, perf** block. Two or more audit lanes, each with its own focus, that read the diff and the receipts and distrust the PR body. The root audits the receipts in the merge-ready report before the verdict.
- [ ] Clean only when every lane is `PASS`. Findings go back to the owner, including a defect that a lane filed as a note. A new head gets a fresh verdict, except for results that stay valid under the patch-id rule in `playbooks/shipping.md`.
- [ ] <The merge or append rule from the execution playbook, with the patch-id rule from `playbooks/shipping.md`.>

### Boot recipe, for every live lane

Each live lane runs in its own worktree at the PR head. Drive through the skill path or exact commands recorded in this boot recipe.

- [ ] `git fetch origin <head-branch> && git checkout <head SHA>`.
- [ ] <Start the backend and the surface. Wait for ready.>
- [ ] <Deliver input only through the driver skill's commands. Name the read-only diagnostics.>
- [ ] Save every screenshot to `/tmp/swarm-<pr-id>/worker-<n>/<slug>.png` and return the paths with the report.

## <Task as a verb phrase> (<PR id>)

**Depends on.** <PR id, or None.>

**Files.**

- [ ] Edit `<path>`.
- [ ] Create `<path>`.
- [ ] Delete `<path>`.

**Build.**

- [ ] <One change. Name the symbol and the file.>

**You see.**

- [ ] <One observable result, with the exact log line or screen state.>

**Verify, unit.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

- [ ] <Test file and the case it gains.> Run `<command>`.

**Verify, live.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked. Ten lanes on the configured `swarm workers` model at the PR head, per the boot recipe.

- [ ] Lane 1. Regression lane against trunk. Run <the same load-bearing scenario> at trunk and head. If trunk lacks the feature, record that and gate <the behavior the diff adds plus the end state the user waits for>. Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 2. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 3. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 4. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 5. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 6. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 7. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 8. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 9. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 10. <Scenario.> Save `<slug>.png`. Pass when <predicate>.

**Verify, perf.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

- [ ] Metric. <What is measured at both trunk and head. If trunk lacks the feature, also name the diff-added work and the end-to-end state the user waits for.>
- [ ] Probe. <The command or procedure, run at trunk and at the head, interleaved. Both sides must produce the metric.>
- [ ] Baseline. Record the trunk <value> first.
- [ ] Rule. <Head against trunk, with the number that fails. If the scenarios differ, add absolute budgets for the diff-added work and the user-visible end state instead of an invalid ratio.>

**Review gate.** The operator reviews before merge.

- [ ] Copy lane <n> screenshots into `<media path>/<pr-id>-review-<slug>.png`.
- [ ] Record a 30 to 60 second video of the change in a lane worktree. Save it as `<media path>/<pr-id>-review.mp4`.
- [ ] Post the screenshots and the video in chat. Stop at merge-ready. Wait for the operator's click.

**Merge.**

- [ ] Root's clean verdict at the exact head SHA.
- [ ] Bugbot triage done.
- [ ] Base and verdict are current under the execution playbook and the patch-id rule in `playbooks/shipping.md`.
- [ ] <The owner squash-merges its own PR, or the root appends it to the base-branch stack and the operator lands it bottom-up.>

## Close the program

- [ ] Every box above is checked with its evidence.
- [ ] Reply to the operator with the report the execution playbook names.

## Appendix A. Prototype evidence

<Each open question a prototype answered, with the branch, the SHA, and the artifact links. Each question that stays unproven.>

## Appendix B. Alternatives rejected

<Each approach weighed and why it lost.>

## Appendix C. Risks

<Each risk with the PR it lands in and what the owner watches.>

## Appendix D. Links and reading list

<Docs to read before editing. Which PRs get `skills/how/SKILL.md` and `skills/interrogate/SKILL.md`. The trail per `skills/show-me-your-work/SKILL.md`.>
````

**回复：** 计划路径，各 PR id 及其依赖和需要 review 门禁的集合，原型证明了什么、哪些仍未证明，以及检查脚本的输出。
