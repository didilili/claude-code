# Claude Codes 恢复与运行说明

[返回 README](../README.md) | [源码学习指南](./SOURCE_STUDY_GUIDE.md) | [事件始末说明](./EVENT_STORY.md) | [FAQ](./FAQ.md)

---

## ⚡ 三分钟快速启动

**只想快速跑起来？** 执行以下命令：

```bash
# 1. 安装 Bun（如果还没有）
curl -fsSL https://bun.sh/install | bash
exec $SHELL

# 2. 进入仓库，在与 package.json 同级目录安装依赖
#    （本仓库的 package.json 当前在 docs/ 下时请：cd docs）
cd /path/to/claude-codes
bun install

# 3. 启动（仍在上述目录）
bun run start
```

**首次启动会看到：**

- 工作区信任确认页面（选择”信任”继续）
- 可能需要 API Key（见下方说明）

**遇到问题？** 查看 [FAQ.md](./FAQ.md) 或继续阅读本文档。

---

## 📖 关于本文档

这份文档专门说明：为了让当前 `claude-codes` 仓库能够真正跑起来，我做了哪些补齐、修复和配置。

它不是源码解析文档，而是一份偏工程视角的”恢复记录 + 运行手册”。

## 这份文档解决什么问题

读完这份文档，你应该能知道：

- 这个仓库原始缺了哪些东西
- 我为了让它可运行补了哪些依赖、文件和配置
- 哪些位置做了兼容修复
- 现在应该怎样安装、验证和启动
- 它已经能跑到什么程度，还有哪些边界

## 目标

当前仓库最初只有一份恢复出来的 `src/` 源码树和说明文档，但并不具备一个完整可运行仓库通常需要的这些东西：

- 包管理配置
- 运行脚本
- TypeScript 配置
- 依赖锁文件
- 本地 shim 包
- 某些恢复过程中缺失的源码文件
- 启动时的少量兼容修复

这份说明的目的，就是把“这个仓库为什么现在能启动”讲清楚，也让后来接手这个仓库的人能快速复现当前状态。

## 我补了哪些文件

为了把这个仓库从”源码镜像”补成”可运行工作区”，我新增或补齐了下面这些内容：

### 核心配置文件
- `docs/package.json` - 依赖列表和运行脚本（当前位于 `docs/`）
- `docs/bun.lock` - 固定依赖解析结果
- `tsconfig.json` - TypeScript 编译配置（仓库根目录）

### 格式化和编辑器配置
- `.prettierrc.json` - Prettier 格式化规则
- `.prettierignore` - 格式化忽略文件
- `.editorconfig` - 编辑器统一配置
- `.vscode/settings.json` - VS Code 工作区配置

### 本地 shim 包（7 个）
- `shims/ant-claude-for-chrome-mcp/` - Chrome MCP 兼容层
- `shims/ant-computer-use-input/` - 计算机使用输入兼容层
- `shims/ant-computer-use-mcp/` - 计算机使用 MCP 兼容层
- `shims/ant-computer-use-swift/` - Swift 计算机使用兼容层
- `shims/color-diff-napi/` - 颜色差异 NAPI 模块
- `shims/modifiers-napi/` - 修饰键 NAPI 模块
- `shims/url-handler-napi/` - URL 处理 NAPI 模块

### 运行时依赖
- `docs/image-processor.node` - 图像处理本地二进制模块（当前与 `docs/package.json` 同区）

### 开发入口
- `src/dev-entry.ts` - 适合恢复版源码的开发入口

### 补齐的源码文件
恢复过程中缺失的源码文件，包括但不限于：
- `src/assistant/index.ts`
- `src/assistant/sessionDiscovery.ts`
- `src/bridge/peerSessions.ts`
- `src/ink/global.ts` - 修复 Ink 全局模块导入
- 以及其他约 50+ 个缺失的 TypeScript 文件

### 文档文件（均在 `docs/` 目录）
- `README.md`（仓库根目录）- 项目主页与文档导航
- `docs/RUNNING_SETUP.md` - 本文档
- `docs/SOURCE_STUDY_GUIDE.md` - 源码学习指南
- `docs/EVENT_STORY.md` - 事件始末
- `docs/FAQ.md` - 常见问题
- `docs/LEARNING_PATH.md` - 学习路线图
- （可选）其他深度文档：`docs/ARCHITECTURE_DEEP_DIVE.md`、`docs/PROMPTS_ANALYSIS.md` 等

