# Databricks 分析与系统表

## 这个来源包含什么

Databricks 是产品分析、数据管道和数仓遥测层。它与 Datadog 互补。Datadog 是*基础设施/运行时*视角，Databricks 是*产品/数据*视角（用户做了什么、跑了哪些实验、功能使用如何演变、某个阈值常量从何而来）。

- **产品分析事件。** `your_warehouse.events.analytics_track_event`（原始表），以及 `<your_analytics_db>.<schema>.<table>` 中按事件划分、带类型、已去重的 dbt 模型。记录用户行为：功能调用、点击、接受/拒绝、提交、客户端上报的错误。
- **用量与计费事件。** `your_warehouse.events.usage_event` / `<your_analytics_db>.<schema>.stg_usage_events`，`your_warehouse.events.raw_model_event` / `<your_analytics_db>.<schema>.stg_raw_model_events`。用于由成本或用量驱动的决策。
- **实验 / 功能开关数据。** 曝光表和结果表。**schema 因公司而异。** 在假定表名之前先用 `SHOW TABLES` 探查。
- **系统表。** `system.query.history`、`system.compute.warehouses`、`system.billing.*`、`system.access.audit`。回答「这个查询贵不贵？」「有人多久跑一次？」「数仓负载什么时候飙升？」
- **dbt 血缘。** `<your_analytics_db>.<schema>` 中的模型揭示了哪些管道依赖某张表/某个字段。上游变更经常是消费端代码改动的动机。
- **Databricks notebook。** 工程师在改代码之前写的探索性分析。**无法通过 SQL MCP 查询。** 如果你怀疑理由就在某个 notebook 里，把它作为缺口点名。

## 如何搜索

使用 Databricks SQL MCP。主要工具：`execute_sql_read_only`。如果它返回 `statement_id`，用 `poll_sql_result` 轮询，而不是重新执行。

**查询前先定位。** schema 因公司而异。在信任某个表名之前先探查：

```sql
SHOW TABLES IN <your_analytics_db>.<schema> LIKE '*<keyword>*';
DESCRIBE TABLE <your_analytics_db>.<schema>.stg_<event>;
```

**每个查询都要限定时间。** 这些表非常大，不加约束的扫描会超时。按 `_timestamp`（事件）或 `start_time`（`system.query.history`）过滤，时间窗口要框住上线日期，通常是前后各约 30 天，只有理由充分时才放宽。

**优先用带类型的 dbt 模型，而不是原始表。** `<your_analytics_db>.<schema>.<table>` 已去重、带类型并做了 liquid clustering。`your_warehouse.events.analytics_track_event` 有重复，且 `properties_json` 没有类型。模型命名模式：`stg_<source>_<event_name_with_underscores>`，其中 `<source>` 是 `app`、`backend`、`website` 或 `cli`。当仅凭模式无法确定时，用 `SHOW TABLES` 确认确切的模型名。只有在还没有 dbt 模型，或需要 dbt 刷新延迟期内的事件时，才退回原始表。

**带类型的 dbt 模型上的列约定**（了解这些可以省掉一次 `DESCRIBE` 往返）：

- `_timestamp`、`_id`、`_auth_id`、`_request_id`、`event_name`。每个模型都有的标准列
- `properties_<name>`。带类型、下划线命名的事件属性（`properties_entrypoint`、`properties_size_bytes`，…）
- `context_team_id`、`context_client_version`、`context_country`、`context_client_os`。预先提取的客户端上下文

### 往往有收获的调查模式

挑选与目标相匹配的表 + 列组合：

1. **事件用量轨迹。** 在 PR 合并前后 ±30 天窗口内，对相关 `stg_*` 模型做按天计数。在合并后一两天内从零跃升到稳定用量的阶跃函数，是 PR 上线了该功能的有力旁证。衰减到零则暗示弃用或删除。
2. **护栏 / 防御性检查的来源。** PR *之前* 14 天内相关 `properties_<name>` 列的分布（中位数 / p99 / 最大值）。如果 p99 与目标的阈值常量吻合，说明这个数字可能是根据数据选定的。
3. **实验 / 功能开关查询。** 用 `SHOW TABLES ... LIKE '*experiment*'` 找到曝光表，然后拉取 PR 日期前后相关 flag key 按变体划分的曝光计数。
4. **迁移、回填或性能重写的查询历史证据。** 对 `system.query.history` 按 `statement_text ILIKE '%<table_or_symbol>%'` 过滤，并用较紧的 `start_time` 窗口，能找出很可能驱动了改动的昂贵查询（按 `total_duration_ms` 排序，或聚合 `SUM(read_bytes)`、`COUNT(*)`）。
5. **dbt 血缘。** 如果目标从某个 `<your_analytics_db>.<schema>` 模型读取或写入，该模型自身的 git 历史（在本仓库中）往往带着理由。把这条线索交回给 git 调查者，而不是自己去追。

## 好的证据在这里长什么样

除了上面的模式之外：

- 某个错误分类事件的计数在一个防御性代码 PR 之后的几天里降到接近零。暗示该 PR 解决了这类错误
- 某条曝光表记录点名了目标的功能开关 key，并在 PR 上线日期前后带有 "shipped" / "concluded" 的决定

## 常见陷阱

- **有埋点 ≠ 是原因。** 一个事件的存在说明有人在意到要记录它，而不是说目标代码*因为*它而存在。在声称因果之前，要配上 git 调查者提供的 PR/提交引证。
- **无声的埋点变更。** 事件量的阶跃可能意味着开始记录一个新事件，而不是用户行为变了。在把这种爬升解读为功能上线信号之前，先检查同一窗口内是否有埋点相关的 PR。
- **schema 漂移。** 事件属性会演变。今天带类型 dbt 模型上的某一列，在目标代码编写时可能还不存在。较早的数据可能只在原始 `properties_json` 中带有该属性。
- **dbt 刷新延迟。** `<your_analytics_db>.<schema>.*` 按计划重建（通常每小时/每天）。对于最近几个小时的事件，退回 `your_warehouse.events.*` 并按 `_id` 去重。
- **因公司而异的表。** 实验、功能开关、计费和用量表各不相同。报告一个你从未确认过其存在的表的结果，是典型的失败模式。先用 `SHOW TABLES` / `DESCRIBE TABLE` 探查。
- **保留期断崖。** 如果相关窗口早于该表的保留期或 dbt 模型的创建日期，那是一个*缺口*，而不是空结果。明确点名它，以免综合者把「没有结果」读成「没有活动」。
- **notebook 无法查询。** SQL MCP 看不到 Databricks notebook。如果你怀疑理由就在某个 notebook 里，返回一个缺口。

## 返回什么

对每项相关发现：
- 类型（产品事件 / 实验曝光 / 用量或计费事件 / 系统表记录 / dbt 模型）
- 完全限定的表名，以及你运行的确切查询
- 查询的时间窗口
- 紧凑的数值摘要（计数、百分位、首次/末次出现时间戳）。**不要倾倒原始行。**
- 与目标上线日期的时间相关性（例如 "first row 2024-08-15, PR #49074 merged 2024-08-14"）
- 相关性 + 强度：direct / circumstantial / weak
