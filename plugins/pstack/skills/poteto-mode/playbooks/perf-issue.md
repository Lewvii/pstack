### Perf 问题

**衡量的叙事由你负责。规划、审查、核实数字。** 每个修复都绑定一次测量，不要拿读源码代替测量。

1. 通过对应的 driver 技能抓取基线 trace。基线以及之后每一个数字都用 **benchmark-checklist** 技能把关。
2. 用 `how` 为假设打底。没实际跑过之前，不要断言性能上限。
   按顺序尝试性能口诀，从最便宜的开始：
   1. 不做它。停掉结果没人用的工作，而不是把它变便宜。
   2. 做它，但只做一次。
   3. 少做一点。
   4. 晚点再做。
   5. 趁没人看的时候做。
   6. 并发地做。
   7. 用更便宜的方式做。

   前面的口诀已经达到目标时，就停。
3. 依据 trace 规划修复。跨越函数边界时先叫 `architect`。用你配置的 perf-issue 模型（默认值见 poteto-mode 的 Models 一节）把实现委派给 subagent。按 Subagents 一节的要求，为每个 subagent 选择合适的推理档位。审查 diff。抓取修复后的 trace。
   运用 **sequence-verifiable-units** 原则技能，每次尝试先验证再进行下一次。
4. 解析并比较产物（JSON 导入 sqlite，做 diff）。结论「不确定」或观测面不对都不算通过，要标出来。
5. 在 PR 中引用这次测量。
6. 走一次 **Opening a PR**。

对照某个指标做持续改进（而非一次性修复）时，使用 Hillclimb playbook（`playbooks/hillclimb.md`）。

**回复：** 基线数字、修复后数字、差值、产物路径。