这些内容的作用分别是：

### 核心配置文件
- `docs/package.json` - 提供依赖列表和运行脚本，定义项目元信息
- `docs/bun.lock` - 固定依赖解析结果，减少不同机器之间的安装漂移
- `tsconfig.json`（根目录）- 让 TypeScript/Bun 能按恢复源码的结构正确解析模块

### 格式化和编辑器配置
- `.prettierrc.json` - 统一代码格式化规则（单引号、不加分号、2 空格等）
- `.prettierignore` - 指定不需要格式化的文件和目录
- `.editorconfig` - 跨编辑器的基础配置（换行符、缩进等）
- `.vscode/settings.json` - VS Code 特定配置，自动格式化和规则约束

### 本地 shim 包
- 补上恢复版源码依赖的本地包占位和兼容实现
- 这些包在原始 npm 包中是内置的，恢复后需要手动补齐
- 每个 shim 包都包含 `package.json` 和基础实现文件

### 运行时依赖
- `docs/image-processor.node` - 运行时需要的本地二进制模块（图像处理）
- 这是一个占位文件，实际功能可能需要真实的二进制模块

### 开发入口
- `src/dev-entry.ts` - 提供一个更适合恢复版源码工作区的开发入口
- 处理宏定义、环境变量和启动配置

### 补齐的源码文件
- 解决恢复出来的源码树存在的相对导入缺口
- 包括 assistant、bridge、ink、components 等多个子系统的缺失文件
- 特别是 `src/ink/global.ts` 用于修复 Ink 全局模块导入问题

## 依赖与运行时配置

当前仓库已经按 `Bun` 作为主运行时配置完成。

### 运行时要求

- `bun >= 1.3.5`
- 推荐系统：macOS / Linux
- Node.js 可保留，但主运行入口不是 Node，而是 Bun

### 根目录 README 与 `docs/` 的分工

完整项目说明与文档导航在仓库根目录 **[README.md](../README.md)**；运行细节、源码学习、事件背景等专题在 **`docs/`** 下各 Markdown 文件。

```bash
cd /Users/tools/Music/cc/claude-codes
bun install
bun run start
```

下面这些更完整的运行背景、启动说明和格式化细节，都统一放在这份文档里维护。

### 我添加的核心脚本

`package.json` 中目前提供了这些最关键的脚本：

```json
{
  "dev": "bun run ./src/dev-entry.ts",
  "start": "bun run ./src/dev-entry.ts",
  "version": "bun run ./src/dev-entry.ts --version",
  "format": "prettier . --write --ignore-unknown",
  "format:check": "prettier . --check --ignore-unknown",
  "format:ts": "prettier \"**/*.{ts,tsx,cts,mts}\" --write",
  "format:check:ts": "prettier \"**/*.{ts,tsx,cts,mts}\" --check"
}
```

## 我做的关键兼容修复

恢复版源码能“装上依赖”不代表能“真的启动”。为了让它进到真实启动流程，我额外做了几处兼容修复。

### 1. 修复 Ink 全局模块导入

涉及文件：

- `src/ink/components/Box.tsx`
- `src/ink/components/ScrollBox.tsx`
- `src/ink/global.ts`

问题是恢复版源码里引用了一个并不适合直接作为运行时模块导入的全局声明文件，导致运行期模块解析不稳定。

我做的处理是：

- 把组件里的导入改为 `../global.js`
- 新增一个最小运行时模块 `src/ink/global.ts`

这个文件内容非常简单：

```ts
export {}
```

它的作用只是让运行时导入链成立，而不去改变原本的类型声明逻辑。

### 2. 修复 Commander 参数兼容问题

涉及文件：

- `src/main.tsx`

恢复版源码里存在一个 `-d2e` 的调试别名。这个写法在当前 Commander 组合下会带来参数注册兼容问题。

我做了两件事：

- 在启动早期把 `-d2e` 重写为 `--debug-to-stderr`
- 把 Commander 里对应的非法短选项定义改为只保留长选项

这样做的结果是：

- 不会因为参数定义异常而直接崩掉
- 旧别名仍然能兼容

