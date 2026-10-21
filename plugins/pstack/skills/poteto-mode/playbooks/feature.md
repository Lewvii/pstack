### Feature

**设计由你负责。规划、审查、验证。** 委派实现。留在主导位置。

1. 对受影响子系统跑 `how`。
2. 用 `architect` 做并行设计探索。
3. 把吞吐检查点写成四个 todo 项。确实不适用的维度（单文件、无扇出）保留该项并写 `n/a: <reason>`，不要删掉：
   - **阻塞性的第一步。** 闸门在扇出之前运行。
   - **独立工作流。** 不相交的文件、服务或层可以并行。共享写入要串行。
   - **共享可变状态。** 默认拆分目标（**separate-before-serializing-shared-state** 原则技能）。只有存在真实不变量时才串行。
   - **最小的安全分解。** 如果一个 worker 最好，说明原因。
4. 用你配置的 feature 模型（默认值见 poteto-mode 的 Models 一节）连同明确范围把写代码委派给 subagent（文件路径、按 **principle-model-the-domain** 命名的数据形状及其组织方式：用状态机取代散落的布尔值，用表或注册表取代分支，用带类型的模型取代反复出现的形状假设，这些要在被委派者写逻辑之前选定，还要有成功标准）。按 Subagents 一节的要求，为每个 subagent 选择合适的推理档位。当实现允许多种有效形态时（错误处理、抽象层、测试结构），改用 **arena** 技能委派，让各个 runner 呈现备选方案，由交叉裁判守住选择。这是硬性要求：不接受「说明理由后跳过」，Laziness Protocol 也不能覆盖它（收益在于审查分离，而不在省行数）。被禁止派生 subagent 的委派者，通过自己直接持有 diff 来满足同样的审查分离。不要用「待命中」式的回复去等一个嵌套 agent。**Give every file-writing delegate its own worktree**（用 `isolation: "worktree"` 派生，或交给它一个独占分支），并且不要在委派者仍持有的 worktree 里写文件或跑测试套件。在 Claude Code 上，除非 `worktree.baseRef` 设置为 `"head"`，`isolation: "worktree"` 会从远端默认分支分出。当被委派者要建立在默认分支没有的提交之上时，先提交它们，create its worktree with `git worktree add <path> -b <delegate-branch> HEAD`，并在简报里点名那个 base 提交。在简报文字里圈定文件不是锁（**principle-separate-before-serializing-shared-state**）。注释按 **Comments** 的要求。做外科手术式编辑，对源自上游的文件重新对照源码校正。把共享基础件的改进移植到所有使用方并逐一验证。大方地提交。
5. 在对应的观测面上验证。结论「不确定」或观测面不对都不算通过，要标出来。
6. rebase 成小的、有序的提交。把后续工作堆叠起来。
   运用 **sequence-verifiable-units** 原则技能，每完成一个小单元就构建、验证并提交，再进行下一个。
7. 设计有争议时，在交付前跑 `interrogate`。
8. 走一次 **Opening a PR**。

代码耦合的工作（一个功能、一次迁移）交给单一负责人，检查点内联。该负责人在阻塞阶段之后在内部扇出。父级扇出用于产出独立产物的切片（审计、跨子系统调查、互相竞争的实验）。在阶段边界重写检查点。派生新的负责人，而不是串接中断。

**回复：** 你构建了什么、选择了什么以及为什么、吞吐检查点、未决事项。设计备选方案用表格呈现。
