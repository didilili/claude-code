# Claude Code 源码学习指南

[返回 README](../README.md) | [恢复与运行说明](./RUNNING_SETUP.md) | [事件始末说明](./EVENT_STORY.md) | [学习路线图](./LEARNING_PATH.md)

---

## 🎯 快速导航

**第一次读源码？** 推荐顺序：

1. 阅读本页”架构总览”建立全局认知
2. 查看”核心子系统”了解主要模块
3. 从”推荐阅读路径”选择一条深入
4. 配合 [LEARNING_PATH.md](./LEARNING_PATH.md) 系统学习

**想快速定位？** 直接跳转：

- [架构总览](#架构总览)
- [核心子系统](#核心子系统)
- [关键文件](#关键文件)
- [推荐阅读路径](#推荐阅读路径)

---

## 📖 关于本文档

这份文档专门服务于「如何系统学习这份源码」，把根目录 `README.md` 里的源码学习部分独立出来，避免首页过长。

它的目标不是替你做源码分析结论，而是帮你更快建立阅读顺序、结构感和重点入口。

## 这份文档适合谁

更适合下面几类读者：

- 想把这份仓库当作 Claude Code 客户端结构样本来读的人
- 想研究 agent CLI、工具系统、命令系统、MCP、权限和终端 UI 的开发者
- 想做分享、写文章、做课程或做二次整理的人

如果你现在最关心的是“项目怎么启动”，建议先去看 [RUNNING_SETUP.md](./RUNNING_SETUP.md)。

## 建议怎么读

第一次进源码时，比较推荐按这个顺序：

1. 先看 [架构总览](#架构总览)，建立整体地图
2. 再看 [核心子系统](#核心子系统)，理解 Claude Code 为什么不是单一 CLI 壳
3. 再从 [关键文件](#关键文件) 和 [推荐阅读路径](#推荐阅读路径) 里挑一条线深入
4. 如果要做事实校验或二次整理，最后参考 [如何验证与探索](#如何验证与探索)

## 这份镜像里有什么

从当前仓库结构看，这份泄露的完整版客户端源码覆盖了 Claude Code 的主要客户端层：

```text
src/
├── main.tsx         # 主入口，CLI 启动与会话初始化
├── QueryEngine.ts   # 主查询/编排逻辑
├── Tool.ts          # Tool 类型与协议
├── tools.ts         # 内置工具注册表
├── commands.ts      # Slash 命令注册表
├── cli/             # CLI 处理逻辑
├── commands/        # 内置命令
├── tools/           # 文件/搜索/执行/代理等工具
├── components/      # React + Ink 终端 UI 组件
├── services/        # API、MCP、analytics、settings、sync 等服务
├── bridge/          # IDE / Remote Bridge 相关逻辑
├── remote/          # 远程会话
├── plugins/         # 插件系统
├── skills/          # 技能系统
├── voice/           # 语音相关功能
├── state/           # 应用状态管理
└── ...
```

几个关键入口文件的规模也能反映这不是一个简单壳子：

- `src/main.tsx`: `4683` 行
- `src/QueryEngine.ts`: `1295` 行
- `src/Tool.ts`: `792` 行
- `src/commands.ts`: `754` 行
- `src/tools.ts`: `389` 行

## 架构总览

如果把这份代码当作一个“终端里的 agent 产品”来看，它的大体执行链路可以概括为：

```text
用户输入
  -> CLI 解析与启动
  -> REPL / 会话初始化
  -> Query Engine
  -> Anthropic API / 流式响应
  -> Tool 调用循环
  -> 终端 UI 渲染
```

也就是说，这不是一个简单的“命令行包壳”，而是一套完整的终端原生应用：

- 启动入口在 `src/main.tsx`
- 会话主循环核心在 `src/QueryEngine.ts`
- UI 由 `React + Ink` 驱动
- 工具调用依赖 `src/Tool.ts` 与 `src/tools/`
- Slash 命令依赖 `src/commands.ts` 与 `src/commands/`
- API、MCP、OAuth、策略限制、插件、同步等基础设施则集中在 `src/services/`

## 核心子系统

### 1. 工具系统

Claude Code 的“能力”大多以 Tool 的形式挂载。当前仓库中：

- `src/tools/` 下有 `42` 个一级工具目录
- `src/tools.ts` 负责工具注册与能力组合
- 工具覆盖文件读写、搜索、Web、任务、LSP、MCP、计划模式、工作树、Agent/Team 等场景

从工程角度看，这意味着 Claude Code 不是“一个大 prompt”，而是“一个能调度很多受约束工具的代理 runtime”。

### 2. 命令系统

Slash 命令构成了用户操作面。当前仓库中：

- `src/commands/` 下有 `86` 个一级命令目录
- 同时还有 `15` 个根级命令文件
- `src/commands.ts` 是命令注册表与入口索引

从源码里可以直接看到的高频命令包括：

- `/review`
- `/commit`
- `/mcp`
- `/memory`
- `/tasks`
- `/permissions`
- `/resume`
- `/doctor`
- `/diff`
- `/skills`

### 3. 服务层

`src/services/` 是这份代码最值得研究的部分之一。当前镜像中能看到至少这些重要服务域：

- `api/`: API client、bootstrap、文件相关接口
- `mcp/`: MCP 连接、注册、配置与资源访问
- `oauth/`: 登录与身份认证
- `analytics/`: 埋点、特性开关、遥测
- `plugins/`: 插件系统
- `lsp/`: 语言服务协议集成
- `policyLimits/`: 组织/策略限制
- `remoteManagedSettings/`: 远程托管设置
- `settingsSync/` 与 `teamMemorySync/`: 同步能力
- `tools/`: 工具执行与编排辅助

### 4. Bridge 与 Remote

如果你对“Claude Code 如何接 IDE、远程会话、多端协作”感兴趣，这部分尤其值得看：

- `src/bridge/`: IDE bridge、消息协议、权限回调、会话桥接
- `src/remote/`: 远程会话与远端状态管理
- `src/server/`: 服务器/直连相关入口

这也是这份镜像最能体现产品化程度的地方之一，因为它显示出 Claude Code 并不只是本地 REPL，而是朝着 IDE、远程、协作、守护进程等方向延伸。

### 5. 权限与配置

Claude Code 的另一个核心价值，不只是“会不会写代码”，而是“怎样安全地写代码”。从源码可见：

- `src/hooks/toolPermission/` 负责工具权限相关逻辑
- `src/schemas/` 负责配置和规则 schema
- `src/migrations/` 负责配置迁移
- `src/utils/settings/` 负责设置读取、校验与策略整合

这说明权限模型、配置模型和企业策略并不是附属功能，而是产品主干的一部分。

### 6. 特性开关与内部代号

源码大量使用 `bun:bundle` 的 `feature()` 做编译期特性裁剪，常见例子包括：

- `KAIROS`
- `PROACTIVE`
- `BRIDGE_MODE`
- `VOICE_MODE`
- `COORDINATOR_MODE`
- `WORKFLOW_SCRIPTS`
- `DIRECT_CONNECT`
- `SSH_REMOTE`

这类开关非常适合拿来研究产品分层、实验功能和构建裁剪策略。

## 从源码里能看出什么

这份代码至少展示了 Claude Code 客户端的这些能力与方向：

- 完整的命令行代理工作流：读文件、改文件、执行 shell、检索代码、管理会话
- 终端 UI 使用 `React + Ink`
- 明确的工具注册体系，内含文件、搜索、Web、任务、MCP、计划模式、Agent/Team 等工具
- 明确的 Slash 命令体系，覆盖 `review`、`mcp`、`permissions`、`tasks`、`plugins`、`resume`、`status` 等
- MCP 集成相当深，相关代码遍布 `services/mcp/`
- 存在 Bridge / Remote / Direct Connect / SSH Remote / IDE 集成相关代码
- 有插件、技能、hooks、worktree、team/swarm、memory、teleport 等扩展机制
- 有语音、遥测、权限、策略限制、托管设置、远程设置同步等系统级能力

另外，源码中还能看到不少特性开关或内部代号，例如：

- `KAIROS`
- `ULTRAPLAN`
- `COORDINATOR_MODE`
- `BRIDGE_MODE`
- `DIRECT_CONNECT`
- `SSH_REMOTE`
- `WEB_BROWSER_TOOL`
- `WORKFLOW_SCRIPTS`
- `BUDDY`

这些名字说明产品路线曾覆盖比公开表面更广的实验功能，但**代号存在并不等于功能已正式发布**。

## 它不是什么

为避免误解，这个仓库**不是**以下内容：

- 不是 Claude 模型本身
- 不是训练代码、训练数据或推理权重
- 不是 Anthropic 全部后端服务代码
- 不是保证可直接构建、可直接发布的官方开发仓库
- 不是带官方支持的开源项目

更准确地说，它是“**从公开 npm 包 source map 中恢复出来的 Claude Code 泄露完整版客户端源码**”。

## 关键文件

如果你准备系统读这份代码，可以优先从这些文件或目录进入：

| 路径                   | 作用                                         |
| ---------------------- | -------------------------------------------- |
| `src/main.tsx`         | CLI 启动入口，参数解析、启动优化、会话初始化 |
| `src/QueryEngine.ts`   | 主查询循环、流式响应、工具调用回路           |
| `src/Tool.ts`          | Tool 类型系统、约束与协议                    |
| `src/tools.ts`         | 工具注册表                                   |
| `src/commands.ts`      | 命令注册表                                   |
| `src/context.ts`       | 系统/用户上下文采集                          |
| `src/replLauncher.tsx` | REPL 启动与交互入口                          |
| `src/services/mcp/`    | MCP 的主实现                                 |
| `src/bridge/`          | IDE 与 Bridge 相关实现                       |
| `src/state/`           | 全局状态与变更观察                           |

## 推荐阅读路径

如果你不是为了“看热闹”，而是想真正把这份代码读明白，下面这几条路径很高效：

### 路径 1：先搞懂一个工具是怎么跑通的

- 先读 `src/Tool.ts`
- 再读一个具体工具目录，例如 `src/tools/BashTool/` 或 `src/tools/FileReadTool/`
- 最后回到 `src/QueryEngine.ts` 看工具是怎样被主循环调度的

### 路径 2：先搞懂 Slash 命令系统

- 先读 `src/commands.ts`
- 再挑几个典型命令目录，如 `src/commands/review`、`src/commands/mcp`、`src/commands/tasks`
- 对比 `PromptCommand`、`LocalCommand`、`LocalJSXCommand` 这几类实现差异

### 路径 3：先搞懂它为什么像一个“终端应用”而不只是“API 壳”

- 读 `src/main.tsx`
- 读 `src/screens/` 与 `src/components/`
- 读 `src/hooks/`、`src/state/`

这条路径最能帮助你理解为什么 Claude Code 的体验更接近一套完整 TUI，而不是普通 CLI 包装器。

### 路径 4：先搞懂企业化和产品化能力

- 读 `src/services/mcp/`
- 读 `src/bridge/`
- 读 `src/services/plugins/`
- 读 `src/utils/settings/` 与 `src/hooks/toolPermission/`

这条路径最适合想研究“生产级 agent 产品”怎么做权限、集成和扩展性的人。

## 如何验证与探索

### 1. 校验公开包

```bash
npm view @anthropic-ai/claude-code@2.1.88 version dist.tarball homepage
```

### 2. 下载并检查 tarball

```bash
npm pack @anthropic-ai/claude-code@2.1.88
tar -xzf anthropic-ai-claude-code-2.1.88.tgz
ls package
```

你会看到至少这些关键文件：

- `package/cli.js`
- `package/cli.js.map`
- `package/package.json`

### 3. 在当前镜像里做结构分析

```bash
rg --files src | wc -l
rg --files src -g '*.ts' -g '*.tsx' | wc -l
find src -maxdepth 1 -type d | sort
```

### 4. 值得优先阅读的文件

- `src/main.tsx`
- `src/QueryEngine.ts`
- `src/tools.ts`
- `src/Tool.ts`
- `src/commands.ts`
- `src/services/mcp/`
- `src/bridge/`
- `src/plugins/`
- `src/skills/`

### 5. 如果你想要更强的社区增强文档

`nirholas/claude-code` 的价值，不在于它“更原始”，而在于它做了更重的文档化整理，包括：

- 更完整的 README 导航
- `docs/` 下的架构、命令、工具、子系统导览
- 一个专门用于交互式源码探索的 MCP server

如果你的目标是：

- 保留当前仓库作为相对克制的源码镜像
- 同时把 `nirholas/claude-code` 作为增强版索引与阅读辅助

这种分工会比较合理。

### 6. 如果你想参考其他社区仓库的整理思路

这几个社区仓库的定位差异也值得一起看：

- `chatgptprojects/claude-code`：更偏“提取结果展示”，适合快速理解事件是什么
- `pengchengneo/Claude-Code`：更偏“可运行 + 展示型 README”，适合看中文语境下如何包装成可读仓库
- `nirholas/claude-code`：更偏“文档化索引仓库”，适合参考 README 分层和 `docs/` 组织方式

如果你面对的是中文读者，一个比较稳妥的做法就是像当前仓库这样分层：

- `README.md`（根目录）：完整首页导航、快速开始与文档索引
- `docs/RUNNING_SETUP.md`：工程恢复与运行
- `docs/SOURCE_STUDY_GUIDE.md`：源码学习（本文档）
- `docs/EVENT_STORY.md`：事件背景
