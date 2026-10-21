# 事故与事后复盘上下文

- **Linear**：查找打了 `incident`、`sev-*`、`postmortem-action-item`、`reliability` 标签的工单
- **Slack**：在目标代码被加入的日期前后，搜索 `#sev-*` 和 `#incident-*` 频道
- **Git**：消息形如 "fix for incident"、"add defensive check"，或 "revert" 之后跟着 "re-apply with..." 的提交，是很强的信号
