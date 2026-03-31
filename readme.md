# Claude Code Source Mirror

> Unofficial, unsupported, and provided for research/documentation purposes only.
>
> This repository is not affiliated with Anthropic. For normal usage, installation, updates, and support, please use the official Claude Code product and documentation.

[中文主文档](./readme.md) | [English README](./README_EN.md)

---

### 目录

- [项目说明](#项目说明)
- [快速事实](#快速事实)
- [事件始末](#事件始末)
- [这份镜像里有什么](#这份镜像里有什么)
- [架构总览](#架构总览)
- [核心子系统](#核心子系统)
- [关键文件](#关键文件)
- [推荐阅读路径](#推荐阅读路径)
- [如何验证与探索](#如何验证与探索)
- [相关公开资料](#相关公开资料)
- [免责声明](#免责声明)

### 项目说明

这个仓库保存的是一份 **Claude Code 泄露的完整版客户端源码**。它来自公开发布的 npm 包中附带的 source map，可被恢复为完整的客户端源码树。

更准确地说：

- 公开 npm 包 `@anthropic-ai/claude-code@2.1.88` 中包含 `cli.js` 和 `cli.js.map`
- 我已直接从 npm registry 校验到该 tarball 确实存在，且 `cli.js.map` 文件大小约为 `59.8MB`
- 我进一步解包确认：`cli.js.map` 中包含 `4756` 个 `sources`，并且 `sourcesContent` 为完整非空内容
- 当前这个本地镜像仓库保存的是恢复后的 `src/` 源码树，适合做架构研究、功能分析、命令索引和产品考古

这意味着它可以被视为“**泄露出来的完整版客户端源码**”，但**不等于** Anthropic 内部私有 monorepo 的全部内容。

### 快速事实

| 项目                 | 内容                                                                                                 |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| 仓库性质             | 非官方泄露完整版客户端源码 / 研究用途                                                                |
| 对应公开包           | `@anthropic-ai/claude-code@2.1.88`                                                                   |
| 泄露载体             | npm 包内的 `cli.js.map`                                                                              |
| npm tarball 体积     | `31.2MB`                                                                                             |
| npm 解包体积         | `102.8MB`                                                                                            |
| tarball 文件数       | `20`                                                                                                 |
| 本地 `src/` 文件数   | `1902`                                                                                               |
| 本地 `ts/tsx` 文件数 | `1884`                                                                                               |
| 本地源码总行数       | `512,685`                                                                                            |
| 能看到的内容         | CLI、命令系统、工具系统、Ink/React TUI、MCP、插件、技能、Bridge、Remote、Voice、权限、遥测、特性开关 |
| 看不到的内容         | Claude 模型权重、训练数据、Anthropic 私有后端、完整内部 CI/运维资产                                  |

### 事件始末

把这件事按时间顺序拆开，大致是这样：

#### 第一阶段：Claude Code 先作为官方产品公开出现

- Anthropic 先公开推出了 Claude Code，并持续在官网、文档和产品能力说明中完善其定位
- 到 `2025-05-22` 的 `Introducing Claude 4` 公告中，Anthropic 已明确写到 Claude Code 已经 `generally available`
- 也就是说，在“源码事件”发生之前，Claude Code 本身已经是一个公开销售和公开文档化的官方产品，而不是地下项目

#### 第二阶段：公开 npm 包中附带 source map

- 社区随后发现，公开发布的 npm 包 `@anthropic-ai/claude-code` 中不仅有打包后的 `cli.js`
- 还包含了一个体积很大的 `cli.js.map`
- 对于前端/TypeScript 工具链来说，source map 原本是调试辅助文件；但如果其中携带完整 `sourcesContent`，就足以恢复出接近原始的源码树

#### 第三阶段：社区验证并恢复源码

- 这次事件的关键不在“猜测”，而在于可以直接验证
- 我已在本地确认 `@anthropic-ai/claude-code@2.1.88` 的 tarball 中确实存在 `cli.js.map`
- 我也进一步确认，这个 map 文件包含完整、非空的 `sourcesContent`
- 因此社区不需要逆向混淆代码，只需要把 source map 中的内容解包写回文件，就能重建出大规模可读的 TypeScript/TSX 源码

#### 第四阶段：镜像仓库、解读文章和讨论迅速扩散

- 一旦源码能被稳定恢复，传播速度就会非常快
- 社区很快出现了多个镜像仓库、目录梳理版 README、功能导览、命令清单和探索工具
- 中文社区因此产生了“Claude Code 开源了”“又双叒叕开源了”这类标题式传播
- 这些标题虽然抓眼球，但在技术和法律语义上并不精确

#### 第五阶段：真正值得关注的不是“八卦”，而是它暴露了多少产品细节

- 这次事件之所以引发大规模关注，不只是因为“看到了代码”
- 更因为这份代码量级足够大，覆盖了 Claude Code 客户端的核心组织方式
- 它让外界第一次以接近工程视角的方式，看到了 Anthropic 如何把 agent、工具调用、权限、MCP、IDE 集成、任务系统、团队协作和终端 UI 拼成一个完整产品

### 这份镜像里有什么

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

### 架构总览

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

### 核心子系统

#### 1. 工具系统

Claude Code 的“能力”大多以 Tool 的形式挂载。当前仓库中：

- `src/tools/` 下有 `42` 个一级工具目录
- `src/tools.ts` 负责工具注册与能力组合
- 工具覆盖文件读写、搜索、Web、任务、LSP、MCP、计划模式、工作树、Agent/Team 等场景

从工程角度看，这意味着 Claude Code 不是“一个大 prompt”，而是“一个能调度很多受约束工具的代理 runtime”。

#### 2. 命令系统

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

#### 3. 服务层

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

#### 4. Bridge 与 Remote

如果你对“Claude Code 如何接 IDE、远程会话、多端协作”感兴趣，这部分尤其值得看：

- `src/bridge/`: IDE bridge、消息协议、权限回调、会话桥接
- `src/remote/`: 远程会话与远端状态管理
- `src/server/`: 服务器/直连相关入口

这也是这份镜像最能体现产品化程度的地方之一，因为它显示出 Claude Code 并不只是本地 REPL，而是朝着 IDE、远程、协作、守护进程等方向延伸。

#### 5. 权限与配置

Claude Code 的另一个核心价值，不只是“会不会写代码”，而是“怎样安全地写代码”。从源码可见：

- `src/hooks/toolPermission/` 负责工具权限相关逻辑
- `src/schemas/` 负责配置和规则 schema
- `src/migrations/` 负责配置迁移
- `src/utils/settings/` 负责设置读取、校验与策略整合

这说明权限模型、配置模型和企业策略并不是附属功能，而是产品主干的一部分。

#### 6. 特性开关与内部代号

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

### 从源码里能看出什么

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

### 它不是什么

为避免误解，这个仓库**不是**以下内容：

- 不是 Claude 模型本身
- 不是训练代码、训练数据或推理权重
- 不是 Anthropic 全部后端服务代码
- 不是保证可直接构建、可直接发布的官方开发仓库
- 不是带官方支持的开源项目

更准确地说，它是“**从公开 npm 包 source map 中恢复出来的 Claude Code 泄露完整版客户端源码**”。

### 公开时间线

#### 2025-05-22

- Anthropic 在 `Introducing Claude 4` 公告中明确写到 Claude Code 已 `generally available`
- 这说明 Claude Code 作为产品本身早已公开，只是其源码并未以标准开源仓库方式主动发布

#### 2026-03-31

- 社区开始公开讨论 Claude Code npm 包中的 source map 暴露源码问题
- Chaofan Shou 在 X 上发布了最早一批广泛传播的公开链接之一：
  `https://x.com/Fried_rice/status/2038894956459290963?s=20`
- 我在本地直接校验到 `@anthropic-ai/claude-code@2.1.88` 的 tarball 内确实包含 `cli.js.map`
- 解包后的 `cli.js.map` 包含完整 `sourcesContent`，因此可以恢复出人类可读的 TypeScript/TSX 源码

### 关键文件

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

### 推荐阅读路径

如果你不是为了“看热闹”，而是想真正把这份代码读明白，下面这几条路径很高效：

#### 路径 1：先搞懂一个工具是怎么跑通的

- 先读 `src/Tool.ts`
- 再读一个具体工具目录，例如 `src/tools/BashTool/` 或 `src/tools/FileReadTool/`
- 最后回到 `src/QueryEngine.ts` 看工具是怎样被主循环调度的

#### 路径 2：先搞懂 Slash 命令系统

- 先读 `src/commands.ts`
- 再挑几个典型命令目录，如 `src/commands/review`、`src/commands/mcp`、`src/commands/tasks`
- 对比 PromptCommand、LocalCommand、LocalJSXCommand 这几类实现差异

#### 路径 3：先搞懂它为什么像一个“终端应用”而不只是“API 壳”

- 读 `src/main.tsx`
- 读 `src/screens/` 与 `src/components/`
- 读 `src/hooks/`、`src/state/`

这条路径最能帮助你理解为什么 Claude Code 的体验更接近一套完整 TUI，而不是普通 CLI 包装器。

#### 路径 4：先搞懂企业化和产品化能力

- 读 `src/services/mcp/`
- 读 `src/bridge/`
- 读 `src/services/plugins/`
- 读 `src/utils/settings/` 与 `src/hooks/toolPermission/`

这条路径最适合想研究“生产级 agent 产品”怎么做权限、集成和扩展性的人。

### 如何验证与探索

#### 1. 校验公开包

```bash
npm view @anthropic-ai/claude-code@2.1.88 version dist.tarball homepage
```

#### 2. 下载并检查 tarball

```bash
npm pack @anthropic-ai/claude-code@2.1.88
tar -xzf anthropic-ai-claude-code-2.1.88.tgz
ls package
```

你会看到至少这些关键文件：

- `package/cli.js`
- `package/cli.js.map`
- `package/package.json`

#### 3. 在当前镜像里做结构分析

```bash
rg --files src | wc -l
rg --files src -g '*.ts' -g '*.tsx' | wc -l
find src -maxdepth 1 -type d | sort
```

#### 4. 值得优先阅读的文件

- `src/main.tsx`
- `src/QueryEngine.ts`
- `src/tools.ts`
- `src/Tool.ts`
- `src/commands.ts`
- `src/services/mcp/`
- `src/bridge/`
- `src/plugins/`
- `src/skills/`

#### 5. 如果你想要更强的社区增强文档

`nirholas/claude-code` 的价值，不在于它“更原始”，而在于它做了更重的文档化整理，包括：

- 更完整的 README 导航
- `docs/` 下的架构、命令、工具、子系统导览
- 一个专门用于交互式源码探索的 MCP server

如果你的目标是：

- 保留当前仓库作为相对克制的源码镜像
- 同时把 `nirholas/claude-code` 作为增强版索引与阅读辅助

这种分工会比较合理。

### 相关公开资料

- Anthropic 官方产品页:
  <https://www.anthropic.com/claude-code>
- Anthropic 官方概览文档:
  <https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview>
- Anthropic 官方 GitHub 仓库:
  <https://github.com/anthropics/claude-code>
- Anthropic《Introducing Claude 4》:
  <https://www.anthropic.com/news/claude-4>
- Chaofan Shou on X:
  <https://x.com/Fried_rice/status/2038894956459290963?s=20>
- 社区增强镜像 `nirholas/claude-code`:
  <https://github.com/nirholas/claude-code>
- 2026-03-31 当天的 Reddit 讨论样本:
  - <https://www.reddit.com/r/ClaudeAI/comments/1s8lkkm/i_dug_through_claude_codes_leaked_source_and/>
  - <https://www.reddit.com/r/claude/comments/1s8p012/claude_code_source_code_has_been_leaked_via_a_map/>
  - <https://www.reddit.com/r/LocalLLaMA/comments/1s8ijfb/claude_code_source_code_has_been_leaked_via_a_map/>

### 免责声明

本仓库中的代码、命名、提示词、实现细节及相关知识产权归其原始权利人所有。此镜像 README 仅用于：

- 研究
- 文档整理
- 架构学习
- 安全与产品分析

如果你需要实际使用 Claude Code，请优先选择 Anthropic 的官方产品、官方文档和官方仓库。
