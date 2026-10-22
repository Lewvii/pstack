### Refactoring

Resolve the driver skill through [poteto-mode's Non-negotiables](../SKILL.md#non-negotiables).

通过 [poteto-mode 的 Non-negotiables](../SKILL.md#non-negotiables) 解析出 driver 技能。

**契约由你负责。结构变，行为不变。** 这与 Feature（增加行为）和 Bug fix（纠正行为）不同。

如果清理过程中发现缺失的功能或真正的缺陷，把它拆出去，先按钉住的契约交付结构改动。重新设计是允许的，但要明确说出来并路由到 Feature。大型或跨切面的结构性工作属于 **figure-it-out** 技能。本 playbook 处理从中型以内、聚焦的改动。

1. 先钉住行为契约。对受影响子系统运行 **how** 技能以了解契约，然后在任何结构移动之前，写一个刻画测试、快照或等价性装置来捕获当前行为。如果该区域没有覆盖，先写这个钉住，再碰结构。类型检查和 lint 不是钉住。
2. 按 **principle-model-the-domain** 指出代码缺失的结构。形状已经清晰且局部时，无聊的代码就留着。重塑必须删掉分支或非法状态，而不是增加间接层。
3. 指出目标形状。说明如果今天从头写，模块布局、类型和调用图应该是什么样（**principle-foundational-thinking**、**principle-redesign-from-first-principles**）。目标跨越函数边界时，在动手前运行 **architect** 技能，对形状做并行设计探索。
4. 先减后加。在引入新形状之前，删死代码、合并只有一个调用者的包装、去掉多余的校验器、清除孤立引用（**principle-subtract-before-you-add**）。达到目标形状的最小改动才发（**principle-laziness-protocol**）。投机性地「也许有用」的清理要撤销。
5. 以小步、保持行为的方式移动，每一步都让钉住保持绿色。API 重塑时，在同一波里迁移所有调用者并删除旧 API（**principle-migrate-callers-then-delete-legacy-apis**）。不要兼容垫片，不要新旧并行的路径。每个重命名都对实际文件抽查一遍。重命名会静默漏掉字符串、散文和反向引用中的用法。用你配置的 refactoring 模型（默认值见 poteto-mode 的 Models 一节）连同明确范围（文件路径、被移动的名字、要保持的行为）把机械性编辑委派给 subagent。按 Subagents 一节的要求，为每个 subagent 选择合适的推理档位。
6. 在真实产物上证明行为未变，而不是「它能编译」（**principle-prove-it-works**）。更大的重塑要跑等价性检查：一个对比新旧输出的脚本、把录制的基线对着新代码回放、或者通过相关 driver 技能在对应观测面上跑一遍冒烟。
7. 确认这个改动值得保留。成功的衡量标准是降低读者的认知负担（**principle-minimize-reader-load**）。如果 diff 没有在某处降低读者负担，撤销它。
8. rebase 成小的有序提交。一个减法提交，然后是重塑，再是后续的清理。按 **sequence-verifiable-units** 原则技能塑形，让每一个保持行为的切片在下一个开始前保持绿色。走一次 **Opening a PR**。

**回复：** 改了哪些结构、用来钉住它的契约、等价性证明、读者负担的变化、交付了什么、撤销了什么。不引入新行为。
