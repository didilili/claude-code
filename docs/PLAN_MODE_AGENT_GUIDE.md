# 计划模式与 Agent 系统深度解析

> 深入分析 Claude Code 的计划模式（Plan Mode）和多 Agent 协作系统

## 目录

- [1. 计划模式详解](#1-计划模式详解)
- [2. Agent 系统架构](#2-agent-系统架构)
- [3. Agent 定义与配置](#3-agent-定义与配置)
- [4. 工具限制机制](#4-工具限制机制)

---

## 1. 计划模式详解

### 1.1 什么是计划模式

计划模式是 Claude Code 的核心功能之一，用于处理复杂的实现任务。

**核心理念**：
- 先规划，后执行
- 获取用户审批
- 避免浪费工作

### 1.2 何时使用计划模式

从 `EnterPlanModeTool/prompt.ts` 提取的 7 个使用场景：

**1. 新功能实现**
```
示例："Add a logout button"
问题：放在哪里？点击后做什么？
```

**2. 多种实现方案**
```
示例："Add caching to the API"
选择：Redis / in-memory / file-based
```

**3. 代码修改**
```
示例："Update the login flow"
问题：具体改什么？
```

**4. 架构决策**
```
示例："Add real-time updates"
选择：WebSockets vs SSE vs polling
```

**5. 多文件变更**
```
示例："Refactor the authentication system"
影响：多个文件需要协调修改
```

**6. 需求不明确**
```
示例："Make the app faster"
需要：先分析性能瓶颈
```

**7. 用户偏好重要**
```
如果需要用 AskUserQuestion，不如用 EnterPlanMode
计划模式可以先探索，再展示选项
```

### 1.3 何时不用计划模式

**跳过计划模式的场景**：
- 单行或少量代码修改
- 明显的 bug 修复
- 用户给出详细指令
- 纯研究任务（用 Agent 工具）

### 1.4 计划模式工作流

**6 个步骤**：

1. **探索代码库**：使用 Glob、Grep、Read 工具
2. **理解现有模式**：分析架构和设计
3. **设计实现方案**：制定详细计划
4. **展示给用户**：等待审批
5. **澄清疑问**：使用 AskUserQuestion
6. **退出计划模式**：用 ExitPlanMode 开始实现

### 1.5 计划模式面试阶段

从代码中发现的隐藏功能：

```typescript
const whatHappens = isPlanModeInterviewPhaseEnabled()
  ? ''
  : WHAT_HAPPENS_SECTION
```

**功能**：
- 启用后，工作流指令通过 attachment 传递
- 减少 Prompt 长度
- 动态调整流程

---

## 2. Agent 系统架构

### 2.1 Agent 定义结构

从 `AgentTool/prompt.ts` 提取的核心结构：

```typescript
export type AgentDefinition = {
  agentType: string        // Agent 类型
  whenToUse: string        // 使用场景描述
  tools?: string[]         // 允许的工具列表
  disallowedTools?: string[]  // 禁止的工具列表
}
```

### 2.2 工具访问控制

**三种模式**：

**1. 允许列表（Allowlist）**
```typescript
tools: ['FileRead', 'Grep', 'Glob']
// 输出：FileRead, Grep, Glob
```

**2. 禁止列表（Denylist）**
```typescript
disallowedTools: ['Bash', 'FileWrite']
// 输出：All tools except Bash, FileWrite
```

**3. 混合模式**
```typescript
tools: ['FileRead', 'Bash', 'Grep']
disallowedTools: ['Bash']
// 输出：FileRead, Grep（过滤掉 Bash）
```

### 2.3 Agent 列表注入优化

**性能优化发现**：

从代码注释中发现的关键优化：

```typescript
/**
 * The dynamic agent list was ~10.2% of fleet cache_creation tokens: MCP async
 * connect, /reload-plugins, or permission-mode changes mutate the list →
 * description changes → full tool-schema cache bust.
 */
```

**问题**：
- Agent 列表动态变化
- 导致工具描述变化
- 触发完整缓存失效
- 占用 10.2% 的缓存创建 token

**解决方案**：
- 将 Agent 列表作为 attachment 注入
- 而非嵌入工具描述
- 避免频繁缓存失效

### 2.4 Fork Subagent 功能

从代码中发现的实验性功能：

```typescript
const forkEnabled = isForkSubagentEnabled()
```

**功能**：
- 允许 Agent "fork"（分叉）
- 创建独立的执行上下文
- 支持并行工作流

---

## 3. Agent 定义与配置

### 3.1 Agent 配置文件

**位置**：`.kiro/agents/`

**示例结构**：
```markdown
---
agentType: context-gatherer
whenToUse: Starting work on an unfamiliar codebase
tools: [Glob, Grep, Read]
---

# Context Gatherer Agent

This agent explores the codebase and gathers relevant context.
```

### 3.2 内置 Agent 类型

**已发现的 Agent 类型**：
- `general-purpose`: 通用任务
- `statusline-setup`: 状态栏配置
- `claude-code-guide`: 文档查询
- `context-gatherer`: 上下文收集

---

## 4. 工具限制机制

### 4.1 为什么需要工具限制

**原因**：
- 防止 Agent 滥用权限
- 限制破坏性操作
- 提高安全性

### 4.2 实现细节

```typescript
function getToolsDescription(agent: AgentDefinition): string {
  const { tools, disallowedTools } = agent
  const hasAllowlist = tools && tools.length > 0
  const hasDenylist = disallowedTools && disallowedTools.length > 0

  if (hasAllowlist && hasDenylist) {
    const denySet = new Set(disallowedTools)
    const effectiveTools = tools.filter(t => !denySet.has(t))
    return effectiveTools.join(', ')
  }
  // ...
}
```

### 4.3 最佳实践

**推荐配置**：
- 只读 Agent：`tools: ['Read', 'Glob', 'Grep']`
- 分析 Agent：`disallowedTools: ['Bash', 'FileWrite']`
- 全功能 Agent：不设置限制

---

## 参考资料

- [ARCHITECTURE_DEEP_DIVE.md](./ARCHITECTURE_DEEP_DIVE.md)
- [TOOLS_SYSTEM_GUIDE.md](./TOOLS_SYSTEM_GUIDE.md)

