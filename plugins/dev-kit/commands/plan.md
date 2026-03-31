---
description: 编写细粒度的实施计划，包含具体文件路径、代码示例和验证步骤
---

# 编写实施计划

编写全面的实施计划，假设工程师对代码库零上下文且品味可疑。记录他们需要知道的一切：每个任务要修改哪些文件、代码、测试、可能需要查看的文档、如何测试。以细粒度任务的形式给出整个计划。DRY。YAGNI。

假设他们是熟练的开发者，但对我们的工具集或问题域几乎一无所知。假设他们不太了解良好的测试设计。

**用法：** `/plan [feature-name]`

**首先：** 根据设计文档/需求，使用代码搜索能力查找相关代码、术语定义和业务逻辑，定位需要修改的文件。

**开始时宣布：** "我正在创建实施计划。"

**保存计划到：** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## 细粒度任务粒度

**每个步骤是一个操作（2-5 分钟）：**
- "编写失败的测试" - 步骤
- "运行它确保失败" - 步骤
- "实现最小代码使测试通过" - 步骤
- "运行测试确保通过" - 步骤
- "提交" - 步骤

## 计划文档标题

**每个计划必须以这个标题开头：**

```markdown
# [功能名称] 实施计划

**目标：** [一句话描述这个构建什么]

**架构：** [2-3 句关于方法的描述]

**技术栈：** [关键技术/库]

---
```

## 任务结构

```markdown
### 任务 N: [组件名称]

**文件：**
- 创建：`exact/path/to/file.py`
- 修改：`exact/path/to/existing.py:123-145`
- 测试：`tests/exact/path/to/test.py`

**步骤 1: 编写失败的测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**步骤 2: 运行测试验证它失败**

运行：`pytest tests/path/test.py::test_name -v`
预期：失败，显示 "function not defined"

**步骤 3: 编写最小实现**

```python
def function(input):
    return expected
```

**步骤 4: 运行测试验证它通过**

运行：`pytest tests/path/test.py::test_name -v`
预期：通过
```

## 记住
- 始终使用精确的文件路径
- 计划中的完整代码（不是"添加验证"）
- 精确的命令和预期输出
- DRY, YAGNI

**参数：** $1
