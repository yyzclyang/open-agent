# 代码质量审查员提示词模板

在调度代码质量审查 subagent 时使用此模板。

**目的：** 验证实现构建质量良好（干净、经过测试、可维护）

**只在规格合规审查通过后才调度。**

```
Task tool (superpowers:code-reviewer):
  使用 requesting-code-review/code-reviewer.md 中的模板

  WHAT_WAS_IMPLEMENTED: [来自实现者的报告]
  PLAN_OR_REQUIREMENTS: [plan-file] 中的任务 N
  BASE_SHA: [任务前的提交]
  HEAD_SHA: [当前提交]
  DESCRIPTION: [任务摘要]
```

**除标准代码质量关注点外，审查员还应检查：**
- 每个文件是否有一个清晰的职责和明确定义的接口？
- 单元是否被分解到可以独立理解和测试的程度？
- 实现是否遵循计划中的文件结构？
- 此实现是否创建了已经很大的新文件，或显著增大了现有文件？（不要标记预先存在的文件大小——关注此变更带来的影响。）

**代码审查员返回：** 优点、问题（严重/重要/次要）、评估结论
