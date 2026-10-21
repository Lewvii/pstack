---
name: technical-writing
description: "分层技术写作规范：Diátaxis 结构、Google 开发者文风句子、STE 指令规则、Global English 句法。用于 /technical-writing，或在编写、评审文档、RFC、readme、PR 描述、提交信息时使用。"
---

# 技术写作

写到疲惫的工程师第一遍就能看懂。四层规则全部适用。

三层之上还有三条凌驾规则：

- **砍掉每个不起作用的词。** 一句话去掉某个词仍然成立，那个词就该去掉。「In order to」就是「to」。「It is important to note that」什么也不是。
- **用短小的日常词。** 用「use」，不用「utilize」；用「help」，不用「facilitate」；用「do」，不用「perform」。长词必须用精确性来证明自己的长度。
- **规则让句子变差时，换个办法改，或者别动它。** 一句话遵守了所有规则、读起来却像机器写的，那就是失败。

代码库就是词表。写真实的符号、文件、开关或命令名，不要写同义词，也不要写对它的描述。

不要生造行话。用开发者会开口说出来的词：「move」「delete」「a budget that only decreases」，而不是「evacuate」「ratchet」或「endgame」。文档第一次出现时就解释清楚含义的具名模式可以用。发现新的违例词及其替换词，连同 diff 一并写进你的回答，作为对 `unslop` 抽象隐喻规则的补充。不要直接改那个技能。

## 变化节奏

一份文档可以遵守每一层规则，读起来仍然像机器写的：每句都切得很短，处处没有观点，什么都没说具体。

- 有意混合句子长度。短句落下要点。长句承载一个事实及其条件或后果。
- 一句话一个想法，不等于每句都一个长度。把承载两个想法的句子拆开。承载一个想法的长句保留。
- 文体允许时要有观点。Explanation 本就在权衡量，所以说出你怎么看，而不是罗列利弊。Reference 保持干燥。
- 具体胜过干瘪。不写「schema changes can cause issues」，写「a column rename fails the build」。

## 先选文体（Diátaxis）

一份文档，一种文体。两个问题定它：内容是在指导行动（做）还是增进理解（想），以及它服务于学习还是工作？

- 行动 + 学习：**tutorial**（教程）。
- 行动 + 工作：**how-to**（操作指南）。
- 理解 + 工作：**reference**（参考）。
- 理解 + 学习：**explanation**（解释）。

这套罗盘可以用在整份文档上，也可以用在一句话上。

**Tutorial：边做边学。** 学习者能否成功是你的责任，不是他们的。开头说学习者将做出来什么，而不是他们将「学到」什么。每一步都产出看得见的结果，宜早宜频。告诉他们应该看到什么：预期输出、提示符的变化、日志行。把解释压到一个分句加一个链接。保持具体。用「我们」的口吻、以命令形式写：「First, do x. Now, do y.」

**How-to：通往目标的步骤。** 解决人面临的问题，而不是机器能执行的操作。假定读者有能力，跳过教学。只讲行动：不跑题、不铺垫背景、不为完整而完整，那些用链接给出。允许分叉和判断：「If you want x, do y.」用任务给指南命名：「How to calibrate the radar array」，而不是「Radar array calibration」。

**Reference：供查阅的事实。** 只做描述。不指导、不说服、不表态。干燥、完整、确定。陈述事实、选项、限制和错误，不加模糊限定。镜像被描述对象的结构，让代码和文档可以对照着走。把材料放在读者预期的地方。尽可能从代码生成。

**Explanation：理解与为什么。** 一个边界清晰的主题，可以脱离产品单独阅读。每个标题前都应能容忍一个隐含的「About...」。锚定一个真实的「为什么」问题。给出上下文：设计决策、历史、约束、备选方案。只有这里允许表达观点，别处不行。

不要混文体：tutorial 里不要放 reference 表格，reference 里不要有 tutorial 式的搀扶，how-to 里不要论证。拆分，然后用链接连接。

## 面向读者写句子（Google 开发者文风）

