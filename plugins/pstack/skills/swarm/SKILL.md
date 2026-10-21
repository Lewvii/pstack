---
name: swarm
description: "分发 N 个并行 worker，收敛它们，返回一份报告。用于 /swarm、'swarm this'，或并行覆盖、竞赛、车轮战和探索。"
---

# Swarm

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

## 开始

在启动任何东西之前，先开一个 todolist，每个阶段一条。

1. 框定
2. 分发
3. 汇总
4. 报告

## 阶段 A：框定

1. 说明完成谓词，以及 swarm 必须返回的产物或报告。
2. 选择形态。切分成若干片、让 N 个 worker 在相同 brief 上竞赛，或两者混合。对于竞赛或混合形态，在 spawn 之前声明 `first pass`、`rank all` 或 `best-of`。
4. 从 `pstack-models.md` 的 `swarm workers` 行选择 worker 模型。如果覆盖表或该行缺失，使用 [Models](#models) 中的默认值。对于 `auto` 或 `inherit-parent`，省略 `model`，让 worker 运行在父模型上。如果 `Agent` 工具拒绝某个 slug，使用默认值并说明。如果它连默认值也拒绝，就从错误信息中取同家族最近的合法 slug。模型竞赛时，提前为每一方命名模型。按照 **poteto-mode** skill 的 Subagents 一节，为工作挑选每个 subagent 的推理级别。
5. worker 需要写入时，给每个 worker 各自可写的输出。当 worker 要验证或测量 commit 时，每份 brief 都要写明确切的 SHA。测量类 brief 还要写明方法（样本数、一个样本是什么、顺序）。worker 把这两项都记录在结果里。

## 阶段 B：分发

在一条消息中 spawn 全部 N 个 worker，使用 `subagent_type: "general-purpose"`、`run_in_background: true`，以及步骤 4 的模型（`auto` 或 `inherit-parent` 时留空）。Claude Code 的所有 subagent 都运行在本机，所以隔离来自阶段 A 分配的 worktree 或输出目录，而不是远程环境。

当 worker 必须从非默认分支开始时，在它自己的 worktree 中检出该分支，并在其 brief 中写明 worktree 路径。

每份 brief 都要自成一体。写明目标、范围、确切的切片或竞赛方、如何验证、以及要报告什么。报告使用 `PASS`、`ISSUES` 或 `BLOCKED` 并附证据。能证明缺陷的 worker 报告 `ISSUES`，并列出它能证明的每个问题，而不只是第一个。

如果有 worker 掉线，用 N-1 继续并注明。

## 阶段 C：汇总

读取各终端结果。丢弃未记录其 brief 所述 SHA 和方法的结果，并让该 worker 重跑一次。第二次仍缺失时，记为缺口。缺口不算通过。覆盖形态下，每个必需的切片都需要结果。竞赛形态下，应用事先声明的选择规则。使用 first pass、rank all 或 best-of。不要粘贴 worker 的原始输出。

## 阶段 D：报告

返回一份统一的聊天内报告，包含表格、每个问题的一行说明、缺口或掉线情况，以及使用时的竞赛规则。

## Models

Role defaults, stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in the `pstack-models.md` override sheet overrides each at runtime; `/setup-pstack` writes it and lists its path per runtime.

- swarm workers: `opus`

## Reasoning effort

A role value in the override sheet may name a reasoning effort after its model, as in `opus @xhigh`. Levels on Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Which ones apply depends on the model. A value without `@` takes the sheet's `default effort` line, a level or `session`, and `session` when the sheet has no such line. `session` sets no effort, so the dispatch is the usual one. Strip the suffix before reading the model: `inherit-parent` or `auto` still omits `model` at every level, and a model name is passed as `model`. On Claude Code, a level picks the effort agent from the `subagent_type` you would otherwise use. `pstack:poteto-agent` becomes `subagent_type: "pstack:poteto-agent-<level>"`. `general-purpose`, or no `subagent_type`, becomes `subagent_type: "pstack:effort-<level>"`. The effort agents set only `effort`, so the model you pass still decides the model. On Codex, pass the level as `spawn_agent`'s `reasoning_effort` and keep the usual instructions.
