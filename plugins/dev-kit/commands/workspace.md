---
description: 基于 git worktree 的独立工作区管理，支持创建和完成工作区
---

# 工作区管理

## 创建工作区

创建基于 git worktree 的独立工作区。

**用法：** `/workspace create [name]`

**步骤：**
1. 检测当前是否为 git 仓库
2. 确保工作目录干净（无未提交更改）
3. 检测主分支（main/master）
4. 更新主分支（除非使用 --skip-update）
5. 生成或使用提供的 workspace 名称
6. 创建 worktree 到 `.lp-commands-workspaces/{name}`
7. 保存元数据（原始分支、创建时间等）, 应该在 `.lp-commands-workspaces/.metadata` 中
8. 添加 /.lp-commands-workspaces 到 .git/info/exclude

**参数：**
- `name` (可选): 工作区名称，不提供则自动生成

**示例：**
```
/workspace create feature-auth
```

## 完成工作区

完成工作区，选择 merge 回原分支或创建 PR。

**用法：** `/workspace complete`

**步骤：**
1. 检测当前是否在仓库根目录（不能在 workspace 内执行）
2. 列出所有可用工作区
3. 如果多个工作区，让用户选择
4. 加载元数据获取原始分支
5. 提供选项：
   - Merge 回原分支
   - 创建 GitHub PR
6. 执行相应操作

**示例：**
```
/workspace complete
```
