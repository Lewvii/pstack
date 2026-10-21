---
name: thermo-nuclear-code-quality-review
description: 对抽象质量、巨型文件和面条式条件分支增长进行极其严格的可维护性评审。用于 thermo-nuclear code quality review、thermonuclear review、深度代码质量审计，或格外严厉的可维护性评审。
---

# Thermo-Nuclear Code Quality Review

当需要进行一次异常严格、聚焦于实现质量、可维护性、抽象质量和代码库健康的评审时，使用本技能。

最重要的是，本技能应推动评审者对代码结构保持**雄心**。不要仅仅找出局部的清理机会。主动寻找 "code judo" 式的动作：在保持行为不变的前提下，让实现显著更简单、更小、更直接、更优雅的重构。

## Core Prompt

从下面这段基线提示开始：

> Perform a deep code quality audit of the current branch's changes.
> Rethink how to structure / implement the changes to meaningfully improve code quality without impacting behavior.
> Work to improve abstractions, modularity, reduce Spaghetti code, improve succinctness and legibility.
> Be ambitious, if there is a clear path to improving the implementation that involves restructuring some of the codebase, go for it.
> Be extremely thorough and rigorous. Measure twice, cut once.

## Non-Negotiable Additional Standards

应用上面的基线提示，外加以下明确的评审规则：

0. **对结构性简化保持雄心。**
   - 不要止步于 "这里可以再干净一点"。
   - 寻找机会重新框定这次改动，让整条分支、整个 helper、整个模式、整组条件判断或整层结构彻底消失。
   - 优先选择事后看来让代码显得理所当然的解法。
   - 假设往往存在一个 "code judo" 动作：一次重新组织，更充分地利用现有架构，让改动显著更简单、更优雅。
   - 如果看到一条删除复杂度而非重新摆放复杂度的路径，就大力推动它。

1. **不要让一个 PR 在没有充分理由的情况下，把文件从 1k 行以下推到 1k 行以上。**
   - 默认把这视为强烈的代码质量异味。
   - 优先提取 helper、子组件、模块或局部抽象，而不是让文件膨胀超过 1000 行。
   - 如果 diff 越过了这个阈值，明确询问是否应先分解代码。
   - 只有当存在充分的结构性理由、且结果文件依然组织清晰时，才可以豁免。

2. **不允许现有代码中出现随意的面条式增长。**
   - 对新增的临时条件判断、散落的特例、插入无关流程的一次性分支保持高度怀疑。
   - 如果改动在"随机位置加了奇怪的 if 语句"，把它视为设计问题，而不是风格上的吹毛求疵。
   - 优先把逻辑推入专门的抽象、helper、状态机、策略对象或独立模块，而不是搅乱既有路径。
   - 指出那些让周边代码更难推理的改动，即使它们在技术上是能工作的。

3. **倾向于清理设计，而不只是接受能跑的代码。**
   - 如果行为可以保持不变而结构能显著变干净，就推动更干净的版本。
   - 不要给那些把代码库弄得更乱的"能跑"实现盖橡皮章。
   - 相比只是把同样的复杂度摊到别处的重构，强烈优先能彻底移除活动部件的简化。

4. **优先直接、朴素、可维护的代码，而非取巧或魔法式的代码。**
   - 把脆弱、临时或"魔法"式的行为视为代码质量问题。
   - 对掩盖了简单数据形状假设的通用机制保持怀疑。
   - 标出那些增加了间接层却没有换来清晰度的薄弱抽象、恒等包装和直通 helper。

5. **当类型和边界的整洁性影响可维护性时，大力推动改进。**
   - 当本可以有更清晰的类型边界时，质疑不必要的可选性、`unknown`、`any` 或满是 cast 的代码。
   - 优先显式的类型化模型或共享契约，而非形状松散的临时对象。
   - 如果某个分支靠静默回退来掩盖不明确的不变量，就问这个边界是否应该显式化。

