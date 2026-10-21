# Linear 工单

## 这个来源包含什么

- 描述功能、缺陷及其动机的 issue
- 附在 issue 上的项目文档（通常是 PRD 或规格说明）
- 父 issue / 子 issue 关系（更宽泛的计划 → 具体工单）
- issue 上的评论（澄清、范围变更、「我们为什么做这个」的理由）
- 标签（例如 `compliance`、`customer-request`、`perf`），提示动机的类型
- 解释范围变更的状态更新
- 附件和关联的 GitHub PR

产品/业务背景往往就在 Linear 里：也就是「我们这么做是因为客户 X 提出了要求」或「这是为了 Q3 合规计划」那一层。

## 如何搜索

使用 Linear MCP。

1. **从关联工单开始。** 如果种子提交或 PR 引用了工单 ID（例如 `ENG-1234`、`[BUG-567]`），先用 `get_issue` 拉取它们。完整阅读 issue，包括评论。
2. **按关键词列出相关 issue。** 用 `list_issues` 的文本搜索查功能名、关键符号或业务术语。尝试多种说法。
3. **遍历 issue 树。** 如果你落在一个子 issue 上，拉取它的父 issue。子 issue 偏战术层面。父 issue 往往承载着「为什么」。
4. **阅读项目文档。** 如果该 issue 属于某个项目，用 `get_project` 查看附带的文档。项目级文档是最常记录规格和理由的地方。
5. **检查标签和里程碑。** 标签提示动机的类别（customer-request、incident-followup、compliance）。里程碑把工作与截止日期绑定，而截止日期常常揭示动机。

## 好的证据在这里长什么样

- 陈述业务问题的 issue 描述："Customer Acme needs X because of their SOC2 audit"
- 记录了决定的评论："We decided to go with approach B because approach A would require touching the billing service"
- 标题像一项计划的父 issue："Q3 Enterprise Readiness" 或 "Reduce Payment Failures"
- 附带的 PRD 或规格说明
- 诸如 `customer:acme`、`incident-followup`、`compliance`、`perf-regression` 的标签

## 常见陷阱

- **范围漂移。** PR 引用的工单可能被关闭后又以不同的范围重新打开。阅读完整历史。
- **机械化模板。** 有些团队要求写 "Why" 小节，却用套话填充。泛泛的文字（"improve user experience"）大概不是真正的答案。
- **过时的工单。** 旧工单往往反映的是后来改变了的计划版本。检查日期，并与代码的上线日期交叉核对。
- **作为重复关闭的链条。** 顺着 duplicate-of 关系追溯到标准工单。
- **私有工作区内容。** 如果无法访问某个 issue，把它记为缺口，而不是去猜。

## 返回什么

对每个相关工单：
- 工单 ID 和标题
- 从描述或评论中引用的问题/动机（不要转述。综合者需要确切文本来引证）
- 标签、父 issue、项目
- 作者、创建日期、关闭日期
- 工单链接（如可得）
