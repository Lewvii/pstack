---
name: no-comments
description: "启动 comment-sicko subagent，修复被采纳的发现，并为声称的约束提供编码方案。"
---

# No comments

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

启动 comment-sicko。对已采纳的发现采取行动。

尊重 comment-sicko 的新鲜视角。

## 范围

使用调用方给定的文件或 diff。否则使用当前相对于 base 分支（默认 `main`）的 diff，包括工作区。

## 步骤

1. 以 `subagent_type: "pstack:comment-sicko"` 启动 `Agent`。传入范围。按照 **poteto-mode** skill 的 Subagents 一节，为工作挑选每个 subagent 的推理级别。不要复述它的规则。
2. 检查它的报告与 diff。拒绝应用代码的改动、范围越界、受异常保护的删除、理由陈述错误的 `MUST KILL`，以及把保留的有意代码当作有罪的标记。关于我们代码中意外之处的标记，保持可处理。不要恢复那些注释。只有当有证据表明保留项涉及我们无法改变的东西时，保留才成立。审计遗漏的限定范围 lint 和 TypeScript 抑制。正确性或安全性方面的抑制，仍作为可处理的 `MUST KILL`。只有在给出确切例外和限定范围证明时，才恢复删除。在接受单薄的 `IMPORTANT` 或 `do not remove` 的 kill 或保留之前，对其符号运行 `/how` 或 `/why`。若某个 kill 含义不明，不要恢复。若某个保留被驳倒或仍含义不明，删除它。回退并重跑一份被拒绝的报告，并指明失败原因。第二次被拒时，报告它未解决，并让 `/no-comments` 失败。
3. 直接修复简单的已采纳标记：删除死路径、去掉参数，或改用真正的 API。如果任何修复需要先定形状，就为已采纳的集合和周边代码运行一次 `/architect`。停在草图。架构负责定形状。步骤 4 负责实现。
4. 在范围内实施最小的根因修复。移除每一个点名的绕过方案。如果根因在范围之外，就落地范围内最小的修复，并将其余部分报告为未解决。the **principle-fix-root-causes** 和 the **principle-redesign-from-first-principles** skills 只指导意图。两者都不授权扩大围栏，也不授权修复围栏之外的实例。绝不附加症状守卫。
5. 约束类注释会说 `do not remove`、`do not change wording` 或 `talk to X before changing`。对涉及我们无法改变之物的保留项，予以保留。提供范围内最省的类型、运行时、测试或 CI lint 方案。等待交互式批准。无人值守和 eval 场景需要调用方预先批准。若获批准，先编码再删除。否则删除，把约束报告为未解决，并草拟范围外的工作。
6. 报告删除数量、恢复的注释、重跑情况、架构草图、修复、编码方案提议、已编码项、未落实的约束以及其他未完成的工作。