6. **让逻辑待在规范层，并复用现有 helper。**
   - 标出泄漏到共享路径中的功能逻辑，或通过 API 泄漏出去实现细节。
   - 优先使用现有的规范工具或 helper，而不是定制的一次性实现。
   - 把代码推向正确的 package、service 或 module，而不是让架构漂移常态化。

7. **当更干净的结构显而易见时，把不必要的串行编排和非原子更新视为设计异味。**
   - 如果独立的工作被无理由地串行化，就问这个流程是否应该并行。
   - 如果相关的更新可能让状态处于半应用状态，就推动更原子的结构。
   - 不要过度关注微优化，但要标出那些让实现更脆弱的、本可避免的编排复杂度。

## Primary Review Questions

对每一处有意义的改动，问：

- 是否存在一个能让实现显著更简单的 "code judo" 动作？
- 这次改动能否重新框定，从而减少所需的概念、分支或 helper 层？
- 这改善了还是恶化了局部架构？
- 这次 diff 是否在本该有更好抽象的地方增加了分支复杂度？
- 某个原本内聚的模块是否变得更耦合、更富状态、或更难扫读？
- 这段逻辑是否待在了正确的文件和层？
- 这次改动是否把某个文件或组件撑过了健康的规模边界？
- 是否存在重复的条件判断，暗示缺少某个模型或 helper？
- 实现是直接而清晰的，还是依赖特例和附带控制流？
- 这个抽象真的物有所值，还是只是个包装？
- 这次 diff 是否引入了掩盖真实不变量的 cast、可选性或临时对象形状？
- 这段逻辑是否待在规范层，还是 diff 让细节跨边界泄漏了？
- 这段编排是否比必要的更串行、更不原子？

## What to Flag Aggressively

出现以下情况时升级你的发现：

- 一个复杂的实现，而更干净的重新框定本可删掉整类复杂度。
- 只是在搬动代码、却没有减少读者必须放在脑子里的概念数量的重构。
- 因这个 PR 而越过 1000 行的文件，尤其是新代码本可以拆出去的情况。
- 被硬接到无关代码路径上的新条件判断。
- 让现有控制流复杂化的一次性布尔值、可空模式或标志位。
- 泄漏进通用模块的功能专属逻辑。
- 掩盖了简单结构、让代码更难推理的通用"魔法"处理。
- 增加了间接层却什么都没简化的薄弱包装或恒等抽象。
- 模糊了真实契约的不必要 cast、`any`、`unknown` 或可选参数。
- 复制粘贴的逻辑，而不是提取 helper。
- 在已经很繁忙的函数中间实现的狭窄边界情况处理。
- 技术上通过了测试，却让代码更不模块化或更不可读的重构。
- 很可能变成永久债务的"临时"分支。
- 明明代码库已有标准工具，却另写的定制 helper。
- 加在错误层或 package 的逻辑，而它本应存在于更中心的位置。
- 明显独立的工作本可并行、从而更简单清晰的串行异步流程。
- 让状态原子性不必要地变差的部分更新逻辑。

## Preferred Remedies

发现代码质量问题时，优先提出这样的建议：

- 整层删掉间接，而不是打磨它。
- 重新框定状态模型，让条件判断消失，而不是被集中管理。
- 改变归属边界，让该功能成为现有抽象的自然延伸。
- 把特例逻辑变成例外更少的更简单默认流程。
- 提取一个 helper 或纯函数。
- 把大文件拆成更小、更聚焦的模块。
- 把功能专属逻辑移到专门的抽象之后。
- 用类型化模型或显式分派器替换条件链。
- 把编排与业务逻辑分开。
- 把重复分支合并成一条更清晰的流程。
- 删掉没有实际澄清 API 的包装。
- 复用现有标准 helper，而不是引入近乎重复的实现。
- 让类型边界更显式，从而使控制流更简单。
- 把逻辑移到已经拥有该概念的 package、module 或 layer。
- 当并行能同时简化编排时，将独立工作并行化。
- 当部分状态更难推理时，把相关更新重构成更原子的流程。

