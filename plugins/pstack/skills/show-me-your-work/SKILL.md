---
name: show-me-your-work
description: "为长时间运行或无人值守的工作留下一份可复核的决策轨迹：一个 TSV 日志，每个决策一行（做了什么、为什么、证据、结果）。默认留在本地；当复核者需要这份轨迹才能信任结果时再提交。用于 /show-me-your-work、自主运行或多阶段运行，或人离开一阵后回来复核的工作。"
---

# Show me your work

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

只保留一份权威日志。

## 格式

一个 TSV 文件，每个决策一行。单元格保持单行。证据是指针，不是散文。

复制 `references/decision-log-template.tsv`（表头行）来开始一份干净的日志。各列如下：

- **ts.** ISO8601 时间戳。
- **phase.** 阶段或工作流。
- **decision.** 选择了什么或做了什么，一行说明。
- **why.** 用平实的话说清原因。如果某条原则驱动了它，就直说，不要当成行话标签。
- **evidence.** 能证明它的链接或路径：commit SHA、PR 编号、`file:line`，或某个产物、trace、截图的路径。绝不要写成一段话。
- **result.** 结果或谓词状态：`tests green`、`reverted`、`pixel-diff 0`、`INCONCLUSIVE`、`open`。

下面是一个例子，用大白话写，让复核者一眼就能读懂。

```
ts	phase	decision	why	evidence	result
2026-05-24T09:02:00Z	frame	counted the work first, about 100 components and roughly 75 hours	wanted to know the size before starting a long run	commit 3a9f1c2	found 5 things to sort out before starting
2026-05-24T09:40:00Z	harness	took screenshots of the old version before changing anything	so we can compare old against new and catch any visual change	scripts/snapshot.sh, baseline/	saved 120 reference screenshots
2026-05-24T11:15:00Z	widget	moved the widget styles over without changing how it looks	keep the change small and the result identical	commit 7c21e0a, pixel-diff 0	looks identical, tests pass
2026-05-24T12:30:00Z	widget	threw out a helper's work because its screenshots were blank	checked the real files instead of trusting its summary	worktree reset	reverted, tightened the instructions for next time
```

## 记录一行

像跟同事讲你做了什么那样写每一条。平实的语言、具体的动作，不用 AI 腔或抽象行话（**unslop** skill 同样适用于日志文本）。

使用辅助脚本 `scripts/log.sh <logfile> <phase> <decision> <why> <evidence> <result>`。它会打上 `ts` 时间戳，首次使用时写入表头，去掉多余的制表符和换行，并把以 `=`, `+`, `-`, `@`, 或 `"` 开头的单元格加上单引号前缀。直接用 `printf` 追加一行也可以，但如果单元格内容来自生成文本或用户提供的文本，就要注意这些同样的字符。

记录决策点和检查点，而不是每个动作：选择了一条分叉、完成了一个单元并附其验证结果、一次转向或回退并附其触发原因、浮现了一个阻塞、修好了一道关卡。循环运行中，每次迭代一行。琐碎和显然的事跳过。

一次运行就是一个 agent 对话，包括它后续的轮次以及对其的任何总结。接手、替换的 agent 或新的聊天都开始一次新的运行。当一次运行向已有行的日志追加时，它的第一行 phase 为 `start`；在另一次运行的 `start` 行之后的它的第一行也是如此。因此，若某次运行在后续轮次回到某份日志，它先读日志的最后几行，看这期间是否有别的运行写过。`start` 行会标明它之前那些本运行未写入的行的 `ts` 范围，其证据指向本次运行，例如 agent id。`start` 这个 phase 不用作其他用途。

## 日志放在哪

默认情况下，日志是工作产物，不提交。放在工作目录的 `decisions.tsv`，或多个工作同时进行时的 `.audit/<task-slug>.tsv`，并把它排除在 git 之外。

只有当工作够大、复核者需要这份轨迹才能信任结果时，才提交它。

## 规则

- 只追加。错误的判断用新的一行取代它。绝不编辑或删除历史。
- 优先使用由已提交脚本产出的证据，而不是手工的一次性操作（**encode-lessons-in-structure** principle skill）。

## 对照 transcript 审计日志

运行结束时，在交付之前，检查日志说的是真话。读取本次运行在 Claude Code 各项目 transcript 目录下的记录，即 `~/.claude/projects/<encoded-cwd>/` 中的本次运行记录。不要对 `~/.claude/projects/` 做通配。那会读到无关的私人聊天。把本次运行的各行与实际发生的事逐一核对。这些行的每一段都始于本次运行的某个 `start` 行（若日志由本次运行创建，则始于第一行），止于下一次运行的 `start` 行：

- 检查每一行都对应一个真实决策或动作。
- 检查每一行的证据都能解析，并显示出该行所声称的内容。
- 一个影响了工作却没有被记录的分叉、转向或放弃的方案，是一个缺口。把它补上。

修正日志，而不是改故事。审计绝不编辑或删除任何一行，哪怕是编造的。当某行既没记录真实决策也没记录真实动作，或其声称或证据有误时，补一行取代它，写明实际发生了什么以及一个能解析的指针。本次审计不检查本运行各段之外的行。如果本次运行自己的工作发现其中某行有误，就照任何错误判断一样取代它。

## 对轨迹做跨模型复核

交付之前，在一个与完成该工作的模型家族不同的模型上启动一个 subagent。自我复核不能替代它。该 subagent 阅读审计轨迹和本次运行的 transcript，然后标出用户应当留意的点。这不是重做工作，而是扫描次优或有风险之处。按照 **poteto-mode** skill 的 Subagents 一节，为工作挑选每个 subagent 的推理级别。

- 证据薄弱或缺失的决策记录。
- 被跳过、或在 transcript 中未获证明就被声称完成的验证步骤。
- 事后看来有风险的选择（过早、范围蔓延、掩盖症状）。
- 用户随手扫一眼就会错过的缺口。

凡是产出了轨迹的运行，其每条回复都以 "Attention" 一节结尾。先用 reviewer 的模型单独成行（`reviewed by <model>`），然后列出每个标记，指向具体的行或时刻。"No flags" 是合法取值。模型名则不是。

## 复核轨迹

从上到下读，沿证据指针查看，做抽查。GitHub 会把已提交的 TSV 渲染成表格。`column -s$'\t' -t decisions.tsv` 可在终端中渲染。

## 组合使用本技能

其他技能把审计轨迹交给这里处理，而不是自己另造一份。按名称引用它，让本技能掌管格式。不要重述各列。
