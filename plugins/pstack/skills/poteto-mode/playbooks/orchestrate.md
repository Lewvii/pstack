### Orchestrate

Resolve the driver skill through [poteto-mode's Non-negotiables](../SKILL.md#non-negotiables).

**你负责的是整个项目，绝不是代码。编写 brief，清空队列，保持 frontier 为绿，做决策。** 适用于把整个项目交给一个常设协调者聊天的情形：持续多天，大量堆叠 PR，数十到数百个 subagent，人类每天查看两次而不是每五分钟一次。把单个任务推进到某个判定条件，用 Autonomous run。一次需要定制工作流的大型运行，用 figure-it-out。当工作的寿命超过任何单个 agent 时，才走这里。一个 agent 在会话预算内就能完成的工作，不算项目。

流程的繁简必须与项目规模相称。对于廉价且几乎相同的单元，按各节的指示精简流程。

其余一切都建立在三条规则之上。

- 完成通知是队列事件，不是中断。
- 每次 spawn 和每次 resume 都原样携带 standing orders。
- brief 就是产品。含糊的 brief 会悄无声息地失败，因为 worker 无法向你提问。

<a id="roles-and-placement"></a>
#### 角色与部署

- **Coordinator（本聊天）。** 负责框定问题、编写 brief、清空 inbox、负责向人类报告、做判断。它从不编写或修改代码。有冲突的合并、链式 rebase 和代码改动一律作为任务派出。在本地 git 操作代价低的仓库上，机械地合入一个已验证的单元（快进或干净地 cherry-pick worker 的 commit，然后 push）属于簿记工作，coordinator 可以自己做。把完成的工作排在一个空闲的 stacker 后面，是截止时间到了却一无所获的原因。整个循环端到端由 agent 完成。agent 只通过 `Agent` 工具来 spawn、resume 和收取结果。按 Subagents 一节，为每个 subagent 选择与任务匹配的推理级别。状态的读写在清空点通过 `orch` CLI 完成，一条命令进，一行输出出。该 CLI 从不 spawn、等待或唤醒任何东西。
- **Sub-coordinator。** 常驻，每条 track 一个，且只在项目规模超出单个 coordinator 的清空能力时才设立。coordinator 自己就能清空的 track 不需要中间层。每多嵌套一层，就要重新付出一整套定向开场的成本，而一个阻塞式的 sub-coordinator 会把它的子 agent 藏起来，让父级空等。它负责本 track 的单元和看板，编写其 worker 的 brief；在运行环境允许 subagent 再 spawn 的地方，自己 spawn worker 和 verifier。不允许的地方，它直接负责本 track 的单元，并保持同样的 review 分离。在批次边界汇总上报。绝不转发子 agent 的原始报告。在途子 agent 的上限是一次清空能处理的数量，大约十个，以滚动窗口方式维持。绝不用阻塞式批次，那样每个批次都要付出最慢子 agent 的代价。
- **Worker / verifier。** 后台 subagent（`run_in_background: true`）。Claude Code 没有远程 worker 环境，所以隔离靠每个写入者一个 worktree 或分支，而不是单独的机器。运行时验证通过 driver skill 进行，brief 中写明解析后的 skill 路径或确切命令。subagent 看不到本聊天，所以它的 brief 要内联所需内容，或指向仓库和 store 中的路径。宁可用更少、范围更宽的 worker。每个 worktree 或分支只有一个写入者（principle-separate-before-serializing-shared-state）。单元的 verifier 要使用与其 worker 不同的模型家族。

层级深度保持为 coordinator、track、worker 三层。按项目设计 track 划分（构建、合入和验证是常见的切法，不是必须的结构）。硬编码的 swarm 树试过，因过于僵化而被搁置。

<a id="store-layout"></a>
#### Store 布局

在仓库之外、会话之外创建 `~/.claude/orchestrate/<project-slug>/`。store 的寿命必须超过本聊天：项目要运行好几天，会话重启是预料之中的（见"存活与故障"），而且 store 就是事后复盘的材料。会话 scratchpad 再方便也不能用。它属于会话范围且是临时的，一次重启就会把项目唯一的状态一并带走。每个文件只有一个写入者。owner 发布事实，读取方在读取时汇总。簿记使用已安装插件下的 `bun skills/poteto-mode/scripts/orch/orch.ts`（下文写作 `orch`），而它的规范格式是纯 TSV 和 JSON，不用 CLI 也能读。

- `preferences.md` 是 standing orders 登记表：带编号的行，每行一条约束（模型策略、stack 形态和数量、验证标准、禁止的路径、升级策略）。对于人类的裁定，按 [Claims about human decisions](../../why/references/epistemics.md#claims-about-human-decisions) 保留原始措辞和出处。标注 agent 的解读以及缺失的原文。副本不是新的决定。把它原样粘贴进每次 spawn 和每次 resume。指令会在多次 resume 中逐渐衰减，每丢一条就要多花人类一轮对话。当你发现自己在重复某条指令时，先把这一行追加进去，再行动（principle-encode-lessons-in-structure）。
- `overview.md` 是持久的 PR 和 issue 数据库。只追加。绝不因为某个事件就整体重写。
- `units.tsv` 每个单元一行：id、track、state、branch、PR、head SHA、brief path。原地更新行。
- `frontier.json` 是计算得出的合并 frontier，见"Stack 安全"。
- `ledger.tsv` 是验证账本，见"验证"。
- `inbox/` 存放完成通知的指针。`gates.md` 存放挂起的人类门禁（问题、选项、无回复时的默认值）。
- `decisions.tsv` 是通过 show-me-your-work skill 留下的记录。
- `status.md` 在每次清空时由 `units.tsv` 和 `ledger.tsv` 派生，绝不手工维护。从表格重新生成它，而不是往里面叙述事件。

<a id="the-brief"></a>
#### Brief

你给 agent 的 prompt 是你唯一的产品，一份草率的 brief 会在整棵树中层层放大成垃圾。每次 spawn 都携带它的全部内容。填不出的字段，说明这个单元还没界定清楚。

```
GOAL         one sentence, the outcome, executable by a stranger with no chat access
SCOPE        paths this unit may write; paths it may not; its exclusive worktree or branch
CONTEXT      pointers to files and PRs; upstream reports pasted in full when this unit
             depends on them, because workers cannot see siblings
ACCEPTANCE   checkable criteria, one per line
VERIFY       exact commands or the resolved driver skill path, plus known gotchas
TIMEBOX      rough cap on runtime; on expiry, return partial findings and stop rather than run on
FORBIDDEN    no PR retargets or merges, no rebase, no force-push, no fixes outside scope,
             plus unit-specific bans
REPORT       status, branch, head SHA, PRs, verdict, what you actually ran, deviations,
             suggested follow-ups
STANDING     <preferences.md pasted verbatim>
```

brief 的大小要与单元相称。只有一条命令的单元，把模板压缩成一段话，但仍要写明目标、范围、验证命令和报告格式。为两行改动套上 4KB 的脚手架，编写和遵守它的成本都比改动本身高。spawn 时可以通过 store 路径引用 standing orders 文件。原样粘贴用于每次 resume，以及任何必须独立成立的 brief。

sub-coordinator 的 brief 额外包含它的 track 边界和单元列表、spawn 预算、清空协议，以及汇总格式（每个子 agent：名称、状态、PR、head SHA、结论、一行说明，外加 track 状态和 frontier 变化量）。

依赖关系是上下文的传递，不只是先后顺序。未声明的上游上下文会让 worker 去猜。缺少字段就拒绝 spawn。每个批次中，对每个 sub-coordinator 抽查一份 worker brief，与被抽查的批次同时进行，绝不作为挡在它前面的门禁。抽查不通过时，停掉该 track，修正 sub-coordinator 的指令，而不只是修正那个 worker，因为 brief 的质量会在运行后期下滑。绝不通过链式 resume 延续 brief。以合并后的范围重新 spawn 一个新的。

<a id="steps"></a>
#### 步骤

1. **框定。** 把完成判定写成可计数的形式（"all 126 units merged, each ledger-verified `unit-test-verified` or better"）。量化范围：单元数、大致工作量、预期的 stack 数量，以及墙钟预算。如果一个 agent 在这个预算内就能完成，到此为止，改用 Autonomous run。精简不能依赖其他文档是否存在。它的含义是：直接在本会话中完成工作，在有帮助的地方用普通 worker，验证就地进行，边做边合入，不用下面的 store、登记表或试点机制。对照预算安排合入。到预算的大约 70% 时，停止 spawn，把已验证的内容合入。按项目命名 track。有争议的划分或单向门决策，在试点之前先过 arena skill。框定只陈述一次。可逆的准备工作不必等待，直接进行。
2. **安装运行时。** 运行 `orch init`。通过 show-me-your-work skill 开启记录，在任何 spawn 之前写好 standing orders，并用 `orch frontier set --repo <repo-dir>` 从现有 PR 初始化 `frontier.json`；forge 为 Origin 时加上 `--forge origin`。
3. **试点。** 让一个单元走完整条路径：brief、worker、验证、进入 stack、账本记录、合并。试点的意义在于，趁代价只是一个 agent 而不是五十个时，证伪 brief 模板、验证配方和单元粒度。在任何扇出之前，依据试点证据修正约定。试点规模要与单元相称。对于由几乎相同的廉价单元组成的项目，第一个单元就是试点，按普通单元运行并就地执行验证命令，它一合入就开始扇出。专门的试点流程（独立的 verifier agent、审计门禁）适用于昂贵或新颖的单元形态，不适用于克隆式单元，因为串行试点在那里没有可证伪的东西。
4. **扩展。** 以滚动窗口 spawn worker，直到在途上限，子 agent 完成一个就补一个。阻塞式批次要为每批中最慢的子 agent 买单。只有超过"角色"中所说的单次清空阈值时，才 spawn track 的 sub-coordinator。每次清空后重新计算就绪的工作。把上游报告传递进下游 brief。兄弟之间的通信只能向上走。brief 抽查与被抽查的批次同时进行，不通过时阻止下一次补充，而不是当前这一批。
5. **清空。** 在每个清空点执行下面的队列规则。
6. **合入。** 合入是持续进行的，绝不是最后一个阶段。集成从第一个已验证的单元开始，与剩余批次并行。在重型仓库上，stacker 从第一批起就是常设角色，单元一验证就集成。在本地 git 代价低的仓库上，coordinator 按"角色"一节自己合入已验证的单元。在处理 stack 上层工作之前，保持 frontier 为绿。以"Stack 安全"为准。只在合并发生或收到报告的新 head SHA 时推进 `frontier.json`。
7. **收尾。** 清空最后的 inbox；把每个 spawn 过的 agent 对到一个终态行（done、abandoned、zombie-reconciled）；在真实产物上确认判定条件；确认每个已合入的 PR 都有针对其当前 head SHA 的结论；按 show-me-your-work 审计记录，包括其跨模型 review；把反复出现的纠正固化到 `preferences.md` 或 brief 模板中。保留 store 原样，它就是事后复盘的材料。

<a id="queue-and-drain"></a>
#### 队列与清空

- 收到完成通知时，运行 `orch inbox push <agent> <unit> <status> [--report PATH]`，然后回到手头的事。绝不就地深入 review。需要 review 的完成结果变成一个 verifier 单元。绝不在清空过程中 review diff。
- 在四个时点分批清空：一个关键段结束时、一次 track 汇总时、frontier 监视器唤醒时（通过 loop skill 设置，并配一个长间隔的心跳兜底），以及向人类报告之前。每批以 `orch inbox drain` 开始。清空过程中到达的通知等下一次。
- 需要先完成的关键段：编写 brief、一次 stack 操作、一次冲突决策、写一个门禁、更新账本或 frontier。
- 每次清空都要给每个指针分类（landed、needs-verify、failed、zombie、noise），通过 `orch unit add`、`orch unit set` 和 `orch ledger record` 写入相应的行，运行 `orch status`，然后在一条消息中 spawn 下一批。
- 在每条 track 汇总时，清点每个 spawn 过的子 agent：已到达、已重新 spawn，或其范围已被明确吸收。悄悄重做缺失子 agent 的工作，既掩盖了浪费的开销，也掩盖了它的结果本应填补的覆盖缺口。
- 清空这一轮以 `orch status` 输出的三行结束：各状态的计数、发生了什么变化、哪些门禁开着。细节放在 `status.md`。完整的回复约定适用于检查点和收尾。

<a id="stack-safety"></a>
#### Stack 安全

- frontier 是计算出来的对象，绝不是叙述。每次合并和 stack 变动后重新计算 `frontier.json`：有序的 PR 列表、分支名、head SHA、代号（generation number）、最低的未合并 PR。该命令沿着 PR 的 base 链遍历到检出目录的当前分支，所以链的顺序和 PR 状态来自 forge。默认使用 `gh`。项目确定的 forge 为 Origin 时，传 `--forge origin`。head SHA 来自本地分支引用，所以要在 stacker 的克隆中解析。没有 PR 的分支会报错，而不是去猜。
- 每个 stack 只有一个 stacker 可以变更拓扑（链式 rebase、force-push、改 base、合并），并在该 stack 内串行执行。把持有者记录在 standing orders 中。在这种规模下，链式 rebase 很慢，并且会阻塞执行它的人，所以给它一个独立单元，让 coordinator 置身事外。
- worker 绝不 rebase，也绝不改 PR 的目标或合并 PR。babysitter 遵循 `playbooks/babysit.md`，每个 stack 一个，范围限定在一个不可变的 frontier 代号上。它们把冲突报告给 stacker，而不是自己 rebase。
- 关闭 PR 和改目标只能通过 stacker。关闭一个 base PR 会让其上方的整条链变成孤儿。合并和 stack 手术和其他工作一样，都是带 brief 的单元。
- 设一个回顾监视器，跟踪已合并的 PR 是否被 revert、合并后 CI 是否坏掉、是否有被遗弃的后续工作。

<a id="verification"></a>
#### 验证

验证的力度要与单元相称。当 VERIFY 只是一条廉价命令时，由 worker 运行并报告输出，coordinator 抽查凭据。专门的 verifier agent（与 worker 使用不同的模型家族）用于验证代价高、需要判断或影响范围大的单元。如果一个 verifier agent 的全部产出只是重跑一条命令，那是走过场，不是验证。

用 `orch ledger record` 写账本行。用 `orch ledger check` 检查当前 PR 和 head SHA。`ledger.tsv` 每个结论一行，以 PR 编号加 head SHA 为键：`live-ui-verified | unit-test-verified | type-check-only | verifier-blocked | verifier-failed`。CI 全绿是结论的输入，不是结论本身。行为类工作需要高于 `type-check-only` 的结论。`verifier-blocked` 不算通过，环境恢复后重新 spawn。`verifier-failed` 要派一个修复单元，而不是重新验证。worker 可以自报结果，verifier 在同一个键上覆盖它。新的 head SHA 会使该行作废，所以 restack 之后要重新验证。"是否验证过"由账本回答，而不是靠记忆，也不是靠对话记录。

单元的产出必须在合入的那一刻就外部化，否则不算完成，绝不攒到运行结束时一起做。worker 推送它的分支，verifier 写下它的账本行，凭据落进 store。只存在于某一台 VM 上的工作，在那台 VM 死掉时就等于从未完成。

<a id="liveness-and-failure"></a>
#### 存活与故障

- 绝不为了查看状态而 resume 一个 agent。resume 会重启一个空闲的 agent。以只读方式探测：账本、`units.tsv`、`gh`、已推送的分支，以及后台任务列表。对话记录的 mtime 不代表存活。
- 无声死亡的 agent 在 inbox 中得到一条合成的复盘行（单元、故障模式、最后的证据、选项）。证据一到就据此重新规划。绝不等到全部静止。
- 按故障模式重试：cap-hit 或 oom，缩小范围后重新 spawn。network-drop，原样重试。tool-error，换一个模型重试。unknown，重试一次。两次重试之后，放弃该单元，绕开它重新规划。
- 几小时后才返回的 zombie，要先与当前的 frontier 和账本核对，才能接受任何东西。通过一个新单元抢救其独有的发现，绝不盲目合并。
- 当继续 spawn 会在整棵树上产生垃圾时（上游输出有问题、验收标准失效、基础设施宕机），在 standing orders 顶部写一条停止线，让在途工作完成，修复原因，再清除停止线。
- 像约束子 agent 一样约束你自己的基础设施重试。连续几次工具中止之后，停止重试。把终态交接写入持久状态（完成了什么、放在哪里、恢复所需的确切命令），然后结束运行。
- 会话重启后，在途的 subagent 都已死亡，但已推送的分支、打开的 PR 和 store 还在。重新读取 standing orders 和 `units.tsv`，重新计算 frontier，按 PR 和分支（而不是 agent id）重新挂接在途工作，根据存储的 brief 加上当前状态为每条 track 重新 spawn 一个 sub-coordinator，清空，继续。死掉的会话留下的 store 锁会在下一次写入时自动清除：`orch` 会替换持有者 pid 已不存在的锁。

<a id="escalation"></a>
#### 升级

需要上报给人类的事项，批量汇总到状态页，而不是逐条上报：不可逆操作（force-push 到共享分支、部署、删除、关闭别人的 PR）、任何实验都无法解决的真正产品或偏好问题、与观察到的现实相矛盾的 standing order、重新规划后仍然存在的项目级死路。提问之前先把每一项作为 `gates.md` 条目挂起，并让工作绕开它。

绝不上报给人类的事项：frontier 推进、rebase 机制、重试、CI 偶发失败的分诊、review 线程的分诊、格式修复、brief 已经禁止的范围（拒绝并继续），以及"我该不该继续"。拿不准时，先做并记录。

运行中途发现的问题，只修阻塞 frontier 的部分。其余的都挂到后续工作中。在这种扇出规模下，一点小小的范围泄漏就会成倍放大成没人要的 PR。

**回复：** 在检查点和收尾时：判定条件以及根据 `units.tsv` 和 `ledger.tsv` 得出的对应计数，各条 track 及其合入的内容，frontier（PR 列表加 SHA），结论汇总，放弃了什么及原因，等待人类处理的门禁（唯一需要人类回答的事项），store 路径，以及记录路径。数字取自表格，不要叙述。附上 PR 链接。
