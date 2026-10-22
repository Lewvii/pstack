# Bugbot 分诊

当 Babysit playbook（`../playbooks/babysit.md`）处理 Bugbot 或评审自动化的评论时，使用这份参考。目标不是默认忽略 Bugbot，而是不再把每条评论都当成必须改代码的要求。

## 决策准则

行动之前先给每个 Bugbot 讨论串分类：

- `fix`：这条评论指出了可信的正确性、安全、隐私、数据丢失、鉴权、计费、迁移、幂等、竞态或已交付行为的问题。在最低的所属 PR 里修掉，然后回复 commit SHA 并解决该讨论串。
- `dismiss`：这条评论命中某个已有记录的、低风险的噪声模式，且当前代码与上下文证明该顾虑不需要改动代码。用简短理由回复并解决讨论串。
- `ask`：这条评论是新出现的、高严重度的、与安全/隐私/数据相关的，或含义不清。问用户，不要猜。

拿不准时就问。跳过一条噪声性的代码质量评论代价很低；跳过真实的数据或安全缺陷则不是。

## 已习得模式的格式

按这个形状添加将来的模式：

```markdown
### <short pattern name>

- Confidence: candidate | recurring | strong
- Skip when: <conditions that must be true>
- Do not skip when: <risk boundaries>
- Example signal: <phrases or code context that identify the pattern>
- Source: <PR/comment URL or short historical note>
```

用 `candidate` 表示只有一两个例子。多次真实驳回之后用 `recurring`。只有当模式很窄、被反复验证且低风险时才用 `strong`。

## 常见的可跳过候选

### 有意的 UI 或设计系统视觉改动

- Confidence: candidate
- Skip when: PR 描述、截图、设计评审或邻近代码已经让这次视觉改动有据可查，而 Bugbot 的评论只是在重复某个共享视觉默认值变了。
- Do not skip when: 评论指向无障碍、焦点可见性、键盘导航、颜色对比度，或 PR 并未有意改动的组件 API 契约。
- Example signal: 关于焦点轮廓、按钮尺寸、间距或共享组件视觉默认值的评论，而负责人回复了「intentional」或「intended」。

### Bugbot 看不到的 upstack 或栈内用法

- Confidence: candidate
- Skip when: Bugbot 把某个导出、组件、辅助函数或文件标为未使用，而当前 forge 的 PR 列表与 diff、上层栈的 diff 或 PR 上下文表明它被栈中更晚的 PR 使用。
- Do not skip when: 当前 PR 不在某个栈里、该符号是公开 API，或所谓的 upstack 用法无法被验证。
- Example signal: 「Exported component is never used」，并有人回复「used upstack」。

### 并行实现期间的临时重复

- Confidence: candidate
- Skip when: PR 有意重复一小段代码，以让新路径与正在被删除、替换或验证的旧路径并行存在。
- Do not skip when: 被重复的代码影响安全、计费、数据访问、API 行为，或一个长期存活的共享抽象明显会降低风险。
- Example signal: 「Significant duplication」或「duplicated validation logic」，而负责人说明旧路径将被删除，或重复逻辑是有意局部的。

### 既有框架或组件不变量已覆盖该警告

- Confidence: candidate
- Skip when: 该顾虑已由当前 diff 或附近代码中可见的共享组件、框架契约、类型不变量或单一事实来源保证。
- Do not skip when: 该不变量只是假定而未强制、依赖时序，或跨越异步/状态边界、值可能分叉。
- Example signal: 关于内层 popover 缺少 max-height 的评论，而共享 popover 已强制视口边界；或可空值场景中，本地被检查的值与传入的值来自同一来源。

### 负责人已声明的后续跟进或延后清理

- Confidence: candidate
- Skip when: PR 负责人明确说明该问题是有记录的后续项，当前 PR 没有使该行为变差，且评论不涉及高风险区域。
- Do not skip when: agent 在负责人未参与的情况下行动、该问题属于中高严重度的产品行为，或延后会导致合并一个新增回归。
- Example signal: 「I'll worry about that later」或「we'll delete this eventually」。

### 自我撤回或明确误报的规则评论

- Confidence: recurring
- Skip when: 评论正文或 Bugbot 之后的回复明确指出该发现已撤回、符合规则或是误报，且 agent 能在本地验证相关规则。
- Do not skip when: 唯一证据只是有人在高风险问题上说了「false positive」而没有解释。
- Example signal: 一条文件命名规则评论，其正文说该文件已符合规则。

## 默认询问

即使之前的 PR 驳回过类似的评论，也不要自动跳过这些类别：

