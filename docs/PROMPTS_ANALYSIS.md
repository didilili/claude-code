# Claude Code Prompt 系统深度解析

> 本文档提取并分析 Claude Code 中的所有关键 Prompt，包括系统 Prompt、工具 Prompt、安全规则等

## 目录

- [1. Prompt 系统概览](#1-prompt-系统概览)
- [2. BashTool Prompt 详解](#2-bashtool-prompt-详解)
- [3. Git 操作 Prompt](#3-git-操作-prompt)
- [4. 安全规则 Prompt](#4-安全规则-prompt)
- [5. 其他工具 Prompt](#5-其他工具-prompt)

---

## 1. Prompt 系统概览

### 1.1 Prompt 架构

Claude Code 的 Prompt 系统采用分层设计：

```
System Prompt (全局)
    ↓
Tool Prompts (工具级)
    ↓
Context Prompts (上下文)
    ↓
User Message (用户输入)
```

### 1.2 Prompt 文件组织

每个工具都有独立的 `prompt.ts` 文件：

```
src/tools/
├── BashTool/prompt.ts
├── FileEditTool/prompt.ts
├── AgentTool/prompt.ts
└── ...
```

---

## 2. BashTool Prompt 详解

### 2.1 后台任务提示

```
You can use the `run_in_background` parameter to run the command in the background. 
Only use this if you don't need the result immediately and are OK being notified 
when the command completes later. You do not need to check the output right away - 
you'll be notified when it finishes. You do not need to use '&' at the end of the 
command when using this parameter.
```

**设计意图**：
- 引导 AI 使用后台任务功能
- 避免 AI 使用 `&` 符号（会导致问题）
- 明确异步通知机制

---

## 3. Git 操作 Prompt

### 3.1 Git 安全协议

从 `BashTool/prompt.ts` 第 87-94 行提取的核心安全规则：

```
Git Safety Protocol:
- NEVER update the git config
- NEVER run destructive git commands (push --force, reset --hard, checkout ., 
  restore ., clean -f, branch -D) unless the user explicitly requests these actions
- NEVER skip hooks (--no-verify, --no-gpg-sign, etc) unless the user explicitly requests it
- NEVER run force push to main/master, warn the user if they request it
- CRITICAL: Always create NEW commits rather than amending, unless the user explicitly 
  requests a git amend
- When staging files, prefer adding specific files by name rather than using 
  "git add -A" or "git add ."
- NEVER commit changes unless the user explicitly asks you to
```

**关键洞察**：
1. **防止数据丢失**：禁止破坏性操作
2. **保护 Hook**：不跳过 pre-commit 等钩子
3. **避免 amend 陷阱**：Hook 失败后 amend 会修改错误的 commit
4. **防止泄密**：避免 `git add -A` 意外提交敏感文件

### 3.2 Commit 工作流

完整的 4 步 Commit 流程（第 96-110 行）：

**步骤 1：并行收集信息**
```bash
# 并行执行
git status              # 查看未跟踪文件
git diff               # 查看变更
git log                # 查看提交历史
```

**步骤 2：分析并起草 Commit Message**
- 总结变更性质（新功能/增强/修复/重构）
- 检查敏感文件
- 1-2 句话描述"为什么"而非"是什么"

**步骤 3：执行 Commit**
```bash
git add <specific-files>
git commit -m "$(cat <<'EOF'
Commit message here.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
git status  # 验证成功
```

**步骤 4：Hook 失败处理**
- 修复问题
- 创建新 commit（不是 amend！）

### 3.3 Pull Request 工作流

从第 130-150 行提取的 PR 创建流程：

**步骤 1：收集分支信息**
```bash
git status
git diff
git log
git diff [base-branch]...HEAD  # 查看完整变更
```

**步骤 2：起草 PR**
- 标题 < 70 字符
- 详细信息放在 body

**步骤 3：创建 PR**
```bash
gh pr create --title "the pr title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 3.4 Undercover 模式

从第 48-51 行发现的隐藏功能：

```typescript
const undercoverSection =
  process.env.USER_TYPE === 'ant' && isUndercover()
    ? getUndercoverInstructions() + '\n'
    : ''
```

**用途**：Anthropic 内部员工使用时，隐藏内部代号和模型 ID，避免在公开 commit 中泄露。

---

## 4. 安全规则 Prompt

### 4.1 工具使用限制

从 Bash Prompt 中提取的关键限制：

```
- NEVER run additional commands to read or explore code, besides git bash commands
- NEVER use the TodoWrite or Agent tools (during commit)
- DO NOT push to the remote repository unless the user explicitly asks
- IMPORTANT: Never use git commands with the -i flag (interactive not supported)
- IMPORTANT: Do not use --no-edit with git rebase
```

### 4.2 文件安全

```
- Do not commit files that likely contain secrets (.env, credentials.json, etc)
- Warn the user if they specifically request to commit those files
- When staging files, prefer adding specific files by name
```

### 4.3 HEREDOC 格式要求

```
In order to ensure good formatting, ALWAYS pass the commit message via a HEREDOC:

git commit -m "$(cat <<'EOF'
   Commit message here.
   
   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
   EOF
   )"
```

**原因**：避免 shell 转义问题，确保多行消息正确格式化。

---

## 5. 其他工具 Prompt

### 5.1 工具引用规则

从 Bash Prompt 中发现的工具引用模式：

```
- Use ${FILE_READ_TOOL_NAME} instead of cat/head/tail
- Use ${FILE_EDIT_TOOL_NAME} instead of sed/awk
- Use ${FILE_WRITE_TOOL_NAME} instead of echo/cat
- Use ${GLOB_TOOL_NAME} instead of find/ls
- Use ${GREP_TOOL_NAME} instead of grep/rg
```

**设计意图**：
- 提供更好的用户体验
- 便于权限控制
- 统一的错误处理

### 5.2 Anthropic 内部 vs 外部用户

代码中发现两套不同的 Prompt：

**内部用户（USER_TYPE === 'ant'）**：
```
For git commits and pull requests, use the `/commit` and `/commit-push-pr` skills:
- `/commit` - Create a git commit with staged changes
- `/commit-push-pr` - Commit, push, and create a pull request

Before creating a pull request, run `/simplify` to review your changes
```

**外部用户**：
- 完整的内联指令
- 详细的步骤说明
- 包含 Co-Authored-By 归属

---

## 6. Prompt 设计模式总结

### 6.1 防御性编程

所有 Prompt 都采用防御性设计：

1. **明确禁止**：使用 NEVER、DO NOT 等强烈词汇
2. **重复强调**：关键规则多次重复
3. **提供示例**：用 `<example>` 标签展示正确用法
4. **解释原因**：说明为什么要这样做

### 6.2 条件化 Prompt

根据环境动态生成 Prompt：

```typescript
// 根据用户类型
if (process.env.USER_TYPE === 'ant') { ... }

// 根据功能开关
if (shouldIncludeGitInstructions()) { ... }

// 根据沙箱状态
if (SandboxManager.isEnabled()) { ... }
```

### 6.3 Prompt 组合

Prompt 通过函数组合构建：

```typescript
export function getSimplePrompt() {
  return [
    getBackgroundUsageNote(),
    getCommitAndPRInstructions(),
    getSandboxInstructions(),
    // ...
  ].filter(Boolean).join('\n\n')
}
```

---

## 7. 关键发现

### 7.1 隐藏的 26 个指令

根据公众号文章提到的"26 个隐藏指令"，这些指令分布在：

- Git 安全协议（8 条）
- 工具使用限制（6 条）
- 文件操作规则（5 条）
- 并行执行指令（4 条）
- 其他（3 条）

### 7.2 自修复机制

Prompt 中内置自修复逻辑：

```
4. If the commit fails due to pre-commit hook: fix the issue and create a NEW commit
```

这是一个简单但有效的自修复指令。

### 7.3 归属机制

代码中发现的归属文本生成：

```typescript
const { commit: commitAttribution, pr: prAttribution } = getAttributionTexts()
```

用于在 commit 和 PR 中添加 "Co-Authored-By" 或生成标记。

---

## 8. 学习价值

### 8.1 Prompt 工程最佳实践

1. **结构化**：使用标题、列表、示例
2. **明确性**：避免模糊表述
3. **防御性**：预防常见错误
4. **条件化**：根据上下文调整

### 8.2 可复用的 Prompt 模式

- Git 安全协议可用于任何 Git 工具
- HEREDOC 格式可用于任何多行输入
- 并行执行模式可用于任何批量操作

---

## 参考资料

- [ARCHITECTURE_DEEP_DIVE.md](./ARCHITECTURE_DEEP_DIVE.md) - 架构深度解析
- [SOURCE_STUDY_GUIDE.md](./SOURCE_STUDY_GUIDE.md) - 源码学习指南

