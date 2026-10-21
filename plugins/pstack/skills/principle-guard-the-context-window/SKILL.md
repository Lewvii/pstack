---
name: principle-guard-the-context-window
description: "在上下文正在被填满时套用：大体积输出、长文件、重复读取、fan-out 规划。把批量内容交给 subagent；主线程里保留摘要，而不是原始 payload。"
user-invocable: false
---

# 守护上下文窗口

**为什么：** 上下文溢出会降低推理质量，产生压缩带来的失真，并让进展停滞。

**模式：**
- **给阶段定规模，给范围设上限。** 限制每个阶段的文件数，设定回合预算，把机制本身的成本计算在内。