- 安全、隐私、鉴权、计费、数据保留、训练数据以及权限边界的发现。
- 高严重度的发现。
- 迁移、schema、幂等、并发以及跨系统行为的发现。
- 建议的修复很小、明显降低风险且不改变产品意图的评论。

历史数据显示，人有时会驳回安全/数据流的评论。把这些当作负责人的判断，而不是全团队的跳过规则。

## 来自近期 babysit 的候选经验

当新的候选经验看起来对团队有用但尚未成熟时，在 babysit 过程中或之后追加到这里。等若干个 PR 确认了某个模式之后，优先把 `recurring` 的候选提升到上面的小节。

### 手工重新实现浏览器原生行为

- Confidence: candidate
- Skip when: 实际上几乎不要跳过。当 diff 用人工等价物替换浏览器原生行为时（原生 sticky → JS 定位的克隆，原生滚动定位 → 转发的 wheel/touch 事件，paint-order 遮挡 → mask/clip-path），Bugbot 针对这类代码提出的逻辑缺陷一直都被证明是成立的。
- Do not skip when: 该发现涉及事件转发缺口（wheel deltaMode、touch pan、边缘的滚动链、tap slop）、mask/clip 命中测试差异，或此类代码中 observer 与 React 状态之间的时序竞态。默认选择 fix。
- Example signal: 「masks do not affect hit-testing」「overlay blocks wheel scroll」「ignores deltaMode」「runs in the IntersectionObserver callback before React applies state」。
- Source: 一个 sticky 遮挡 PR：Bugbot 跑了六轮，约十八条发现，每一条都修掉了而不是驳回。

### 契约测试漂移的声称可以廉价验证，先跑测试

- Confidence: candidate
- Skip when: 验证本身永远不要跳过，它只花一条命令。当 PR 附带一个钉住协议或文档措辞的契约测试（对 SKILL.md 的正则、文档措辞的快照），而 Bugbot 声称「测试已经与文档不符」（或反之），在分类之前先在 PR tip 上跑那个测试。红的运行从经验上确认了该声称；绿的运行则是驳回回复的具体反证。
- Do not skip when: n/a：这是验证捷径，不是驳回模式。注意重复轮次的「倾向驳回」启发式在这里会误判：钉住散文的测试之所以漂移，恰恰是因为先前的修复轮次改动了那段散文。
- Example signal: 「Contract test omits the pre-fix wait」，且该 PR 更早的修复提交改写了被钉住的段落；在 tip 上跑该测试正好在它所引用的断言上失败。
- Source: 一个钉住散文的 PR，Bugbot 跑了八轮；该声称在第 7 轮确实成立，尽管之前每一轮都已修复并解决。

### 同一 PR 更晚处已修掉的过期安全评审发现

- Confidence: candidate
- Skip when: 某个 agentic 安全评审（或类似工具）声称缺少一处鉴权/校验调用，而当前 PR tip 明显已包含那道确切的门禁（带测试），通常是在评审运行之后的一个加固提交里补上的。
- Do not skip when: 所引用的辅助函数对当前讨论的主体是空操作、该检查运行在它所保护的副作用之后，或对所声称主体的覆盖缺失。
- Example signal: 一条 HIGH 等级的「missing authorization check」，而该守卫在 tip 上已经在副作用之前被调用。
- Source: 一个 webhook 端点 PR，其加固提交晚于评审运行。

### 放宽一个刻意收窄的错误条件会掩盖真正的错误

- Confidence: candidate
- Skip when: 该发现要求把一个收窄的错误条件（特定的 `errno`、错误码或状态类别）放宽成兜底捕获，而那种收窄编码了真实的区分。典型形态是只在 `ENOENT` 上触发的依赖回退：「二进制未安装」与「命令运行了但失败」是不同情形。在任何非零退出上重试，会把一次合法的失败（未找到、鉴权过期、网络）重放到回退路径上，然后报告回退的错误，从而掩盖真正的那个。
- Do not skip when: 收窄的条件漏掉了同一类别里的情形（另一个「二进制不可用」的 errno，如 `EACCES`，或另一种传输层失败）、未处理的路径会丢数据或留下部分状态，或该重试是幂等的且原始错误仍被暴露。
- Example signal: 「only retries when X fails with ENOENT … never tries the fallback even when a working Y exists」，指向的代码其回退是为缺失依赖而存在，而不是为失败的操作。
- Source: 一个 CLI 重命名 PR，其回退是为缺失的二进制而存在，而不是为失败的命令。
