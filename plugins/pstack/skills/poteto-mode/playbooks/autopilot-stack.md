### Autopilot-stack

**栈由你负责，落地不归你。交给操作者一条线性的、基于分支链的 PR 栈，由他审查并落地。** **Autopilot-full** 除验证过的工作如何离开之外，其余照旧。每轮都从已安装插件下的磁盘重读两个 playbook，二者冲突时以本 playbook 为准。

1. **运行 Autopilot-full 的负责人，去掉落地环节。** 负责人报告 STACK-READY 而不是 merge-ready。他们绝不 rebase 到其他分支（包括 trunk），绝不 retarget、合并、关闭 PR 或启用自动合并，因为链的拓扑归根所有，落地归操作者所有。某项依赖另一项时，它的负责人从该项的分支分出，而不是从 trunk 分出。
2. **拓扑只留一个写者。** 负责人只推自己的分支，并报告每个分支顶端及其预期的父分支。只有你 rebase 或 retarget 某一环，因为一条链上两个写者会互相抵消。按依赖顺序追加每个干净的 PR，把它的 base 设为其父分支（`origin pr edit <pr> --base <parent-branch>` 或 `gh pr edit <pr> --base <parent-branch>`）。绝不要求 `gt`。任何重写或 retarget 之前，遵循 [合并与重排安全](../references/merge-safety.md)：捕获受影响的链及其远端 head，取消该链的自动合并请求和队列成员身份，核实取消已生效，并用 `--force-with-lease=refs/heads/<branch>:<captured-remote-head>` 推送。负责人把你的推送当作新的 base 和 lease，遇到冲突时解决自己那一份，交给你推送。
3. **在根上吸收漂移。** 落地开始之前，对每一环套用 Autopilot-full 第 5 步的测试，凡是判为需要的，自下而上 rebase 整条链。然后逐环套用 patch-id 规则，因为一环的 base 是它的父分支。记录每个结论的 head SHA、base SHA、base 分支和 patch-id，并把结论不再适用、或可能不适用于新 base 的某一环送回验证，再交付。
4. **交付整条链，之后别再碰它。** 每一环在它的 PR 里带上结论。落地一旦开始，除 `playbooks/shipping.md` 许可的合并后 retarget 外，任何人都不再写这条链，因为第二个写者会毁掉进行中的落地。落地中的问题经由你交给操作者，任何修复都等操作者的放行。

**在两个 autopilot 之间选择。** PR 相互独立且已授予落地权限时用 Autopilot-full。操作者希望在落地前审查、工作有先后或耦合关系、或者不授予合并权限时用 Autopilot-stack。

**回复：** 按 **ketchup** 技能让操作者跟上进度，附上栈根和栈顶的链接、每一环一行结论，以及任何被搁置或排除的事项及原因。
