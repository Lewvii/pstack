# 在 GitHub Copilot 上设置 pstack

On GitHub Copilot this file replaces the model, effort, and hook questions in steps 1, 3, and 4 of [setup-pstack](SKILL.md), the sheet header in step 6, and step 7. Steps 2, 5, 8, and 9 still apply. pstack ships no Copilot model defaults, so every model in the sheet comes from the user's answers. Never pick a model the user has not chosen, and never write the Claude Code names in [Models](SKILL.md#models) or step 6's values into a Copilot sheet.

在一个至少与 gpt-5.4-mini 或 Sonnet 级 Claude 模型相当的模型上运行设置。在 Haiku 级模型上，大约一半的冒烟测试中，设置选中了用户从未选过的模型。

## 提问

Copilot 的 `ask_user` 每次调用只接受一个问题，且选项列表固定。该列表要么是 `choices` 列表，要么是 `requestedSchema` 中一个 `oneOf` 字符串属性，取决于 `ask_user` 是否采用 elicitation 表单形式。它的界面会自行加一个自由文本选项。把它当作单选处理，因为 Copilot app 的 `ask_user` 没有多选。每个模型问题都通过带 `choices` 列表的 `ask_user` 来问。绝不要把所有模型合成一个开放问题来问，比如哪些角色要改、改成什么，也绝不要在一次调用里塞两个问题。用连续的单选问题模拟多选，如下文的面板槽位所示。

- 不要推荐、预选或标注任何模型为 `(Recommended)`。首次设置没有可推荐的默认值。
- 重新运行时，把步骤 2 保存的该问题取值放在最前，标注 `(current)`。选中它就保持该值。
- 用户输入的答案只有是检测到的模型 ID 或 `inherit-parent` 时才有效；否则重问同一个问题。
- 用户在请求中已经给出的答案也算数。只问它尚未回答的问题。
- 当 `ask_user` 不可用（`-p` 运行，或 `--no-ask-user`）而仍有模型问题未定，不要替用户选择，也不要写覆盖表。告诉用户在交互式会话中运行 `setup-pstack`，并让本次会话的各个角色按 `inherit-parent` 运行。

## 当前状态

在 Copilot 上，步骤 2 的已保存值就是插件 hook 放入上下文的角色行。如果上下文表明这个 Copilot home 还没有 pstack 模型覆盖表，或者覆盖表无效，那就没有任何已保存值。只有在这些信息都不在上下文中时（例如仅安装技能的部署）才去读覆盖表。不要用 `bash` 探测覆盖表或其目录。每条这样的命令都会触碰工作区之外的路径并请求访问权限。只要 Copilot 在运行，`${COPILOT_HOME:-~/.copilot}` 就存在，所以不要创建它。

## 覆盖表表头

Copilot 覆盖表使用步骤 6 的角色行和设置，但表头换成下面这段，替代步骤 6 的那段说明，因为每个角色都保留一行，而 `check-sheet.sh` 会检查这一点：

```markdown
# pstack model configuration

Per-role model choices for pstack skills on GitHub Copilot, written by setup-pstack. Every role keeps its line; rerun setup-pstack to change one. A value of `inherit-parent` or `auto` runs that role on the parent session's model (the `task` call omits `model`); an alias entry in a panel list still counts toward that panel's fan-out. A model may carry a reasoning effort, as in `gpt-5.5 @xhigh` (levels: low, medium, high, xhigh, max), which the `task` call passes as `reasoning_effort`. `default effort` sets the level for a value without one; `session` keeps the parent session's effort. `session hook: off` stops the SessionStart hook from injecting the poteto-mode mandate; any other value, or no line, leaves it on. `panel vendors: any` records that the user kept a panel whose models come from one vendor.
```

<a id="detecting-models"></a>
## 检测模型

