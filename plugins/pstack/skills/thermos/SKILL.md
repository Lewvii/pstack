---
name: thermos
description: "并行启动两个 thermo-nuclear 评审 subagent，然后综合它们的发现。用于 thermos、double thermo review，或合并的 bug/安全与代码质量分支审计。"
---

# Thermos

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

把两轮 thermo 评审作为异步后台 subagent 并行运行，然后综合它们的结果。

## 工作流

1. 从用户请求、PR、当前分支或相关改动文件中确定评审范围。
2. 收集 diff 以及 reviewer 评估改动所需的任何文件或上下文摘录，使他们无需猜测。
3. 在同一条消息中启动两个 subagent，均带 `run_in_background: true`：
   - `subagent_type: "pstack:thermo-nuclear-review-subagent"`，负责 bug、破坏性变更、安全、开发者体验退化、功能开关泄漏以及其他分支审计风险。
   - `subagent_type: "pstack:thermo-nuclear-code-quality-review-subagent"`，负责可维护性、结构、文件规模增长、面条式代码、抽象以及代码库健康风险。
4. 给每个 subagent 传入相同的限定范围 diff 和文件上下文，并要求它返回按优先级排序的发现，附带文件引用和证据。
5. 两者都完成后，综合结果，把发现放在前面，并在 reviewer 之间去重。对重叠的发现加重权重，用你自己的判断解决分歧，保持总结简短。

如果各个后台总结已经对用户可见，不要整段复述它们。给出统一的结论、信号最强的发现，以及仍然存在的不确定性。
