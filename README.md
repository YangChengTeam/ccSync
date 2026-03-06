# claude-config-sync

> Sync Claude Code hooks, commands, skills and more across projects — with auto PR.

---

## 为什么需要它

你在 A 项目里磨出了一套顺手的 Claude Code 配置——一个自动 lint 的 hook，一个生成 PR 的 command，一个专门处理代码审查的 sub-agent。然后你开始做 B 项目，这些经验全部归零。

**claude-config-sync 解决的正是这件事**：让好的配置可以流动，让经验可以迁移。

---

## 它同步什么

```
.claude/
  hooks/        PostToolUse / PreToolUse 等自动化脚本
  commands/     自定义 slash commands（.md）
  skills/       可复用的 skill 模块
  subagents/    sub-agent 定义
  mcp.json      MCP server 配置
```

---

## 安装

将 `sync-config.md` 放入 Claude Code 的 commands 目录：

```bash
# 用户级（所有项目可用）
cp sync-config.md ~/.claude/commands/

# 或项目级（仅当前项目）
cp sync-config.md .claude/commands/
```

---

## 用法

在任意项目中，通过 Claude Code 调用：

```
/sync-config <目标项目路径> [类型]
```

**类型**可选：`hooks` `commands` `skills` `subagents` `mcp` `all`（默认 `all`）

**示例：**

```bash
# 同步所有配置到 d 项目
/sync-config ~/projects/d

# 只同步 commands
/sync-config ~/projects/d commands

# 只同步 hooks 和 mcp
/sync-config ~/projects/d hooks
/sync-config ~/projects/d mcp
```

---

## 工作流程

```
扫描当前项目 .claude/
        ↓
检查目标项目（是否为 git 仓库）
        ↓
创建新分支 sync-claude-config-<timestamp>
        ↓
增量合并配置文件（不覆盖冲突项）
        ↓
自动生成说明 → 追加写入目标项目 CLAUDE.md
        ↓
调用目标项目 /commit-push-pr → 提交 PR
```

---

## 冲突处理

同步永远是**增量合并**，不破坏目标项目现有配置：

| 情况 | 处理方式 |
|------|----------|
| 同名 command / skill | 跳过，列入报告 |
| mcp.json key 冲突 | 保留目标项目的值 |
| 同名 hook 脚本 | 跳过并提示 |

同步结束后输出完整摘要：已复制文件、跳过文件、PR 链接。

---

## CLAUDE.md 自动文档

每次同步后，claude-config-sync 会在目标项目的 `CLAUDE.md` 中追加一个说明块：

```markdown
## 同步自 project-a — 2025-03-06

| 类型 | 名称 | 说明 |
|------|------|------|
| hook | post-edit-lint | 每次文件编辑后自动运行 ESLint |
| command | commit-push-pr | 生成 commit message 并开 PR |
| skill | code-review | 多维度代码审查 |
```

说明由 Claude 读取文件内容后自动生成——**同步的同时，知识也沉淀了**。

---

## 依赖

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)
- [GitHub CLI (`gh`)](https://cli.github.com)（PR 降级方案所需）

---

## License

MIT
