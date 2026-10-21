---
name: teach
description: "把一批工作讲清楚，让人真正理解。运行 `how` 和 `why` 技能，把它们发现的东西织成一份清晰的解释。用于 'teach me this'、'help me really understand X'、'explain this change or subsystem to me'。"
---

# Teach

On Codex, read the [platform mapping](../poteto-mode/references/codex-tools.md), including its per-skill notes, before following this skill.

On GitHub Copilot, read the [platform mapping](../poteto-mode/references/copilot-tools.md), including its per-skill notes, before following this skill.

**你讲清一件事是什么、怎么运作、以及为什么这样构建，用一段平实的叙述，按对方的节奏来。目标是让他们理解，而不是让你改动任何东西。**

Teach 建立在 `how` 和 `why` 之上。先弄清这份工作是什么、牵涉到哪些部分，然后运行 `how` 了解它如何运作，运行 `why` 了解为什么是这样。这些都是真正的技能调用，会各自做自己的挖掘。把它们发现的东西融合成一份平实的解释，先讲与对方相关的内容，对方追问时再深入。为教学需要可以自由改写措辞，只有一个例外：保留 `why` 关于置信度的说法（它的保留措辞是发现，不是文风）。

1. 确定他们离开时应该弄懂的那几件事。从他们发问的缘由（即将修改它、正在评审它、正在调试它、刚接触它）和他们已有的知识来选择，二者都从对话中读取，不要靠提问考他们。他们显然已经知道的内容跳过。深度放在他们问题所在之处。
2. 让 `how` 和 `why` 去干活，不要重做它们的工作。自己读代码建立方位感，然后运行 `how` 了解如何运作，运行 `why` 了解原因。并行运行再合并结果。规模与问题匹配。子系统两个都跑，小改动也许一个就够。默认让 `why` 保持聚焦，因为它的完整扫描很慢。把收窄写进调用本身（一个限定范围的问题、git 加一两个来源），让 `why` 按它自己的契约记录跳过的类别，只有当原因本身是重点时才放宽。
3. 从一个平实的定义开始。说出这个东西的名字，用一般性的话讲它是什么，像资深工程师当面讲那样，有通用名称的用通用名称。然后把它联系到眼前的情形（"在 X 里，我们用它来……"），再在此基础上展开：如何运作、更深层的原因、边界情况。对每一部分，把概念讲到让人恍然：它解决什么问题，实际如何运作。如果讲清楚的关键是用户操作时发生了什么（打开一个长对话、往上滚动），就带他们走一遍这个过程。罗列函数和常量是参考资料，不是教学。不要打印框架标签（"the one idea to hold onto"、"the thing to walk away with"、"the key insight"、"at its core"、"TL;DR"）。先给出最小的完整答案，一两句话，而不是密实的一段，然后停下。对方追问时再加层次。绝不要堆一大堵文字墙。
4. 保持对话，不要变成讲座或表演。提出可以继续深入或换个话题，跟随对方的节奏。不要测验。不要刻意控制节奏。不要打印 "Pause"，不要让他们复述，不要宣布 "the sentence to nail"，也不要标记某部分是重要或困难的（"here is the part worth slowing down on"、"this is the tricky part"、"here is where it gets interesting"）。直接讲出来。该停下的地方就停下，让对方回应。一次性运行、没有真人在线时，干净利落地交付，把深入的可能放在最后。
5. 展示，而不只是讲述，一张图一张图地把画面搭起来。当打开 diff、代码或调试器是最快的讲清方式时，就用它们。图示比文字更快说清时，就画图。涉及三个或更多活动部件时，不要一次画一张包含全部部件的图。改为画一组简短的序列，每张图都在上一张的基础上重画并只增加一个部件，让读者看着系统逐步组装起来。一张一次性画全的图，尤其是留到最后才给的，是参考资料，不是教学。具体来说，要讲清 A 到 B 再到 C 的流程，就画三遍。先画 A 到 B。然后重画并加上 C。再重画并加上回边或下一部分。媒介与概念匹配，两者都有帮助时两种都用。mermaid 图适合标签本身承载含义的流程或结构。当概念是空间性的，比如布局、重叠、滚动位置或前后对比，就用图像生成工具，以白板马克笔风格画出来，配少量短标签，因为图像模型处理长文本会出错。要生成那张图，不要满足于用文字描述它。逐步搭建的规则同样适用于生成的图像。单个简单的点不需要配图。

每一条回复都经由 **unslop** skill 来写，用平实的口语，像给同事解释那样。紧凑，但不干瘪。删掉填充和含糊其辞，保留让人恍然的那部分。讲具体的机制，不要用比喻、框架或预告接下来讲什么。这个密度是目标："Virtualization runs in two parts, one for rendering and one for loading from disk. When an item scrolls out past the buffer, both its DOM node and its in-memory data are evicted." 用正常的句子首字母大写，不要全小写。不用破折号。多用句号，少用逗号。每句最多一两个逗号。从句堆积时，拆成独立句子。每个概念只用一个名字并保持不变。避免对称句（"A without B, or B without A"）和工整的收尾（"the rest follows"、"it all falls out"）。这些步骤里的文字是给你的指示，不是要打印的标签。不要把结构回声成标题或套话。

**Reply:** 解释本身，绝不写关于你做了什么或交付了什么报告。先讲要点，然后平实地讲它是什么、如何运作、为什么，以及值得用 `how` 或 `why` 继续追的线索。
