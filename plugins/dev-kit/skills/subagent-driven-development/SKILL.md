---
name: subagent-driven-development
description: 当在当前会话中执行包含独立任务的实施计划时使用
---

#  subagent 驱动开发

通过为每个任务调度新的 subagent 来执行计划，每个任务完成后进行两阶段审查：先审查设计合规性，再审查代码质量。

**为什么用 subagent：** 你将任务委派给具有隔离上下文的专业 agent。通过精确构建它们的指令和上下文，确保它们保持专注并成功完成任务。它们不应继承你当前会话的上下文或历史——你只需构建它们所需的内容。这也为你自己的上下文保留了空间，用于协调工作。

**核心原则：** 每个任务一个新 subagent + 两阶段审查（设计合规 → 代码质量）= 高质量、快速迭代

## 何时使用

```dot
digraph when_to_use {
    "有实施计划？" [shape=diamond];
    "任务大多独立？" [shape=diamond];
    "留在当前会话？" [shape=diamond];
    "subagent 驱动开发" [shape=box];
    "执行计划模式" [shape=box];
    "手动执行或先头脑风暴" [shape=box];

    "有实施计划？" -> "任务大多独立？" [label="是"];
    "有实施计划？" -> "手动执行或先头脑风暴" [label="否"];
    "任务大多独立？" -> "留在当前会话？" [label="是"];
    "任务大多独立？" -> "手动执行或先头脑风暴" [label="否-紧密耦合"];
    "留在当前会话？" -> "subagent 驱动开发" [label="是"]; "留在当前会话？" -> "执行计划模式" [label="否-并行会话"]; } ``` 
**vs. 执行计划（并行会话）：**
- 同一会话（无上下文切换）
- 每个任务一个新 subagent（无上下文污染）
- 每个任务后两阶段审查：先设计合规，再代码质量
- 更快迭代（任务间无需人工介入）

## 流程

```dot
digraph process {
    rankdir=TB;

    subgraph cluster_per_task {
        label="每个任务";
        "调度实现者 subagent (./references/implementer-prompt.md)" [shape=box];
        "实现者 subagent 有问题？" [shape=diamond];
        "回答问题，提供上下文" [shape=box];
        "实现者 subagent 实现、测试、提交、自我审查" [shape=box];
        "调度设计审查 subagent (./references/design-reviewer-prompt.md)" [shape=box];
        "设计审查 subagent 确认代码匹配设计？" [shape=diamond];
        "实现者 subagent 修复设计差距" [shape=box];
        "调度代码质量审查 subagent (./references/code-quality-reviewer-prompt.md)" [shape=box];
        "代码质量审查 subagent 通过？" [shape=diamond];
        "实现者 subagent 修复质量问题" [shape=box];
        "在 TodoWrite 中标记任务完成" [shape=box];
    }

    "读取计划，提取所有任务的完整文本，记录上下文，创建 TodoWrite" [shape=box];
    "还有剩余任务？" [shape=diamond];
    "调度最终代码审查 subagent 审查整个实现" [shape=box];
    "向用户报告完成" [shape=box style=filled fillcolor=lightgreen];

    "读取计划，提取所有任务的完整文本，记录上下文，创建 TodoWrite" -> "调度实现者 subagent (./references/implementer-prompt.md)";
    "调度实现者 subagent (./references/implementer-prompt.md)" -> "实现者 subagent 有问题？";
    "实现者 subagent 有问题？" -> "回答问题，提供上下文" [label="是"];
    "回答问题，提供上下文" -> "调度实现者 subagent (./references/implementer-prompt.md)";
    "实现者 subagent 有问题？" -> "实现者 subagent 实现、测试、提交、自我审查" [label="否"];
    "实现者 subagent 实现、测试、提交、自我审查" -> "调度设计审查 subagent (./references/design-reviewer-prompt.md)";
    "调度设计审查 subagent (./references/design-reviewer-prompt.md)" -> "设计审查 subagent 确认代码匹配设计？";
    "设计审查 subagent 确认代码匹配设计？" -> "实现者 subagent 修复设计差距" [label="否"];
    "实现者 subagent 修复设计差距" -> "调度设计审查 subagent (./references/design-reviewer-prompt.md)" [label="重新审查"];
    "设计审查 subagent 确认代码匹配设计？" -> "调度代码质量审查 subagent (./references/code-quality-reviewer-prompt.md)" [label="是"];
    "调度代码质量审查 subagent (./references/code-quality-reviewer-prompt.md)" -> "代码质量审查 subagent 通过？";
    "代码质量审查 subagent 通过？" -> "实现者 subagent 修复质量问题" [label="否"];
    "实现者 subagent 修复质量问题" -> "调度代码质量审查 subagent (./references/code-quality-reviewer-prompt.md)" [label="重新审查"];
    "代码质量审查 subagent 通过？" -> "在 TodoWrite 中标记任务完成" [label="是"];
    "在 TodoWrite 中标记任务完成" -> "还有剩余任务？";
    "还有剩余任务？" -> "调度实现者 subagent (./references/implementer-prompt.md)" [label="是"];
    "还有剩余任务？" -> "调度最终代码审查 subagent 审查整个实现" [label="否"];
    "调度最终代码审查 subagent 审查整个实现" -> "向用户报告完成";
}
```