### 3. 补齐缺失导入链

在最开始扫描仓库导入关系时，这个仓库存在大量缺失导入目标，导致很多模块并不能真正解析。

恢复过程里，我补齐了缺失的：

- `src/` 子模块
- 本地 shim 包
- 运行时资源目录

补完后，缺失导入数量从大量缺口降到了可解析状态，仓库才能进入真正的启动链路。

## 我增加的格式化配置

后面又继续补了 TypeScript 格式化和编辑器侧的约束，所以我额外增加了：

- `prettier`
- `.prettierrc.json`
- `.prettierignore`
- `.editorconfig`
- `.vscode/settings.json`

当前仓库的实际格式化约定是：

- 单引号
- 不加分号
- 2 空格缩进
- 不使用 Tab
- `LF` 换行
- 多行结构保留 trailing comma
- 单参数箭头函数尽量不加括号

对应配置大致是：

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all",
  "arrowParens": "avoid",
  "tabWidth": 2,
  "useTabs": false,
  "endOfLine": "lf",
  "proseWrap": "preserve"
}
```

我还已经实际跑过一轮：

```bash
bun run format:ts
bun run format:check:ts
```

目前检查是通过的。

## TypeScript 风格补充说明

这里顺手把之前独立文档里的 TypeScript 风格说明也合并进来，避免仓库里再多一份单独的规范文件。

### 原始风格参考

我之前对当前仓库的 TypeScript 文件做过全量统计，目的是理解“这批恢复源码原本更像什么风格”。

统计理解上分成两层：

- 原始已跟踪 TypeScript 文件
- 当前工作区中的全部 TypeScript 文件

其中：

- Git 已跟踪的 `ts/tsx/cts/mts` 文件：`1884`
- 当前工作区中可见的 `ts/tsx/cts/mts` 文件总数：`1988`

原始恢复源码的主流特征更接近：

- 单引号
- 不加分号
- 2 空格缩进
- 不使用 Tab
- `LF` 换行
- 单参数箭头函数尽量不加括号
- 多行结构允许并保留 trailing comma

也就是说：

- 原始风格参考：更接近”无分号”
- 当前实际项目约定：保持原始风格”不加分号”

### 当前项目约定

现在仓库里真正生效的格式化规则是：

- 单引号
- 不加分号
- 2 空格缩进
- 不使用 Tab
- `LF` 换行
- 单参数箭头函数尽量不加括号
- 多行结构保留 trailing comma

如果只用一句话概括当前项目约定，那就是：

`单引号 + 不加分号 + 2 空格 + LF + 单参数箭头尽量不加括号`

### VS Code 侧约束

我还补了 `.vscode/settings.json`，目的不是再发明一套规则，而是让编辑器保存时尽量自动遵循仓库当前约定。

它主要做了这些事：

- 把默认格式化器锁到 `Prettier`
- 打开 `formatOnSave`
- 关闭 TypeScript/JavaScript 内置格式化器，避免和 Prettier 打架
- 明确写入 `2 空格`、`LF`、`单引号`、`不加分号` 这些核心约束

如果你使用 VS Code，建议确认本机已经安装：

- `Prettier - Code formatter`

### 实际使用方式

最常用的命令就是：

```bash
cd /Users/tools/Music/cc/claude-codes

# 格式化全部文件
bun run format

# 只格式化 TypeScript
bun run format:ts

# 检查 TypeScript 是否符合当前约定
bun run format:check:ts
```

### 边界说明

需要明确的一点是：

- 这批源码并不是“百分之百单一风格”的人工精修代码库
- 它是从 source map 恢复出来的真实工程源码
- 因此局部仍然会存在一些历史遗留差异

所以这里的目标不是保留每个文件所有历史细碎差异，而是：

- 理解它原始的主流代码风格
- 在当前仓库里执行一套稳定、一致、可维护的格式化约定
- 让后续修改和现有代码尽量保持同一种工程气质

## 我如何验证它已经能跑

我不是只补了文件，没有验证。我已经在当前机器上实际做过下面这些验证：

### 1. 依赖安装验证

执行：

```bash
bun install
```

结果：

- 依赖可以正常解析和安装
- 本地 `shims/` 包可被正确接入

### 2. CLI 版本验证

执行：

```bash
bun run start --version
```

结果：

- 能正确输出版本号

### 3. 帮助命令验证

执行：

```bash
bun run start --help
```

结果：

- 能正确输出帮助信息

### 4. 真实启动验证

执行：

```bash
bun run start
```

结果：

- 已经可以真正拉起 Claude Code 的交互式界面
- 首次会进入工作区信任确认页

这说明当前仓库不是“只会打印个 help”，而是已经能进入实际 UI 启动链路。

## 现在如何运行

### 第一次安装

```bash
curl -fsSL https://bun.sh/install | bash
exec /bin/zsh

