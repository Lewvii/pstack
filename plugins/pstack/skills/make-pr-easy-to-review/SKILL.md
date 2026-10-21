---
name: make-pr-easy-to-review
description: 通过清理嘈杂的历史、改进 PR 描述、添加 reviewer 指引来为评审准备好 PR，同时不改变代码行为。用于 "make this easy to review"、"tidy this PR"、"clean up commits" 或 "annotate the diff"。
---

# Make PR Easy to Review

准备好一个 PR，让 reviewer 能迅速理解意图、重要文件和风险。默认目标是在不改动行为的前提下提升可评审性。

## 工作流

1. 从用户提供的 URL 或当前分支确定目标 PR。
2. 检查 commit、diff 规模、改动路径、生成文件和 PR 描述。
3. 找出可评审性问题：嘈杂的 commit、过时的描述、无关改动、机械性改动与逻辑改动混杂、缺少测试，或 reviewer 的入口不清晰。
4. 在重写历史或 force-push 之前先提出方案。
5. 执行安全的改进，然后验证工作树或 diff 仍与预期代码一致。

## 历史清理

只在用户要求或同意方案时才重写历史。重写之前：

```bash
gh pr view <PR> --json title,headRefName,baseRefName,state,commits
git fetch origin <headRefName> <baseRefName>
ORIGINAL_TREE=$(git rev-parse origin/<headRefName>^{tree})
```

好的 commit 分组通常按依赖顺序排列：

1. Schema/存储，或生成的 API 定义。
2. 核心逻辑。
3. 接线与集成。
4. UI 或表层行为。
5. 测试。

重写之后，验证内容一致性：

```bash
echo "Original tree: $ORIGINAL_TREE"
echo "Current tree:  $(git rev-parse HEAD^{tree})"
git diff origin/<headRefName> --stat
```

如果工作树被意外改变，不要 push。

## Reviewer 指引

当代码行为应保持不变时，优先使用 PR 描述和评审说明：

- 加一段与真实 diff 相符的 TL;DR。
- 把核心文件与生成或机械性文件分开。
- 点出有风险的行为变更、迁移顺序、发布计划和测试覆盖。
- 当 issue 跟踪器、看板或设计文档能解释意图时，链接过去。

## 约束

- 绝不把有意义的行为变更藏在"清理"里。
- 除非用户明确要求，不要绕过 hook。
- 如果 PR 大到靠说明也无法变得可评审，建议拆分，而不是在问题周围打磨。
