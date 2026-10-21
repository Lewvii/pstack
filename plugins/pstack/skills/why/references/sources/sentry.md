# Sentry 错误历史

## 如何搜索

使用 Sentry MCP。

1. **定位。** 如果你不知道项目 slug 和组织：

   ```
   find_organizations
   find_projects
   ```

2. **搜索与目标相关的 issue。**

   ```
   search_issues (natural language, e.g., "errors in PaymentService timeout", "unhandled exceptions in uploadFile")
   ```

   用目标所处理的异常类名、它的函数名或类名、它检查的错误消息字符串，以及它的文件路径来构造查询。

3. **按发布版本和时间窗口收窄。**

   ```
   search_issue_events (filter by release, time, environment, trace ID, tags)
   get_issue_tag_values (for an issue, see distribution across versions, users, environments)
   ```

   对一个可疑的 issue，检查 **first seen** 和 **last seen**（它停止出现的时间是否与目标的上线日期吻合？）、**受影响的发布版本**（哪些版本出现过，哪个是修复版本？），以及**频率走势**（是否先飙升，然后被解决？）。

4. **拉取完整事件以获取上下文。**

   ```
   get_sentry_resource (pass a Sentry URL or type+ID)
   ```

   堆栈是否经过目标代码？标签和 breadcrumb 是否与目标所防范的条件相符？

5. **检查在目标附近上线的发布版本。**

   ```
   find_releases (around the commit date of the target)
   ```

   把发布版本与 PR 的合并日期交叉核对。

6. **谨慎使用 Seer。**

   ```
   analyze_issue_with_seer
   ```

   Seer 的 AI 根因分析是一个假设生成器。把它们当作推断，而不是权威结论。实际的事件和堆栈才是主要证据。

## 返回什么

对每个相关 issue：
- Issue ID 和标题
- 项目和组织
- 首次出现 / 末次出现时间戳
- 事件计数（如已知，附采样率）
- 受影响的发布版本
- 一段有代表性的堆栈片段，展示与目标的相关性（逐字摘录，而不是总结）
- 首次/末次出现时间与目标上线日期的相关性
- Issue 链接
- 任何作者评论或解决说明
