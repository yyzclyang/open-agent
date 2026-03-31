---
description: 审查 pull request 或 staged changes 的代码质量、安全性和最佳实践
---

# 代码审查

审查 pull request 或 staged changes。

**用法：** `/review [pr-number]`

**步骤：**
1. 如果 args 中未提供 PR 号，使用 `bash("git --no-pager diff --cached -- . ':!pnpm-lock.yaml' ':!package-lock.json' ':!yarn.lock' ':!bun.lockb' ':!Gemfile.lock' ':!Cargo.lock'")` 获取 diff
2. 如果提供了 PR 号，使用 `bash("gh pr diff <number>")` 获取 diff
3. 分析更改并提供全面的代码审查，包括：
   - PR 概述
   - 代码质量和风格分析
   - 具体的改进建议
   - 任何潜在问题或风险

保持审查简洁但全面。关注：
- 代码质量和可维护性
- 安全漏洞
- 性能影响
- 测试覆盖率
- 文档完整性
- 破坏性更改
- 与代码库模式的一致性

使用清晰的章节和要点格式化审查。

**参数：** $1
