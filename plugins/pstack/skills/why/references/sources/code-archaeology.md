# 代码考古（git + 仓库内）

## 这个来源包含什么

- 提交历史（消息、日期、作者、diff）
- PR 描述、评审评论和讨论串（通过 `gh`）
- 行内代码注释、TODO、FIXME、弃用说明
- ADR（架构决策记录），如果仓库有保留
- 测试，其名称和断言往往编码了驱动改动的边界情况
- 在同一批提交中被修改的相关文件（共变信号）
- 仓库中的 CHANGELOG 条目和发布说明
- 提交消息和 PR 正文中的 issue/工单 ID

这是最可信的来源，直接与代码绑定，也是最完整的。凡是经过仓库的东西都应该在这里。

## 如何搜索

扩展种子提交列表：

```bash
# History through renames
git log --follow --oneline -- <file>

# Pickaxe: commits that added or removed this exact text
git log -S '<exact_string_from_code>' -- <file>

# Or for patterns:
git log -G '<regex>' -- <file>

# Who wrote each line and when
git blame -L <start>,<end> <file>

# One commit's full diff
git show <hash>

# Commits between two points affecting this file
git log <old>..<new> -p -- <file>
```

寻找带外文档：

```bash
# ADRs often live in docs/adr/ or similar
rg -l -i 'architecture.decision' --glob '*.md'

# TODOs and FIXMEs near the target
rg -n -C2 '(TODO|FIXME|HACK|XXX|NOTE)' <target_file>

# Related tests
rg -l '<symbol>' --glob '*test*'
```