检测集合就是 `task` 工具的 `model` 枚举，使用枚举原样列出的完整 ID。绝不要把它缩写成家族名。如果本次会话没有 `task` 工具或其枚举不可见，就为每个角色写入 `inherit-parent`，跳过模型问题，并告诉用户在有 `task` 工具的会话中重新运行 `setup-pstack`。[Models](SKILL.md#models) 中的 Claude Code 名称和步骤 6 覆盖表形状中的取值都不是 Copilot 模型 ID；绝不要把它们写进 Copilot 覆盖表。

按厂商对检测到的 ID 分组，依据是每个 ID 第一个连字符之前的文本。`claude` 是 Claude，`gpt` 是 GPT，`gemini` 是 Gemini，`grok` 是 Grok，`kimi` 是 Kimi，其他 ID 归为 Other。同一厂商内保持枚举原有顺序。

<a id="choosing-one-model"></a>

## 选择单个模型

下面每个模型问题通过两次 `ask_user` 调用选定一个值，这样任何选项列表都不会容纳整个枚举：

1. **厂商。** 每个厂商一个选项并带上模型数量，例如 `GPT (12 models)`，然后加 `inherit-parent (run on the session's model)`。只有单个模型的厂商直接显示该模型的 ID，选中它即结束该问题。
2. **模型。** 该厂商的各个 ID，然后加 `Back to vendors`。

当检测集合加上 `inherit-parent` 不超过 8 个选项时，跳过厂商这一步，在一个问题里列出所有 ID。在 pstack 中 `auto` 与 `inherit-parent` 含义相同，因此不作为单独选项；已保存的 `auto` 在重新运行时会显示为 `auto (current)`。

## 按层级的角色

下面每个层级问题会写入其所在行的所有角色。面板行的角色各自采用整个面板列表。

| 层级 | 角色 |
| --- | --- |
| 默认 | `bug-fix`、`perf-issue`、`hillclimb`、`judgment and prose`、`how explorer`、`how explainer`、`why investigators`、`why synthesizer`、`reflect tooling`、`reflect judgment, divergent, synthesizer`、`swarm workers` |
| 最强 | `strongest judgment` |
| Worker | `feature, refactoring` |
| 面板 | `arena runners`、`arena cross-judge pool`、`architect runners`、`interrogate reviewers` |

<a id="question-order"></a>
## 问题顺序

按这个顺序提问。每个模型问题就是上面的厂商加模型两步；其他每个问题都是一次 `ask_user` 调用。

0. **会话模型。** "Run every pstack role on this session's model?"，选项为 `Yes, use the session's model for every role` 和 `No, choose models by tier`。这适用于只提供一个模型或只提供自动选择的方案，也适用于不想逐角色选择的用户。重新运行时，把与已保存覆盖表相符的选项放在最前，标注 `(current)`。选 Yes 会跳过问题 1 到 7 以及模型检测。把所有角色行设为 `inherit-parent`，每个面板角色写三个 `inherit-parent` 条目。然后问问题 8 和 9，并在步骤 8 说明面板的多样性被削弱，且面板条目只能靠推理强度区分。这条路径不需要 `panel vendors: any` 行，因为只含 `inherit-parent` 的面板无需检查厂商。选 No 则进入下面的问题。
1. **默认模型。** "Default model: runs <the Default row's roles>." 它会写入 Default 行的所有角色。
2. **Worker 模型。** "Worker model: runs <the Worker row's roles>." 它会写入 Worker 行的所有角色。
3. **最强模型。** "Strongest model: runs <the Strongest row's roles>." 它会写入 Strongest 行的所有角色。
4. **面板模型第 1 个（共 3 个）**，然后是**第 2 个（共 3 个）**，再是**第 3 个（共 3 个）**，作为三个独立问题。"Panel model N of 3: panels run one subagent per model, and models from different vendors catch different mistakes." 在第 2 和第 3 个槽位，先列出尚未进入面板的厂商，再列其余厂商，最后是 `inherit-parent`。不要提供已在面板中的模型。
5. **更多面板模型。** "Add a 4th panel model?"，第一个选项是 `Done`，第二个是 `Add a 4th`。选择添加会以同样方式再问一个槽位，然后询问是否加第 5 个。到 5 个模型就停止提供。
6. **厂商检查。** 当面板的模型 ID（不计 `inherit-parent`）全部来自同一厂商，且检测到了多个厂商时，询问 "The panel is single-vendor (Claude), so its cross-checks share blind spots."，选项为 `Pick the panel again` 和 `Keep it anyway`。重选会回到面板模型第 1 个。`Keep it anyway` 会在 `session hook` 行之后写入 `panel vendors: any` 行。只检测到一个厂商时，跳过此问题，写入 `panel vendors: any`，并在步骤 8 说明面板的多样性被削弱。只含 `inherit-parent` 的面板无需检查。
7. **覆盖。** "Override any individual role?"，第一个选项是 `No, write the sheet (Recommended)`，随后每个层级行一个选项，选项标签带上该行的当前值，例如 `Strongest (<its model ID>)`。选中某一行后会问要改其中哪些角色，然后按[选择单个模型](#choosing-one-model)的方式为该角色问一个模型，并把其当前值放在最前标为 `(current)`。面板角色则改按问题 4 到 6 的方式逐个询问其列表的槽位。每次覆盖之后重新问这个问题，直到答案为 `No, write the sheet`。
8. **Session hook。** "Keep the session hook that routes tasks to poteto-mode?"，选项为 `On (default)` 和 `Off`。重新运行时，已保存值放在最前并标注 `(current)`。
9. **默认强度。** "Default reasoning effort for pstack subagents?"，第一个选项是 `session (keep the parent's effort)`，然后是 `low`、`medium`、`high`、`xhigh` 和 `max`。它会写入 `default effort` 行。重新运行时，已保存值放在最前并标注 `(current)`。不要逐角色询问强度；角色值上已保存的 `@<level>` 后缀按原样保留。

问题 4 到 6 得到的面板按槽位顺序写入每个面板角色。重新运行时，每个层级问题的已保存值是它所在行的第一个角色，而面板模型第 N 个的已保存值是第一个面板角色的第 N 个条目。若某角色的已保存值与其问题的已保存值不同，则保留该角色自己的值，问题 6 会显示出来。

## 写入与检查

按步骤 5 所述进行校验。Copilot 的文件工具既不展开 `~` 也不展开变量，因此在 `bash` 中用 `echo "${COPILOT_HOME:-$HOME/.copilot}/pstack-models.md"` 打印覆盖表的绝对路径，并在一次工具调用中把整份覆盖表写到该路径，这样写操作最多只请求一次路径访问权限。

然后通过 `bash` 运行 `sh <this skill's directory>/scripts/check-sheet.sh`，传入该目录的绝对路径且不带参数。它按插件 hook 在会话开始时的方式检查 Copilot 覆盖表：每个角色都有一行，每个条目是 `inherit-parent`、`auto` 或带可选 `@<level>` 的模型 ID，并且每个面板都列出两个厂商的模型，除非覆盖表中有 `panel vendors: any`。它输出 `sheet ok`，或输出 `sheet invalid:` 及每个问题。出现 `sheet invalid` 时，修正这些行，整份覆盖表重新写入，再重新运行检查。插件 hook 会自动批准该检查，不会弹出提示。

本次会话余下时间使用你刚写入的值；不要回读覆盖表。之后的会话会从插件 hook 取得这些值，不再询问。

<a id="wire-it-in"></a>

## 接入配置

不要添加 include 行：Copilot 不会展开用户指令中的 `@~/` 路径，而且插件 hook 会在会话开始时注入覆盖表中的角色行。如果 hook 已开启，但本次会话的上下文缺少路由指令（即以 `You have pstack.` 开头的那块），说明它被另一个 hook 或仅安装技能的部署取代了。此时改为提议把下面这条常驻指令追加到 `~/.copilot/copilot-instructions.md`：

```text
pstack: for a task that touches more than one file, changes a signature other files call, involves a design choice, or is a bug with an unknown cause or a performance issue, load the poteto-mode skill and follow it. Resolve Claude tool and model names through poteto-mode's references/copilot-tools.md; role models are in ~/.copilot/pstack-models.md.
```
