# 合并与重排栈的安全规则

这份参考与 [Shipping](../playbooks/shipping.md) 一起使用，并在 [Autopilot-stack](../playbooks/autopilot-stack.md) 做拓扑变更之前使用。CI 与阻塞分类继续由现有的 watcher 负责。这些操作增加的是修订与目标校验，它们不授予合并权限。

## 读取两种待处理机制

使用已安装插件的 `skills/poteto-mode/scripts/watch-pr/` 目录下的 `ship-pr`。它负责 GitHub 传输、解析、取消顺序与回读。watcher 和这个命令共享同一个 `LandingRevision`，其中包含仓库、PR 编号、head OID、base 分支和 base OID。

```sh
ship-pr inspect --repo "$owner/$repo" --pr "$pr" > "$record_file"
ship-pr cancel-pending --record "$record_file"
```

当命令不在 PATH 上时，用完整路径运行已安装的那一份。inspect 会输出 `kind: inspected` 和一条解析后的 landing 记录，其中包含两种待处理机制，以及存在时的 merge commit。把这条记录和验证证据一起保存。取消操作接受已保存的记录，重读当前状态，在存在时关闭 auto-merge，重读队列成员关系，需要时出队，并验证两种机制在同一个打开的 landing 修订上都不存在。它不合并、不推送、不改目标，也不授予落地权限。

只有以 `kind: cancelled` 退出 0，才允许执行计划中的拓扑操作。`changed`、`not-open` 和 `still-pending` 以退出码 1 返回观察到的记录。`unavailable` 以退出码 1 返回读取或变更的失败。缺失字段和不支持的 API 绝不意味着请求不存在。重写之前先把这些结果核对清楚；不要仅仅为了绕过不匹配就替换掉已保存的记录。

先确定受影响的依赖链，并协调它的拓扑写者。对每个受影响的 PR 做 inspect 和取消，包括上下文会随之改变的后代；不相关的 PR 不要碰。取消是一个被观察到的条件，不是防止其他参与者再次上膛的锁。状态一旦变化，就停下并对齐。Origin 必须通过它已安装的服务提供等价且经过验证的操作；这个命令是 GitHub 专用的，不得当作 Origin 支持来呈现。

## 保留并发写入与子分支改动

重写之前，用 `git ls-remote --exit-code origin "refs/heads/$branch"` 抓取远端分支 SHA，并校验返回的正是预期的那个 ref。把这个 SHA 留在操作记录里。仅靠隐式 lease 或一次先前的读取都不够；后台 fetch 可能移动跟踪 ref。

确认父级 squash 已落到预期目标之后，fetch trunk，找出这个子分支当初建立时所基于、且已记录在案的旧父级顶端。验证它是该子分支的祖先，并检查 `old-parent..child` 以确认这个范围里只有子分支的工作。如果不是，先重建边界再继续。绝不要从父级新的 squash commit 去推断这个边界。

```sh
git merge-base --is-ancestor "$old_parent_tip" "$child"
git rebase --onto "$trunk_tip" "$old_parent_tip" "$child"
git push --force-with-lease="refs/heads/$child:$captured_remote_head" \
  origin "HEAD:refs/heads/$child"
```

在子分支干净的 worktree 里执行 rebase，那里 `HEAD` 是准备好的子分支。这样会排除父级旧的提交。检查产生的 diff 与内容，重新评估审查是否仍然适用，并在新 head 上跑检查。如果 lease 拒绝了推送，就 fetch 并对齐并发的工作；不要刷新 lease 然后盲目重试覆盖。只在取消已被验证之后才改目标。准备更高层时，为每个剩余子分支保留已记录的边界。

## 服务端守住了什么

GitHub 的即时合并头条件是 `gh` 里的 `--match-head-commit`、GraphQL `mergePullRequest` 里的 `expectedHeadOid`，或 REST 合并端点里的 `sha`。用其中之一，携带经过验证的确切 SHA。这些都不提供对 base 分支的条件。在操作前后紧邻的时刻观察 base，并协调改目标；不要把这两次观察说成是原子的。

队列或自动请求的生存期可能超过它的准入修订。只有当既有仓库规则对最终修订和合并上下文强制执行了所需的验证时，才使用它。一句散文结论或旧 head 上的绿色勾不算这种门禁。如果这些保证无法建立，就继续盯着一次带保护的即时合并。如果仓库要求走队列，就在这道未满足的门禁处停下，而不是绕过保护。

合并之后，确认状态为 `MERGED`、最终 head 是被批准的那个、base 是预期的，且 merge commit 非空。fetch 目标分支并运行 `git merge-base --is-ancestor "$merge_commit" "$destination_tip"`；然后再检查最终内容，之后才前进。如果服务无法报告相关事实，结果就仍未确认。

服务参考：[GitHub CLI 合并标志](https://cli.github.com/manual/gh_pr_merge)、[GraphQL pull requests 与队列变更](https://docs.github.com/en/graphql/reference/pulls)、[REST 合并端点](https://docs.github.com/en/rest/pulls/pulls#merge-a-pull-request)。当这些内容有出入时，重新核对已安装服务的能力。服务行为要在一次性仓库里测试，绝不用生产 PR 测。
