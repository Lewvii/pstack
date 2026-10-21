---
name: principle-migrate-callers-then-delete-legacy-apis
description: "在旧调用方仍然存在时引入新的内部 API，套用本原则。在同一波改动中迁移调用方并删除旧 API，而不是保留兼容层。"
user-invocable: false
---

# 先迁移调用方，再删除旧 API

- 更新测试，让它们断言新的契约；删除那些只保护重构前实现细节的测试

**适用条件：**
- 没有外部用户依赖向后兼容
- 项目能够承受协调一致的破坏性改动
- 新 API 属于某项简化或重构计划
