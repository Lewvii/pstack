# 来源 playbook

why skill 为每个可用的证据类别派生一个调查者，每个调查者读取下面一份特定于来源的 playbook。这些 playbook 是针对常见 MCP 的具体示例。对于同一类别中的其他 MCP，请改编使用。

| 类别 | Playbook | 所记录的示例 MCP |
|---|---|---|
| 源码管理历史 | [`code-archaeology.md`](./sources/code-archaeology.md) | git, `gh` |
| Issue / 工单跟踪 | [`linear.md`](./sources/linear.md) | Linear（可改编用于 Jira、GitHub Issues、Plane、Shortcut） |
| 长文文档 | [`notion.md`](./sources/notion.md) | Notion（可改编用于 Confluence、Google Docs、Coda） |
| 实时团队聊天 | [`slack.md`](./sources/slack.md) | Slack（可改编用于 Discord、Microsoft Teams、Mattermost） |
| 基础设施可观测性 | [`datadog.md`](./sources/datadog.md) | Datadog（可改编用于 New Relic、Honeycomb、Grafana、Splunk） |
| 错误 / 异常跟踪 | [`sentry.md`](./sources/sentry.md) | Sentry（可改编用于 Rollbar、Bugsnag、Airbrake） |
| 产品分析数仓 | [`databricks.md`](./sources/databricks.md) | Databricks SQL（可改编用于 Snowflake、BigQuery、ClickHouse、dbt） |

跨类别：

- [`incident-postmortem.md`](./sources/incident-postmortem.md)。如果目标代码看起来是防御性的（空值检查、重试、超时、限流、功能开关、出站守卫、OOM 处理），加上这一份。