## 模型选择

使用能处理每个角色的最低能力模型，以节约成本并提高速度。

**机械性实现任务**（隔离函数、清晰设计、1-2 个文件）：使用快速、便宜的模型。大多数实现任务在计划设计明确时都是机械性的。

**集成和判断任务**（多文件协调、模式匹配、调试）：使用标准模型。

**架构、设计和审查任务**：使用最强可用模型。

**任务复杂度信号：**
- 涉及 1-2 个文件且设计完整 → 便宜模型
- 涉及多个文件且有集成关注 → 标准模型
- 需要设计判断或广泛的代码库理解 → 最强模型

## 处理实现者状态

实现者 subagent 报告四种状态之一。正确处理每种情况：

**DONE：** 进入设计合规审查。

**DONE_WITH_CONCERNS：** 实现者完成了工作但标记了疑虑。在继续之前阅读这些疑虑。如果疑虑关于正确性或范围，在审查前先处理。如果是观察性的（如"这个文件变大了"），记录下来并进入审查。

**NEEDS_CONTEXT：** 实现者需要未提供的信息。提供缺失的上下文并重新调度。

**BLOCKED：** 实现者无法完成任务。评估阻塞原因：
1. 如果是上下文问题，提供更多上下文并用相同模型重新调度
2. 如果任务需要更强的推理能力，用更强的模型重新调度
3. 如果任务太大，拆分成更小的部分
4. 如果计划本身有问题，上报给用户

**永远不要**忽略上报或在没有变更的情况下强制同一模型重试。如果实现者说它被卡住了，肯定有什么需要改变。

## 提示词模板

- `./references/implementer-prompt.md` - 调度实现者 subagent
- `./references/design-reviewer-prompt.md` - 调度设计合规审查 subagent
- `./references/code-quality-reviewer-prompt.md` - 调度代码质量审查 subagent

## 示例工作流