当真正的问题是结构性的时候，不要满足于 "maybe rename this" 式的反馈。
如果存在一条通往简单得多的想法的可行路径，就不要满足于把同样混乱的想法收拾得干净一点。

## Review Tone

直接、严肃，对质量要求严格。
不要粗鲁，但也不要将重大的可维护性问题弱化成温和建议。
如果代码正在把代码库弄得更乱，就明说。
如果实现错过了性能显著的简化机会，也要明说。

好的说法：

- `this pushes the file past 1k lines. can we decompose this first?`
- `this adds another special-case branch into an already busy flow. can we move this behind its own abstraction?`
- `this works, but it makes the surrounding code more spaghetti. let's keep the behavior and restructure the implementation.`
- `this feels like feature logic leaking into a shared path. can we isolate it?`
- `this abstraction seems unnecessary. can we just keep the direct flow?`
- `why does this need a cast / optional here? can we make the boundary more explicit instead?`
- `this looks like a bespoke helper for something we already have elsewhere. can we reuse the canonical one?`
- `i think there's a code-judo move here that makes this much simpler. can we reframe this so these branches disappear?`
- `this refactor moves complexity around, but doesn't really delete it. is there a way to make the model itself simpler?`

## Output Expectations

按这个顺序排列发现：

1. 结构性的代码质量退化
2. 错过的显著简化或 code-judo 重构机会
3. 面条式蔓延与分支复杂度的增加
4. 让代码更难推理的边界、抽象或类型契约问题
5. 文件规模与分解方面的担忧
6. 模块化与抽象问题
7. 清晰度与可维护性方面的担忧

存在更大的结构性问题时，不要让评审淹没在低价值的细枝末节里。
宁可给出少量高确信度的评论，也不要一长串表面化的注记。

## Report Format

交付物是分层的，绝不是单个大文件：

- 把评审写到它自己的目录（例如 `/tmp/<project>-thermo/`）：一份汇总报告，加上每个子系统或每位 reviewer 一份详细报告（`01_<subsystem>.md`、`02_<subsystem>.md` 等）。
- 汇总报告要能一口气读完，大约 200 行：结论、每条发现写成一段简短的叙述、建议的修复顺序，以及指向承载该发现完整证据的详细文件的指针。
- 详细文件承载深度：测量数据、产生这些数据的命令、验证状态，以及具体写出的 code-judo 提案。
- 用散文段落写发现。点明文件和证据，然后用句子解释问题和补救办法。不要在汇总中把发现压缩成碎片行、标签堆砌或内联命令转储。
- 当评审通过 **swarm** skill 并行分发多个 reviewer 时，本节覆盖 swarm 的汇总规则：各 reviewer 的报告是交付物的详细文件，不是要丢弃的原始输出。把判断汇总进总结；细节留在磁盘上并链接过去。

## Approval Bar

不要仅仅因为行为看起来正确就批准。
批准的标准是：

- 没有明显的结构性退化
- 在存在可行路径时，没有明显错过的、能让实现显著更简单的机会
- 没有缺乏理由的文件规模膨胀
- 没有由特例分支造成的明显面条式增长
- 没有明显取巧或魔法式的抽象让代码更难推理
- 没有让真实设计被掩盖的不必要包装、cast 或可选性变动
- 没有明显的架构边界泄漏，或本可避免的标准 helper 重复
- 没有错过的、能实质性提升可维护性的明显分解机会

除非作者能清楚论证，否则把这些视为推定性的阻塞项：

- PR 保留了大量附带复杂度，而存在一个可行的 code-judo 动作本可将其删除
- PR 把文件从 1000 行以下推到 1000 行以上
- PR 增加了临时分支，让现有流程更加缠绕
- PR 靠在共享代码中散布功能检查来解决局部问题
- PR 增加了不必要的抽象、包装或满是 cast 的契约，让设计更加间接
- PR 在有明确规范归属的情况下重复了现有 helper，或把逻辑放在了错误的层

如果这些条件都不满足，留下明确的、可操作的反馈，并推动更干净的分解。
