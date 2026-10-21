# Datadog 遥测

## 这个来源包含什么

Datadog 保存的是运行时记录，即生产环境里实际发生了什么，而不是计划或讨论了什么。

- **指标。** 团队埋点的计数器、仪表盘值、直方图。一个指标的*存在*本身就说明有人认为这个数字值得关注。
- **监控与告警。** 团队认为值得把人叫醒的条件。一个在 `rate_limit_hit > 10/min` 时触发的监控，是团队担心这个阈值的直接证据。
- **仪表盘。** 团队认为对某个子系统重要的内容的精选视图。
- **APM trace 与 span。** 请求级的运行时数据，用于「为什么这里慢」/「为什么这里有超时」这类问题。
- **日志。** 大量的事件记录，往往包含驱动防御性代码的错误条件。
- **事故。** 带时间线和关联复盘的正式事故记录。
- **Notebook。** 团队的探索性调查，往往带有假设和分析。

## 如何搜索

使用 Datadog MCP。先宽后窄。

1. **确定所属服务。**

   ```
   search_datadog_services (filter by name or team)
   search_datadog_service_dependencies (see upstream/downstream)
   ```

2. **先看仪表盘和监控。**

   ```
   search_datadog_dashboards (query: feature name, service name, symbol)
   search_datadog_monitors   (same queries)
   ```

   当某个仪表盘或监控覆盖了目标时，记下它的查询和所关注的阈值。这个阈值往往就是「为什么这里被限制在 N？」的答案。

3. **目标周边的指标。**

   ```
   search_datadog_metrics (by name pattern, e.g., the feature or symbol)
   get_datadog_metric_context (metadata: description, units, tags)
   get_datadog_metric (timeseries; "was there a spike around the PR date?")
   ```

   把指标的走势与目标的新增/修改日期对照，是很强的支持性证据："the `payment_timeout` metric spiked 2023-11-03, and the retry logic merged 2023-11-06."

4. **日志。要收窄，不要倾倒。**

   ```
   search_datadog_logs (raw log patterns near the target, set use_log_patterns=true)
   analyze_datadog_logs (SQL-style aggregations, only when you need counts)
   ```

   用符号、错误字符串或功能名搜索。**强烈建议使用限定时间的查询**（例如改动前后各 30 天）。

5. **APM span 与 trace。**

   ```
   aggregate_spans    (stats: "how often does this endpoint fail?")
   search_datadog_spans (inspect individual spans)
   get_datadog_trace  (a specific trace ID)
   ```

   适用于超时、重试、慢路径和跨服务行为。

6. **事故。**

   ```
   search_datadog_incidents (by title, team, date range)
   get_datadog_incident     (full detail for a specific incident)
   ```

   如果目标看起来是防御性的，搜索它被加入前后的事故。一个时间线里写着 "added defensive check for X" 的事故，几乎就是直接证据。

## 好的证据在这里长什么样

- 一个监控，其查询和阈值与代码强制的约束相吻合（代码限制在 100，监控在请求超过 100/min 时告警）
- 一个由目标作者创建的仪表盘，其组件与代码所度量或防范的内容相对应
- 一个指标显示代码合并前生产环境出现尖峰，合并后数值稳定
- 一条事故记录引用了目标代码、相同的符号或相同的错误字符串
- 日志显示了防御性代码会阻止的特定错误模式，时间戳位于改动之前的窗口内

## 常见陷阱

- **相关不等于因果。** PR 之前的尖峰和之后的平稳只是提示，而非定论，因为同一窗口内可能还有其他改动上线。检查相邻的 PR。
- **过度拟合于你找到的那张图。** 一张图反映的是制图者的视角。一张名为 "retry success rate" 的图，证明的是团队关心重试成功率，而不是某一行代码存在的原因。
- **消失的遥测数据。** 指标可能被重命名、删除，或保留期很短。如果找不到相关窗口内的数据，那是一个缺口，而不是空结果。
- **规模带来的噪声。** 积极地按服务、标签和时间窗口收窄日志搜索。用 `analyze_datadog_logs` 做聚合，而不是倾倒原始日志。
- **有埋点 != 是原因。** 一个指标说明有人在意到要去度量它，而不是代码*因为*它而被加入。与提交/PR 日期交叉核对。

## 返回什么

对每个相关条目：
- 类型（dashboard / monitor / metric / log pattern / trace / incident / notebook）
- 标题或名称
- 链接或标识符（dashboard ID、monitor ID、指标名、incident ID）
- 所有者/作者，以及创建/修改日期
- 与问题相关的具体条件、查询或引文（尽量逐字）
- 相关性：它对目标代码说明了什么，以及关联有多强
