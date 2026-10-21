---
name: babysit
description: 盯住一个已开启的 PR：修复失败的 CI、处理直截了当的评审评论，并把它推进到可合并状态。Cursor 内置 /babysit 的 Claude Code 对应实现。在开完 PR 之后使用，此时用户希望 agent 无需重复提示就能照看它。
---

# Babysit a PR

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

这是 Cursor 内置 `/babysit` 的 Claude Code 对应实现。具体实现是对 `gh` CLI 的循环调用，加上用 Claude Code 的 `loop` skill 控制节奏。

在 poteto-mode 内部，**Babysit** playbook（[`../poteto-mode/playbooks/babysit.md`](../poteto-mode/playbooks/babysit.md)）取代本技能：它掌管模式声明、合并前沿、栈安全和 `watch-pr` 监视器。在 poteto-mode 运行之外处理单个 PR 时，本技能仍是独立的 `/babysit` 入口。

## 何时使用

- 有一个已开启的 PR，用户明确希望它保持绿色，而且你尚不在 poteto-mode 运行之中（那种情况由 playbook 管）。
- 用户直接调用 `/babysit`。
- 开 PR 的 subagent 不负责 babysit，应回到父级由父级决定。

## 步骤

1. **获取 PR 状态。**

   ```bash
   gh pr view <number> --json number,title,state,mergeable,reviewDecision,statusCheckRollup,mergeStateStatus,comments,reviews
   ```

2. **按优先级分诊。**
   - 合并冲突（`mergeStateStatus == DIRTY`）：运行 the **fix-merge-conflicts** skill。只有当分支属于你且未共享时才 force-push。
   - 失败的检查（`statusCheckRollup` 中 `conclusion: FAILURE` 的条目）：运行 the **fix-ci** skill。对失败做根因分析；修复底层代码或测试；commit；push。
   - 评审评论：运行 the **get-pr-comments** skill 获取总结，然后只对真正认同的反馈采取行动。当评论只有唯一的机械性答案时（重命名、守卫子句、格式细枝末节），做修改并在 commit message 中引用该评论。当它取决于判断，或你看不出对方在问什么时，不要猜：留着它，并回复你本会怎么做。
   - 评审机器人评论（Bugbot 及类似自动化）：行动前先按 [`bugbot-triage.md`](../poteto-mode/references/bugbot-triage.md) 分类为 fix/dismiss/ask。对安全、数据和高度严重性问题，默认先询问。

3. **循环。** 用 Claude Code 的 `loop` skill 控制复查节奏。从你监视的对象选择间隔：
   - CI 正在运行：轮询 `gh pr checks --watch`（它会阻塞到检查结束，因此无需单独的循环间隔）。
   - 等待 reviewer：20 到 30 分钟的心跳。
   - 空闲但想抓新评论：每小时一次。

4. **何时停止。**
   - 构建绿色、所有评论已解决、分支可干净合并，就可以宣布就绪。
   - 你已跑过三轮 fix、push、复查，仍未完全变绿，就停止，总结还有什么坏了，并把控制权交回。
   - 下一个修复会迫使做出设计选择，就暂停，用 `AskUserQuestion` 交给用户。

5. **报告。** 总结已应用的修复、已处理的评论、推迟的评论（附原因）、当前 PR 状态。每个 commit 都引用其 SHA。

## 硬性规则

- 不要重写别人可能已经拉取过的分支的历史。如果看起来需要 rebase 或 force-push，先跟用户确认。
- 不要为了让测试通过而改动其期望值。只有当行为确实变了、且断言固定的是旧行为时，才修改断言。
- 绝不跳过 hook（`--no-verify`）。
- 绝不通过把失败的检查标为非必需来绕过它。
- 只有当所有检查变绿且没有未解决的评审评论时，才运行 `gh pr ready`。

## 交叉引用

- 开 PR 不会启动 babysit；在 poteto-mode 内部，Babysit playbook 掌管该请求，且只在被要求时才启动。
- 如果 diff 有争议，在开 PR 之前用 `interrogate`；一旦开启，就由 babysit 接管。
- 你在这里写的任何散文（PR 评论、commit message、状态报告）都要用 `unslop`。

## 来源

这是 Cursor `/babysit` 的 Claude Code 对应实现，不是移植，因为 Cursor 的实现是闭源的。本技能独立编写，有自己的文字和结构；工作流参考了 Cursor 公开的 `/babysit` 行为。与其他 PR 工具唯一的重叠是它运行的 `gh` CLI 命令，这些是功能调用而非抄录的文本。