cd /Users/tools/Music/cc/claude-codes
bun install
```

### 已验证可运行的命令

下面这些命令我已经在当前机器上实际验证过：

```bash
cd /Users/tools/Music/cc/claude-codes

# 查看版本
bun run start --version

# 查看帮助
bun run start --help

# 启动交互式 TUI
bun run start
```

当前验证结果是：

- `bun run start --version` 可以正常返回版本号
- `bun run start --help` 可以正常输出帮助
- `bun run start` 已经可以真正拉起 Claude Code 的交互界面，并显示工作区信任确认页

### 启动交互界面

```bash
cd /Users/tools/Music/cc/claude-codes
bun run start
```

### 第一次启动会发生什么

首次执行：

```bash
bun run start
```

通常会先看到工作区安全确认，大意是：

- 是否信任当前目录
- Claude Code 将能够读取、编辑、执行这个目录下的内容

确认信任后，后续才会继续进入会话、认证或模型调用流程。

### 查看版本

```bash
cd /Users/tools/Music/cc/claude-codes
bun run start --version
```

### 查看帮助

```bash
cd /Users/tools/Music/cc/claude-codes
bun run start --help
```

### 非交互最小测试

```bash
cd /Users/tools/Music/cc/claude-codes
export ANTHROPIC_API_KEY=your_key_here
bun run start --bare --print "hello"
```

### 为什么这里推荐 `--bare`

这份仓库是“恢复版源码工作区”，不是 Anthropic 官方完整开发环境。`--bare` 模式会跳过一部分外围初始化，更适合：

- 验证核心 CLI 是否能启动
- 减少不必要的插件、钩子和同步逻辑干扰
- 做本地调试和源码研究

### 一组最实用的命令

```bash
cd /Users/tools/Music/cc/claude-codes

# 安装依赖
bun install

# 查看是否装好
bun run start --version
bun run start --help

# 启动交互界面
bun run start

# 非交互最小测试
ANTHROPIC_API_KEY=your_key_here bun run start --bare --print "hello"
```

## 当前边界

虽然这个仓库现在已经可以启动，但仍然要明确几个边界：

- 这不是 Anthropic 官方完整开发仓库
- 它是恢复后的客户端源码工作区
- 一些功能仍依赖真实的 Anthropic 账号、API 或在线服务
- 某些边缘路径后面仍可能继续暴露出需要补 shim 或兼容修复的问题

换句话说：

- 现在已经达到“项目能安装、能启动、能进真实 UI”
- 但还不等于“完整复刻 Anthropic 内部生产环境”

## 建议阅读顺序

如果你来这里是为了：

- 先跑起来：先看本文件里的 [现在如何运行](#现在如何运行)
- 进一步理解源码结构：继续看 [SOURCE_STUDY_GUIDE.md](./SOURCE_STUDY_GUIDE.md)
- 了解事件背景和传播过程：继续看 [EVENT_STORY.md](./EVENT_STORY.md)

## 建议后续维护方式

如果后面你还要继续把这个仓库往“长期可维护”方向推进，比较建议继续做这些事：

- 补一份更完整的 `.gitignore`
- 继续验证 `--print`、`--bare`、认证链路和更多命令路径
- 把剩余恢复版兼容修复整理成单独变更记录

## 相关文件

如果你要继续追踪这些恢复工作，最值得先看的就是这些文件：

- `README.md`（根目录）
- `docs/package.json`
- `.prettierrc.json`
- `.prettierignore`
- `.editorconfig`
- `.vscode/settings.json`
- `src/main.tsx`
- `src/ink/components/Box.tsx`
- `src/ink/components/ScrollBox.tsx`
- `src/ink/global.ts`
