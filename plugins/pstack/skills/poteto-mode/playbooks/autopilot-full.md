### Autopilot-full

Resolve the driver skill through [poteto-mode's Non-negotiables](../SKILL.md#non-negotiables).

**你负责的是结论（verdict），不是 PR。每个 PR 由一个 owner 从构建一路带到 merge-ready。在你的 swarm（它从未编写这些代码）证明了将要合入的代码、且 operator 点下合并之前，任何东西都不会合并。**

一个结论只覆盖一个补丁，一次绿色 CI 只覆盖一个 head。第 4 步只跑一次完整 swarm，只有出现重大改动时才再跑。`playbooks/shipping.md` 中的 patch-id 规则可以沿用某条 lane 的结果，但每个新 head 都需要自己的绿色 CI。

1. **operator 的门禁优先。** 授权只覆盖队列中的条目，所以 operator 指明归自己的条目以及队列之外的 PR 都停在 merge-ready。要求陈述计划不等于放行。operator 放行后，把完整的项目目标写入 standing orders，并在你的 todolist 中复述，让它在各轮对话中一直有效，直到队列完成。第一个 owner 启动之前，告诉 operator 项目的规模：owner 的数量、第一轮的 verifier lane 数量，以及第 4 步规定的每个 PR 最多两次 fix-forward。A program of more than three owners waits for the operator's go on that size（超过三个 owner 的项目要等 operator 对这个规模放行），unless the operator's request already named it（除非 operator 的请求中已经写明了该规模）。
2. **每个 PR 启动一个 owner，并把它的工作放在替补能找到的地方。** 为整个项目确定一次 forge。默认使用 GitHub CLI（`gh`）。如果 `command -v origin` 成功且 Origin 能解析该仓库，所有 PR 操作都使用 `origin pr`。否则继续使用 `gh` 并记录这次回退。绝不强制要求 Graphite（`gt`）。每个 owner 是一个在独立 worktree 中运行的后台 subagent：它构建代码，按 `playbooks/opening-a-pr.md` 开 PR，在真实产物上证明改动，按 `playbooks/babysit.md` babysit 到全绿，并在第 4、5 步放行后，于 merge-ready 时交出 PR。没有 owner 执行合并。

   尽早开 PR，让 CI 和机器人与工作并行运行。只有经过去冗余处理（the **deslop** skill，`/deslop`）和 `/no-comments` 之后，代码才算就绪，这样完整 swarm 跑在最终代码上。然后 rebase 到当前 trunk，在自证之前报告 code-ready head，让验证跑在补丁将要合入的 base 上。之后的每个 head 都要报告；自证、CI 和 babysit 都完成后报告 merge-ready。修复轮次保持在这个 base 上，除非 trunk 产生冲突或弄坏 CI，因为每次 rebase 都会重启 CI。在 code-ready push 之前，以及在之后任何需要 verifier 的 push 之前，在已提交的 head 上运行仓库针对改动路径的检查。

   假设 owner 的会话会中途死掉。启动后尽快 push，并按 the **show-me-your-work** skill 在项目 store `~/.claude/orchestrate/<slug>/` 下开始一份决策记录，之后每完成一个值得保留的单元就 push 一次。谁启动 subagent，谁就在该 store 中记录它的 id、状态，以及根据同类任务以往运行得出的"超时即视为卡住"的运行时长，这样父 agent 死掉时 root 仍能探测它。只 force-push 你自己的分支，并用 `--force-with-lease` 钉住你最后一次 push 或接手时的 SHA；遇到任何不是你做的 push，停止并向 root 报告。
3. **owner 并行运行，都基于 trunk，绝不堆叠，** 这样每个 PR 都能独立合入。
4. **只做一次 swarm 验证。** 完整 swarm 只在 code-ready head 上跑一次。只有新行为、重新设计或大量新 diff 才再跑一次完整 swarm。rebase、合并冲突解决或小修复只需在新 head 上跑一个 verifier。按 the **swarm** skill，在 code-ready SHA 上展开相互独立的 verifier。它们重跑门禁，通过 driver skill 在真实界面上实测关键行为，并审计 diff。没有 live lane，或者在你审计 merge-ready 报告的凭据之前，结论都不算干净，因为任何 lane 都不信任自己没有核实过的说法。

   把每个已证实的发现（不论 lane 给它打了什么标签）在一次 fix-forward 中交给 owner，并附上一个覆盖同类缺陷所有位置的失败测试。A defect trunk shows without the PR is recorded against trunk and does not gate the PR（即没有该 PR 时 trunk 上也存在的缺陷记在 trunk 名下，不阻塞该 PR），但 PR 复制过来的或新触达的缺陷归 PR 自己。绝不亲自推送修复，因为每个分支只有一个写入者。owner 卡住时，换掉它。

   要求每条 review lane 只报告 diff 弄坏了什么，而不是它触及文件中的所有问题；除非 operator 要求，不要求做变异或模糊测试。Two fix-forwards per PR is the ceiling.（每个 PR 最多两次 fix-forward。）如果某个阻塞性发现在第二次之后仍然存在，停止该 PR 的工作，报告其为 blocked 并附上发现和证据，由 operator 选择再来一轮、改成更窄的 PR，或者关闭它。
5. **operator 只合并你验证过、并在它将要合入的 trunk 上测试过的东西。** 单靠绿色 CI 永远不能授权合并；你的干净结论是 operator 点击合并的前提，绝不是替代。swarm 的 lane 启动之后，合并准备阶段 rebase 到当前 trunk，报告新 head，并等待 CI 变绿。

   在交出之前，fetch trunk，判断它的变化是否会影响该补丁，因为 trunk 每动一次就 rebase 一次永远没有尽头。当将要合入的代码从未被测试过时，rebase 并等待 CI，例如出现冲突、两边都改了同一个文件，或者改动影响了该 PR 会运行哪些检查。两边都改了的 lockfile 不需要 rebase，前提是在合并结果上 frozen install 能通过，且 PR 改动的那些包在那里的解析结果与 head 上一致，因为这个 install 测试的正是将要合入的 lockfile；install 失败则说明必须通过 rebase 修复。当 trunk 改动了补丁的调用方或补丁依赖的东西（例如它调用的代码或某个依赖版本）时，在合并结果上运行补丁的检查，通过后交出。其他情况直接交出。如果 trunk 会迫使合并准备之后出现第三次 rebase，暂停并报告给 root，由 root 编排相互冲突的工作的顺序。

   owner 把 merge-ready 的 PR 连同结论和钉住的 head SHA 呈给 operator，然后返回；按 poteto-mode 的 Subagents 一节，由一个新的 owner 接手下一个条目。如果在 operator 合并之前 trunk 又有变动，按 `playbooks/shipping.md` 中的 patch-id 规则决定是否重新验证。
6. **运行 root 层。** 任何放宽门禁的操作，例如提高预算或加入 allowlist 条目，都需要 verifier 的证明，再加上你重新给出的会签，因为想要全绿的正是 owner 本人。在覆盖审批的授权下，这个会签就是审批，owner 对它的记录要指向你的会签，供 lane 核查。被禁用或软失败的检查一律移除，因为它对每种情况都不再起守护作用。绝不给出或绕过 forge 强制执行的审批，因为任何会签都覆盖不了它。

   operator 放行后，用动态模式下真实的 `/loop` 设置每小时一次的审计 tick（它会自己安排唤醒，而不是阻塞在 sleep 上），并一直保留到没有委派出去的工作为止，因为长时间运行会漂移，agent 也会悄无声息地卡住。每个 tick 都从已安装插件的磁盘上重新读取本 playbook 和常设目标，并修正漂移，例如未报告的 head 或未记录的 agent，做法是让对应的 owner 补录，因为漂移是记录中的缺口，不是卡死。它探测每一个 agent，直到叶子 lane，并依据副作用判断每个 agent 的状态，因为状态文本和摘要可能掩盖卡住的子 agent。一个 agent 在出错、无法开始新一轮、或超过其卡住阈值仍无任何新进展时，就算卡住；到第一个 tick 时既没有 push 也没有写记录行的 owner 同样算卡住。正在等你或 operator 的 agent 不算卡住，回应它即可。卡住的 agent 立即替换，不必等它的停止生效。为 operator 带来变化的 tick，以按 the **ketchup** skill 向其同步进展收尾；没有变化的 tick 保持安静。
7. **operator 叫停时立即撤下。** 暂停指令以零写入命令的形式送达每个 owner。每个 owner 报告自己的进度，并保留它的 brief，直到 operator 解除暂停。

**回复：** 按 the **ketchup** skill 向 operator 同步进展，涵盖每个 PR 的状态和结论、哪些已 merge-ready 等待 operator 点击、会签及其原因，以及尚未关闭的 operator 门禁，并附上每个 PR 和记录的链接。
