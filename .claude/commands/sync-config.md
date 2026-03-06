---
description: 将当前项目的 Claude Code 配置（hooks/mcp/commands/skills/subagents）同步到目标项目并提交 PR
allowed-tools: Bash(find:*), Bash(ls:*), Bash(cat:*), Bash(cp:*), Bash(mkdir:*), Bash(git:*), Read, Write
argument-hint: <target-project-path> [hooks|mcp|commands|skills|subagents|all]
---

## 任务

将当前项目的 Claude Code 配置同步到目标项目，并调用目标项目的 `commit-push-pr` 命令提交 PR。

## 参数解析

- `$1`：目标项目路径（必填），如 `~/projects/d`
- `$2`：同步类型（可选，默认 `all`）：`hooks` / `mcp` / `commands` / `skills` / `subagents` / `all`

## 执行步骤

### Step 1 — 扫描当前项目配置

扫描当前项目 `.claude/` 目录，列出所有可同步的内容：

```
.claude/
  hooks/          → PostToolUse / PreToolUse 等 hook 脚本
  commands/       → 自定义 slash commands（.md 文件）
  skills/         → skills（每个子目录含 SKILL.md）
  subagents/      → sub-agent 定义文件
  mcp.json        → MCP server 配置
  settings.json   → 项目级设置
```

对每个找到的文件/目录，读取内容并生成简短说明（用于写入目标项目的 CLAUDE.md 注释块）。

### Step 2 — 检查目标项目

确认目标路径 `$1` 存在且是 git 仓库：

```bash
ls "$1"
git -C "$1" status
```

若目标路径不存在或不是 git 仓库，终止并提示用户。

### Step 3 — 创建同步分支

在目标项目创建新分支：

```bash
git -C "$1" checkout -b sync-claude-config-$(date +%Y%m%d%H%M%S)
```

### Step 4 — 复制配置文件

根据 `$2` 参数决定同步范围：

**hooks**：复制 `.claude/hooks/` → `$1/.claude/hooks/`（合并，不覆盖同名文件）

**commands**：复制 `.claude/commands/*.md` → `$1/.claude/commands/`

**skills**：复制 `.claude/skills/` 下各目录 → `$1/.claude/skills/`

**subagents**：复制 `.claude/subagents/` → `$1/.claude/subagents/`

**mcp**：读取当前 `.claude/mcp.json`，与目标项目 mcp.json 合并（key 不重复），写回目标

```bash
mkdir -p "$1/.claude/{hooks,commands,skills,subagents}"
```

### Step 5 — 更新目标项目 CLAUDE.md

在目标项目 CLAUDE.md（不存在则创建）追加说明块：

```markdown
## 同步自 <当前项目名> — <日期>

| 类型 | 名称 | 说明 |
|------|------|------|
| hook | post-edit-lint | 每次文件编辑后自动运行 lint |
| command | commit-push-pr | 提交并推送，自动开 PR |
| skill | ... | ... |
```

每一行说明由 Claude 根据读取到的文件内容自动生成，不要填占位符。

### Step 6 — 调用目标项目的 commit-push-pr

切换到目标项目目录，触发其自定义命令：

```bash
cd "$1" && claude --dangerously-skip-permissions "/project:commit-push-pr sync: add Claude Code configs from $(basename $PWD)"
```

若目标项目没有 `commit-push-pr` 命令，则退而使用标准流程：

```bash
git -C "$1" add .claude/ CLAUDE.md
git -C "$1" commit -m "sync: add Claude Code configs from $(basename $(pwd))"
git -C "$1" push -u origin HEAD
gh pr create --title "sync: Claude Code configs" --body "Auto-synced from $(basename $(pwd)) on $(date)"
```

### Step 7 — 输出摘要

同步完成后列出：
- 已复制的文件清单
- 跳过的文件（冲突/已存在）
- PR 链接

---

## 冲突处理原则

- **同名 command/skill**：跳过，列入报告，由用户手动决定
- **mcp.json key 冲突**：保留目标项目的值，不覆盖
- **hooks 同名脚本**：跳过并提示

目标是**增量合并**，永不破坏目标项目现有配置。
