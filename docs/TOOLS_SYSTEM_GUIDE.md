# Claude Code 工具系统详解

> 本文档深入分析 Claude Code 的 54 个工具实现，包括设计模式、核心功能和使用场景

## 目录

- [1. 工具系统概览](#1-工具系统概览)
- [2. 文件操作工具](#2-文件操作工具)
- [3. 命令执行工具](#3-命令执行工具)
- [4. Agent 协作工具](#4-agent-协作工具)
- [5. MCP 集成工具](#5-mcp-集成工具)
- [6. 交互式工具](#6-交互式工具)
- [7. 工具设计模式](#7-工具设计模式)

---

## 1. 工具系统概览

### 1.1 工具分类统计

Claude Code 共有 **54 个工具**，分为以下类别：

| 类别 | 数量 | 核心工具 |
|------|------|---------|
| **文件操作** | 5 | FileRead, FileWrite, FileEdit, Glob, Grep |
| **命令执行** | 3 | Bash, PowerShell, REPL |
| **Agent 系统** | 6 | Agent, SendMessage, Task* |
| **MCP 协议** | 4 | MCP, ListMcpResources, ReadMcpResource, McpAuth |
| **计划模式** | 3 | EnterPlanMode, ExitPlanMode, VerifyPlanExecution |
| **Worktree** | 2 | EnterWorktree, ExitWorktree |
| **Web 功能** | 3 | WebSearch, WebFetch, WebBrowser |
| **交互** | 2 | AskUserQuestion, ReviewArtifact |
| **配置** | 2 | Config, LSP |
| **其他** | 24 | Notebook, Skill, Monitor, Brief 等 |

### 1.2 工具基类架构

所有工具继承自 `Tool.ts` 基类：

```typescript
export interface ToolDef {
  name: string                    // 工具名称
  description: string             // 工具描述（Prompt）
  input_schema: ToolInputJSONSchema  // 参数 Schema
  validate?: (input) => ValidationResult  // 参数验证
  execute: (context) => Promise<ToolResultBlockParam>  // 执行逻辑
}
```

**核心特性**：
- 统一的参数验证
- 权限检查集成
- 进度报告机制
- 错误处理标准化

---

## 2. 文件操作工具

### 2.1 FileReadTool

**功能**：读取文件内容

**关键特性**：
- 支持行范围读取（offset + limit）
- 自动检测文件编码
- 支持 PDF 文件（需指定页码）
- 支持 Jupyter Notebook
- 图片文件可视化展示

**使用场景**：
```typescript
{
  file_path: "/path/to/file.ts",
  offset: 0,      // 可选：起始行
  limit: 100      // 可选：读取行数
}
```

### 2.2 FileEditTool

**功能**：精确替换文件内容

**关键特性**：
- 基于字符串匹配的替换
- 支持 `replace_all` 批量替换
- 自动保留文件编码和换行符
- 文件历史追踪

**安全机制**：
- 必须先用 Read 读取文件
- `old_string` 必须唯一（否则报错）
- 自动检测缩进格式

### 2.3 FileWriteTool

**功能**：创建或覆盖文件

**限制**：
- 必须先读取已存在的文件
- 优先使用 Edit 而非 Write
- 不自动创建文档文件

### 2.4 GlobTool

**功能**：文件模式匹配

**特性**：
- 支持 glob 模式（`**/*.ts`）
- 按修改时间排序
- 替代 `find` 和 `ls`

### 2.5 GrepTool

**功能**：内容搜索

**特性**：
- 基于 ripgrep
- 支持正则表达式
- 三种输出模式：content / files_with_matches / count
- 支持上下文行（-A/-B/-C）
- 支持多行匹配

---

## 3. 命令执行工具

### 3.1 BashTool

**功能**：执行 Shell 命令

**核心特性**：
- 后台任务支持（`run_in_background`）
- 超时控制（默认 120s，最大 600s）
- 沙箱模式
- AST 安全解析
- Git 操作追踪
- Sed 命令自动转换为 FileEdit

**安全机制**：
- 权限检查
- 危险命令检测
- 只读模式验证

### 3.2 PowerShellTool

**功能**：Windows PowerShell 执行

**特性**：
- 类似 BashTool
- Windows 平台专用

### 3.3 REPLTool

**功能**：交互式代码执行

**支持语言**：
- Python
- Node.js
- Ruby
- 其他 REPL 环境

---

## 4. Agent 协作工具

### 4.1 AgentTool

**功能**：启动子 Agent

**参数**：
- `description`: 任务描述（3-5 词）
- `prompt`: 详细任务说明
- `subagent_type`: Agent 类型
- `run_in_background`: 后台运行
- `isolation`: worktree 隔离

**Agent 类型**：
- `general-purpose`: 通用 Agent
- `statusline-setup`: 状态栏配置
- `claude-code-guide`: 文档查询

### 4.2 TaskCreateTool / TaskUpdateTool / TaskGetTool / TaskListTool

**功能**：任务管理系统

**任务状态**：
- `pending`: 待处理
- `in_progress`: 进行中
- `completed`: 已完成
- `deleted`: 已删除

### 4.3 SendMessageTool

**功能**：Agent 间通信

**用途**：向其他 Agent 发送消息

---

## 5. MCP 集成工具

### 5.1 MCPTool

**功能**：调用 MCP 服务器工具

**特性**：
- 动态工具发现
- OAuth 认证支持
- 跨应用访问（XAA）

### 5.2 ListMcpResourcesTool

**功能**：列出 MCP 资源

### 5.3 ReadMcpResourceTool

**功能**：读取 MCP 资源内容

### 5.4 McpAuthTool

**功能**：处理 MCP OAuth 认证

---

## 6. 交互式工具

### 6.1 AskUserQuestionTool

**功能**：向用户提问

**特性**：
- 支持单选/多选
- 支持预览（代码片段、UI 模拟）
- 1-4 个问题
- 2-4 个选项

### 6.2 EnterPlanModeTool / ExitPlanModeTool

**功能**：计划模式管理

**用途**：
- 复杂任务规划
- 用户审批流程

### 6.3 EnterWorktreeTool / ExitWorktreeTool

**功能**：Git Worktree 隔离

**用途**：
- 独立工作环境
- 避免污染主分支

---

## 7. 工具设计模式

### 7.1 权限检查模式

所有工具都实现统一的权限检查：

```typescript
export type PermissionResult = {
  allowed: boolean
  reason?: string
  requiresConfirmation?: boolean
}
```

### 7.2 进度报告模式

长时间运行的工具支持进度报告：

```typescript
export type ToolProgressData = 
  | BashProgress
  | AgentToolProgress
  | WebSearchProgress
  | MCPProgress
```

### 7.3 后台任务模式

支持后台运行的工具：
- BashTool
- AgentTool
- REPLTool

### 7.4 UI 组件模式

每个工具可以有独立的 UI 组件：
- `UI.tsx`: 工具结果渲染
- `prompt.ts`: 工具描述 Prompt
- `constants.ts`: 常量定义

---

## 8. 完整工具列表

### 8.1 按字母排序

1. AgentTool - 启动子 Agent
2. AskUserQuestionTool - 向用户提问
3. BashTool - 执行 Shell 命令
4. BriefTool - 简报生成
5. ConfigTool - 配置管理
6. DiscoverSkillsTool - 发现技能
7. EnterPlanModeTool - 进入计划模式
8. EnterWorktreeTool - 进入 Worktree
9. ExitPlanModeTool - 退出计划模式
10. ExitWorktreeTool - 退出 Worktree
11. FileEditTool - 编辑文件
12. FileReadTool - 读取文件
13. FileWriteTool - 写入文件
14. GlobTool - 文件模式匹配
15. GrepTool - 内容搜索
16. ListMcpResourcesTool - 列出 MCP 资源
17. LSPTool - 语言服务器协议
18. McpAuthTool - MCP 认证
19. MCPTool - MCP 工具调用
20. MonitorTool - 监控工具
21. NotebookEditTool - Jupyter Notebook 编辑
22. OverflowTestTool - 溢出测试
23. PowerShellTool - PowerShell 执行
24. ReadMcpResourceTool - 读取 MCP 资源
25. RemoteTriggerTool - 远程触发
26. REPLTool - REPL 执行
27. ReviewArtifactTool - 审查产物
28. ScheduleCronTool - 定时任务
29. SendMessageTool - 发送消息
30. SendUserFileTool - 发送文件给用户
31. SkillTool - 技能执行
32. SleepTool - 睡眠等待
33. SnipTool - 代码片段
34. SyntheticOutputTool - 合成输出
35. TaskCreateTool - 创建任务
36. TaskGetTool - 获取任务
37. TaskListTool - 列出任务
38. TaskOutputTool - 任务输出
39. TaskStopTool - 停止任务
40. TaskUpdateTool - 更新任务
41. TeamCreateTool - 创建团队
42. TeamDeleteTool - 删除团队
43. TerminalCaptureTool - 终端捕获
44. TodoWriteTool - 待办事项
45. ToolSearchTool - 工具搜索
46. TungstenTool - Tungsten 工具
47. VerifyPlanExecutionTool - 验证计划执行
48. WebBrowserTool - Web 浏览器
49. WebFetchTool - Web 抓取
50. WebSearchTool - Web 搜索
51. WorkflowTool - 工作流

---

## 参考资料

- [ARCHITECTURE_DEEP_DIVE.md](./ARCHITECTURE_DEEP_DIVE.md) - 架构深度解析
- [PROMPTS_ANALYSIS.md](./PROMPTS_ANALYSIS.md) - Prompt 系统分析