```
你：我将使用 subagent 驱动开发来执行此计划。

[一次性读取计划文件：docs/plans/feature-plan.md]
[提取所有 5 个任务的完整文本和上下文]
[创建包含所有任务的 TodoWrite]

任务 1：设计合规审查

[调度设计合规审查]
设计审查员：✅ 设计合规 - 所有需求满足，无多余内容

[获取 git SHA，调度代码质量审查]
代码审查员：优点：测试覆盖良好，代码整洁。问题：无。已批准。

[标记任务 1 完成]

任务 2：恢复模式

[获取任务 2 的文本和上下文（已提取）]
[调度实现 subagent，附完整任务文本 + 上下文]

实现者：[无问题，继续]
实现者：
  - 添加了 verify/repair 模式
  - 8/8 测试通过
  - 自我审查：一切正常
  - 已提交

[调度设计合规审查]
设计审查员：❌ 问题：
  - 缺失：进度报告（设计要求"每 100 项报告一次"）
  - 多余：添加了 --json 标志（未要求）

[实现者修复问题]
实现者：移除了 --json 标志，添加了进度报告

[设计审查员再次审查]
设计审查员：✅ 现在设计合规

[调度代码质量审查]
代码审查员：优点：扎实。问题（重要）：魔术数字（100）

[实现者修复]
实现者：提取了 PROGRESS_INTERVAL 常量

[代码审查员再次审查]
代码审查员：✅ 已批准

[标记任务 2 完成]

...

[所有任务完成后]
[调度最终代码审查]
最终审查员：所有需求满足，可以合并

完成！
```

## 优势

**vs. 手动执行：**
- subagent 自然遵循 TDD
- 每个任务全新的上下文（无混淆）
- 并行安全（ subagent 不互相干扰）
-  subagent 可以提问（工作前和工作期间）

**vs. 执行计划：**
- 同一会话（无交接）
- 持续进展（无需等待）
- 审查检查点自动进行

**效率提升：**
- 无文件读取开销（控制器提供完整文本）
- 控制器精心筛选所需的上下文
- subagent 前置获得完整信息
- 在工作开始前而非结束后发现问题

**质量关卡：**
- 自我审查在交接前捕获问题
- 两阶段审查：设计合规性，然后代码质量
- 审查循环确保修复确实有效
- 设计合规防止过度/不足构建
- 代码质量确保实现构建质量良好

**成本：**
- 更多 subagent 调用（每个任务需要实现者 + 2 个审查员）
- 控制器做更多准备工作（预先提取所有任务）
- 审查循环增加迭代次数
- 但在早期捕获问题（比后续调试成本更低）

## 红线

**永远不要：**
- 在没有用户明确同意的情况下在 main/master 分支上开始实现
- 跳过审查（设计合规或代码质量）
- 在有未修复问题时继续推进
- 并行调度多个实现 subagent（会冲突）
- 让 subagent 自己读计划文件（改为提供完整文本）
- 跳过场景设定上下文（ subagent 需要理解任务在哪里）
- 忽略 subagent 的问题（在让它们继续之前先回答）
- 在设计合规上接受"差不多"（设计审查员发现问题 = 未完成）
- 跳过审查循环（审查员发现问题 = 实现者修复 = 再次审查）
- 让实现者的自我审查替代正式审查（两者都需要）
- **在设计合规 ✅ 之前开始代码质量审查**（顺序错误）
- 在任一审查有未关闭问题时进入下一个任务

**如果 subagent 提问：**
- 清晰完整地回答
- 如需要提供额外上下文
- 不要催促它们进入实现

**如果审查员发现问题：**
- 实现者（同一 subagent）修复
- 审查员再次审查
- 重复直到批准
- 不要跳过重新审查

**如果 subagent 失败：**
- 调度修复 subagent 并提供具体指令
- 不要尝试手动修复（会污染上下文）

## 集成

**工作流技能：**
- **dev-kit:workspace** - 设置隔离工作区（基于 git worktree）
- **dev-kit:writing-plans** - 创建此技能执行的计划
- **dev-kit:executing-plans** - 替代方案：用于并行会话而非同会话执行

**sub agent 参考：**
- `./references/implementer-prompt.md` - 实现者提示词模板
- `./references/design-reviewer-prompt.md` - 设计合规审查提示词模板
- `./references/code-quality-reviewer-prompt.md` - 代码质量审查提示词模板
- `./references/code-reviewer.md` - 最终代码审查模板

**相关命令：**
- **dev-kit:code-reviewer** - 项目步骤完成后的代码审阅命令
