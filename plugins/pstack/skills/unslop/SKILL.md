---
name: unslop
description: 从任何文字中剔除 AI 腔。
---

# Unslop

编辑文字，去掉 AI 腔。逐条扫描下面的模式，然后改写。保留原意，贴合原本的语气。

## 需要识别并修正的模式

规则编号是固定的 id，其他技能会引用它们。删除某条规则会留下空号。

### 内容

3. **空洞的 -ing 短语。** 「highlighting...」「ensuring...」「reflecting...」「showcasing...」「fostering...」这类。删掉，或用真实来源展开。
5. **含糊的归因。** 「Experts believe」「Industry reports suggest」「Some critics argue」。点明来源，或删掉。

### 语言

7. **AI 词汇。** Additionally、crucial、delve、enduring、enhance、fostering、garner、interplay、intricate、landscape（抽象用法）、pivotal、showcase、tapestry（抽象用法）、testament、underscore、vibrant。换成朴素的说法。
8. **把「是」说得花哨。** 「serves as」「stands as」「boasts」「features」。直接说「是」或「有」。
9. **「不只是 X，更是 Y」。** 直接讲要点。
10. **三段式排比。** 不要硬把想法凑成三个一组。该几个就几个。
11. **同义词轮换。** 一段话里 protagonist、main character、central figure、hero 轮着用。选一个，重复它。
12. **虚假区间。** 「从 X 到 Y」而 X 和 Y 并不在同一有意义的尺度上。直接列出各项。

### 风格

13. **滥用破折号。** 完全避免使用长破折号。只用句号或逗号（不要用括号，不要用短破折号，也不要用连字符代替破折号）。要分隔想法就断句，或用逗号。
14. **滥用冒号。** 列表或举例之前用冒号没问题，但不能拿来连接句子。像「如果你来自传统自动化：不要注册事件处理器，而是描述条件」，这个冒号什么也没加上。改写成让要点自己立住，不靠对比框架。「描述调度器何时应当触发，用大白话讲最好」。意思一样，不靠标点当拐杖。
15. **滥用加粗。** 不要给每个专有名词或缩写加粗。
16. **内联标题式列表。** 破绽是用冒号把该行内容重复一遍的加粗标签：「**Performance:** Performance improved...」。这类改成散文。以句号结尾、点出条目、后面跟真正新信息的加粗引导语（「**Schema in TypeScript.** Tables live in one file.」）是可以的，不算破绽。
17. **标题用标题大小写。** 用句子大小写。
18. **装饰性 emoji。** 从标题和列表项里去掉。
19. **弯引号。** 用直引号。

### 交流痕迹

20. **聊天机器人套话。** 「I hope this helps!」「Let me know if...」「Of course!」「Certainly!」「Found the smoking gun!」删掉。
22. **谄媚语气。** 「Great question! You're absolutely right!」直接回应。

### 填充

23. **填充短语。** 「In order to」改成「To」。「Due to the fact that」改成「Because」。删掉「It is important to note that」。
24. **过度模糊限定。** 「could potentially possibly be argued that it might」改成「may」。
25. **空泛的结尾。** 「The future looks bright.」给出具体计划或事实。

### 行话

26. **抽象隐喻名词。** Substrate（base）、wedge、wedge in（add）、vector（way、method）、locus、vantage、nexus、作名词用的 primitive、作隐喻用的 harness、如「API surface」中的 surface、bedrock、作隐喻用的 scaffolding、modality、paradigm、gold-plating（超出任务所需）、作隐喻用的 ratchet（说的是机制的真名，或只紧不松的限值）、指移动代码的 evacuate（move out）、endgame（最后阶段）、north star、flywheel。这些看起来专业，但通常有更朴素具体的词。挑具体的那个。
27. **讲它做什么，不讲它感觉如何。** 「the database stays close at hand」「SQL you can read」「types that follow your schema」说的是感觉。修正方式是点明机制或给出数字：「`.toSQL()` returns the exact string sent to the database」「a column rename fails the build」。问自己这句话告诉读者去做什么、知道什么，然后写那个。如果你没法把它重述成一条具体指令、事实或数字，就删掉。再检查一次：如果这句话原封不动出现在别的项目的文档里也成立，那它对当前项目什么都没说。删掉。
28. **压缩或拆分过密的句子。** 如果读者要回头重读才能解析一句，就拆成两句或砍掉从句。一句话一个想法。
29. **主动语态。** 优先使用。抓到「is/are/was/were + 过去分词」就点明施动者：「queries are validated」改成「the compiler validates queries」，「the file is parsed by the loader」改成「the loader parses the file」。只有在施动者未知或确实无关时才用被动。
30. **砍掉副词，或换更强的动词。** 「runs quickly」改成「is fast」或直接给数字，「significantly improves」改成实测的差值。用副词撑着弱动词，说明该换动词了。
31. **优先用朴素词。** 「utilize」和「leverage」改成「use」，「facilitate」改成「help」，「numerous」改成「many」，「in the event that」改成「if」。花哨的同义词很少更清楚。
32. **做作的文风。** 有直白说法却用隐喻或修饰：格言（「wire it or delete it」）、为效果而用的反问碎片、让代码拟人（「the plan holds it」）、比喻性的动词（「rides along」「stands on」）、套话框架。「A dial worth turning」改成「a parameter worth varying」。怎么想就怎么写。第 26 条管的是隐喻名词。
33. **过度压缩。** 省略冠词、无动词的碎片、符号化写法，以及让读者需要解码而不是阅读的缩写。「Parser rejects bad date → exit 2, no write」改成「The parser rejects a bad date, exits with code 2, and writes nothing.」写出带冠词和动词的完整句子，把箭头和缩写都展开。
