---
allowed-tools: Bash(git log:*), Bash(mkdir -p:*), Read(*), Write(*)
description: 从 Git 提交自动生成工作日志（日报/周报）
---

## 重要
所有的都要使用中文

## Context

- 当前日期: !`powershell -Command "Get-Date -Format 'yyyy-MM-dd'"`
- 当前目录: !`pwd`

## Your task

从 Git 提交记录自动生成项目工作日志。

### 判断报告类型

根据 ARGUMENTS 参数判断：
- 如果 ARGUMENTS 是 "周报" 或 "week" → 生成周报
- 否则 → 生成日报

### 操作步骤：

1. **根据报告类型获取 Git 提交**：
   - **日报**：`git log --all --since=-1day --pretty=format:"%ad|%s" --date=short`
   - **周报**：`git log --all --since=-7days --pretty=format:"%ad|%s" --date=short`

2. **推断项目名称**（按优先级）：
   - 从 `package.json` 的 `name` 字段
   - 从 `go.mod` 的模块名
   - 从 `README.md` 的标题
   - 当前目录名

3. **生成格式**：

   **日报格式**：
   ```markdown
   # [项目名称] 日报 - 2026-03-06

   ## 今日完成
   - 提交信息1
   - 提交信息2

   ---
   *生成时间：2026-03-06*
   ```
   如果今天没有提交，显示"## 今日完成

   今日暂无代码提交"

   **周报格式**：
   ```markdown
   # [项目名称] 周报 - 2026-02-28 至 2026-03-06

   ## 本周完成

   ### 2026-03-06
   - 提交信息1 

   ### 2026-03-05
   - 提交信息2


   ---
   *生成时间：2026-03-06*
   ```

4. **保存路径**：
   - **日报**：`works/日报/日报-[日期].md`
   - **周报**：`works/周报/周报-[日期].md`

5. **输出摘要**：显示报告类型、生成结果和文件路径
