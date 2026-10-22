# 恢复存储

暂停与拾取共用一个项目定位器。`skills/poteto-mode/scripts/` 下的 `resume.mjs` 命令解析项目的 Git 公共目录，并把检查点存放在它的 `pstack/resume/` 目录下。`latest.json` 指向最新的完整检查点，并记录 worktree、分支、时间戳、备注，以及带内容哈希的所请求产物。

Claude Code、Codex 和其他运行时使用同一个命令和位置。各运行时专属的 home 目录与会话记录编码不影响查找。同一仓库的 Git worktree 共用这个定位器，Git 会把它的内容排除在提交之外。这要求有一个本地 Git 检出，且其公共目录能在中断后存活。删除或重新克隆仓库后它不会存活；在那之前先把完整检查点转移到持久存储，并把定位器交给恢复的会话。

## 写入检查点

1. 运行 `node <scripts>/resume.mjs begin --project <worktree>` 分配一个新的检查点目录。在里面写 `resume.md`，记录意图、已验证的进度、当前状态和下一步。
2. 把所请求的产物保存在该目录，并从备注里链接它们。当项目中原有产物的位置能挺过预期的中断时，复用它。不要制造互相竞争的副本。对于将被删除的 worktree，把所请求的材料放进公共检查点存储。在要求严格的地方核对确切措辞、完整性与顺序。
3. 用 `node <scripts>/resume.mjs publish --project <worktree> --note <resume.md> --artifact <file> ...` 发布。把备注里每个本地链接都登记为产物，包括已有的项目文件。Link each local file with an inline Markdown link whose destination is the path, written bare or as `<path>`, with no square brackets in the link text and no title after the path. Reference-style links are not read. The path is relative to the note or absolute, and it is percent-decoded, so write a literal `%` as `%25`. Wrap a path that contains spaces or parentheses in angle brackets, or percent-encode it. Links inside code spans and fenced code blocks are ignored. Each rejected link reports this rule. 这条规则的意思是：用行内 Markdown 链接指向每个本地文件，目标就是路径，写成裸路径或 `<path>`，链接文本里不能有方括号，路径之后不能有标题。引用式链接不会被读取。路径相对备注或为绝对路径，会被百分号解码，所以字面量 `%` 要写成 `%25`。含空格或圆括号的路径用尖括号包裹，或做百分号编码。代码段和围栏代码块里的链接会被忽略。命令在原子替换 `latest.json` 之前会校验文件归属、链接和内容。发布失败不得当作一个就绪的检查点。
4. 运行 `node <scripts>/resume.mjs read --project <worktree>`，打开它的备注和产物。把已发布的检查点文件当作不可变。之后的暂停使用新的检查点目录。如果既有项目产物发生变化，拾取会报告该差异，而不是默默把它当成已保存的材料。

`begin` 会分隔并发写者的文件。发布以原子方式替换一个共享的最新指针；最后一次完整的发布胜出。当多个任务需要各自可恢复时，保留返回的检查点路径。这个指针是定位器，不是任务调度器或执行锁。

## 读取检查点

会话拾取从 `node <scripts>/resume.mjs read --project <worktree>` 开始，只用项目目录。以 `kind: checkpoint` 退出 0 会给出已保存的备注和全部已登记的产物。在搜索会话记录之前先打开它们。命令会再次校验哈希和链接，包括项目产物的复用。

以 `kind: missing` 退出 2 表示没有指针，所以改用会话记录、云端 URL 或分支作为回退。以 `kind: unavailable` 退出 1 表示检查点畸形、不可读、已变化或不完整。诊断那个具体的缺口；不要默默忽略检查点，也不要声称材料从未被保存。这些命令都不会让正在运行的任务暂停。