- 用「你」称呼读者，用现在时。「will」只用于确实会在之后发生的事。
- 说清谁做什么：「the compiler checks」，不是「is checked」。只有在施动者未知或无关紧要时才用被动。
- 指令写成命令句：「Click Submit.」事实直说。绝不用「should be done」。
- 条件放在指令之前：「To delete the document, click Delete.」
- 常见情形放前面。例外放后面。
- 像一位懂行的朋友那样说话。不用流行词，不用比喻，指令里不加「please」，流程里绝不用「simply」「easy」「quickly」。真要是简单，读者就不会读到这了。
- 不要预告（「we will soon support...」），也不要连续几句用同一个开头。
- 链接文字要说明链接去哪：页面标题或简短描述。绝不用「click here」。宁可本页多写一句上下文，也不要链出去。
- 标题承载要点，而不只是主题（「先选文体」，而不是「文体」）。用句子大小写。任务类标题是不带宾语的动词短语（「Create an instance」）。概念类标题是名词短语。每页一个 h1，不跳级。
- 序列用有序列表，其余用无序列表。用一句完整的话引出列表。各项保持平行。
- 代码用代码字体。UI 元素用加粗。用序列逗号（serial comma）。丢掉「etc.」，事先说明列表并不完整。

## 让陈述一次只承载一件事（STE 规则）

- 一句一条指令。其他场合一句一个想法。
- 超过约 20 个词的指令，以及其他超过约 25 个词的句子，拆开。
- 把警告或条件放在它所守护的步骤之前：「If hot oil touches your skin, injuries can occur.」
- 保留「the」和「a」：「Remove backup file」有两种读法，「Remove the backup file」只有一种。
- 每个词一个含义、一个职责，然后保持一致。如果「check」表示检查，就不要同时用它表示约束。
- 每个动作只用一个词并坚持：「start」，不要这里用「start」那里用「initiate」。
- 流程写成直接命令句，绝不写成叙述，也绝不用被动：「Install the component」，不是「the component must be installed」。
- 能避免就避免「-ing」形式的词。

## 不留有两种读法的句子（Global English）

- 让「only」「not」这类词紧挨它修饰的词：「only fails on growth」和「fails only on growth」含义不同。
- 拆开过长的名词串：「the proto import budget check script」改成「the script that checks the proto-import budget」。
- 让每个「it」「they」「this」都明确指向一个显而易见的东西。拿不准就重复名词。绝不用「this」或「which」指代整个从句。
- 不要省略动词：「Phase 1 moves the converters and Phase 2 the runtime」让 Phase 2 少了动词。补上。
- 保留标示结构的小词，比如「Ensure that the switch is off」里的「that」。绝不为省字数牺牲清晰。
- 并列时重复冠词能避免误读就重复：「the client and the host」，当它们本是两个东西时，不要写成「the client and host」。
- 一句话可以有两种分组方式时，说明「and」或「or」连接的是哪部分。「Both...and」「either...or」「if...then」都是免费消歧工具。
- 用句号，不用分号。长破折号换成新的一句。
- 括号里的文字要么是完整的语法单位，要么自成一句。绝不用「(s)」构成复数。
- 不用斜杠：写「a, b, or both」，而不是「a/b」或「and/or」。
- 同一个东西在任何地方都只用一个名字。一份文档里把一个东西叫成「the gate」「the ratchet」「the budget check」，等于教了三样东西。没变的东西不要翻来覆去地改。
- 不用习语、口语、拉丁缩写和隐喻。

## 语气与仓库相关事项

- 对本文技能触及的每一份文档都应用 the **unslop** skill。那个技能负责 slop 模式目录：AI 词汇、填充、模糊限定、格式破绽。
- PR 描述和提交信息也是写作。除 Diátaxis 之外的每一层都适用于它们。PR 正文是一份评审者能在一分钟内读完的简报。不要粘贴 swarm 日志、SHA 列表或指标表格。给出链接。
- 产品 UI 文案不是文档。那些用你产品的文案规范。
- 代码片段用制表符缩进。写真实路径。每个计数或目录树断言，在落地它的那个提交上都必须为真，并附上可重新生成的命令。

## 示例

改写前：

> Configuration of the proto import ratchet budget script parameters is performed via budget.json. Note that it's important to remember that running with --write, which updates the committed budget to reflect the current count, should only be done when lowering it. If exceeded, CI fails.

改写后：

> `budget.mjs` reads the committed budget from `budget.json` and counts the files that import protos. If the count exceeds the budget, CI fails. Run `budget.mjs --write` only to lower the budget.
