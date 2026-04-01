# Claude Code 学习路线图

[返回 README](../README.md)

---

本文档提供系统化的学习路径，帮助你从零开始理解 Claude Code 的完整架构。

## 🎯 学习目标设定

根据你的背景和目标，选择合适的学习路径：

| 背景               | 推荐路径 | 预计时间 |
| ------------------ | -------- | -------- |
| 初次接触           | 入门路径 | 2-3 天   |
| 有 TypeScript 经验 | 进阶路径 | 1-2 周   |
| 想深入特定领域     | 专题路径 | 按需     |

---

## 🌱 入门路径（2-3 天）

**目标：** 理解项目整体结构，能够运行和调试

### 第 1 天：环境搭建与基础认知

**上午（2 小时）**

1. ✅ 阅读根目录 [README.md](../README.md) 了解项目定位
2. ✅ 阅读 [EVENT_STORY.md](./EVENT_STORY.md) 了解背景
3. ✅ 安装 Bun 和依赖

**下午（2 小时）**

1. ✅ 运行 `bun run start --version`
2. ✅ 运行 `bun run start --help`
3. ✅ 启动交互界面，完成工作区信任
4. ✅ 阅读 [RUNNING_SETUP.md](./RUNNING_SETUP.md)

**验证标准：**

- [ ] 能够成功启动 Claude Code
- [ ] 理解项目来源和边界
- [ ] 知道文档的分工

### 第 2 天：代码结构探索

**上午（3 小时）**

1. ✅ 阅读 [SOURCE_STUDY_GUIDE.md](./SOURCE_STUDY_GUIDE.md) 的"架构总览"
2. ✅ 浏览 `src/` 目录结构
3. ✅ 阅读 `src/main.tsx` 前 200 行

**下午（3 小时）**

1. ✅ 阅读 `src/QueryEngine.ts` 的核心循环
2. ✅ 浏览 `src/tools/` 目录
3. ✅ 浏览 `src/commands/` 目录

**验证标准：**

- [ ] 能画出 Claude Code 的基本执行流程图
- [ ] 理解工具和命令的区别
- [ ] 知道主要子系统的位置

### 第 3 天：实践与总结

**上午（2 小时）**

1. ✅ 选择一个简单工具（如 `BashTool`）深入阅读
2. ✅ 追踪一个命令的完整执行路径

**下午（2 小时）**

1. ✅ 尝试修改一个简单功能（如帮助文本）
2. ✅ 运行并验证修改
3. ✅ 总结学习笔记

**验证标准：**

- [ ] 能够解释一个工具的完整实现
- [ ] 能够进行简单的代码修改
- [ ] 建立了自己的学习笔记

---

## 🚀 进阶路径（1-2 周）

**目标：** 深入理解核心机制，能够分析复杂功能

### 第 1 周：核心系统深入

#### Day 1-2: 工具系统

- 阅读 `src/Tool.ts` 完整实现
- 对比 3-5 个不同类型的工具
- 理解工具权限机制

**推荐阅读：**

- `src/tools/BashTool/`
- `src/tools/FileReadTool/`
- `src/tools/FileEditTool/`
- `src/hooks/toolPermission/`

**验证：** 能够实现一个简单的自定义工具

#### Day 3-4: 命令系统

- 阅读 `src/commands.ts` 注册机制
- 对比 `PromptCommand` 和 `LocalCommand`
- 理解命令的生命周期

**推荐阅读：**

- `src/commands/review/`
- `src/commands/mcp/`
- `src/commands/tasks/`

**验证：** 能够添加一个简单的自定义命令

#### Day 5-7: UI 与状态管理

- 理解 React + Ink 的终端 UI
- 阅读核心组件实现
- 理解状态管理机制

**推荐阅读：**

- `src/components/`
- `src/screens/`
- `src/state/`
- `src/hooks/`

**验证：** 能够修改 UI 布局或添加新组件

### 第 2 周：高级特性

#### Day 8-10: MCP 集成

- 理解 MCP 协议
- 阅读 MCP 客户端实现
- 研究资源和工具注册

**推荐阅读：**

- `src/services/mcp/`
- `src/commands/mcp/`

**验证：** 能够配置和使用 MCP 服务器

#### Day 11-12: Bridge 与 Remote

- 理解 IDE 集成机制
- 阅读远程会话实现
- 研究消息协议

**推荐阅读：**

- `src/bridge/`
- `src/remote/`

**验证：** 理解 Bridge 的通信流程

#### Day 13-14: 综合实践

- 选择一个复杂功能深入研究
- 编写技术文档
- 分享学习心得

---

## 🎓 专题路径（按需学习）

### 专题 1: Agent 架构

**时间：** 3-5 天

**学习内容：**

- Query Engine 的设计模式
- 工具调用循环
- 上下文管理
- 流式响应处理

**关键文件：**

- `src/QueryEngine.ts`
- `src/context.ts`
- `src/assistant/`

### 专题 2: 权限与安全

**时间：** 2-3 天

**学习内容：**

- 工具权限模型
- 工作区信任机制
- 策略限制
- 沙箱执行

**关键文件：**

- `src/hooks/toolPermission/`
- `src/services/policyLimits/`
- `src/utils/settings/`

### 专题 3: 插件系统

**时间：** 2-3 天

**学习内容：**

- 插件加载机制
- 技能系统
- Hooks 机制

**关键文件：**

- `src/plugins/`
- `src/skills/`
- `src/hooks/`

### 专题 4: 终端 UI

**时间：** 3-4 天

**学习内容：**

- Ink 框架使用
- 组件设计模式
- 交互处理
- 性能优化

**关键文件：**

- `src/ink/`
- `src/components/`
- `src/screens/`

---

## 📝 学习建议

### 有效的学习方法

1. **边读边记**
   - 记录关键概念
   - 画出架构图
   - 标注疑问点

2. **实践验证**
   - 运行代码验证理解
   - 修改代码观察效果
   - 使用调试工具

3. **对比学习**
   - 对比不同工具的实现
   - 对比不同命令的设计
   - 参考其他项目

4. **输出倒逼**
   - 写技术博客
   - 做分享演讲
   - 贡献文档

### 常见陷阱

❌ **不要：**

- 试图一次读完所有代码
- 陷入细节无法自拔
- 只看不练

✅ **应该：**

- 选择感兴趣的部分深入
- 保持全局视角
- 动手实践验证

---

## 🎯 学习检查清单

### 入门级（必须掌握）

- [ ] 能够运行和调试项目
- [ ] 理解基本架构和执行流程
- [ ] 知道主要子系统的位置和作用
- [ ] 能够阅读和理解简单模块

### 进阶级（建议掌握）

- [ ] 深入理解工具和命令系统
- [ ] 理解 UI 和状态管理
- [ ] 能够修改和扩展功能
- [ ] 理解 MCP 集成机制

### 高级（可选掌握）

- [ ] 理解 Bridge 和 Remote 实现
- [ ] 掌握权限和安全机制
- [ ] 能够设计新的子系统
- [ ] 能够贡献高质量代码

---

## 📚 推荐资源

### 相关技术栈

- [TypeScript 官方文档](https://www.typescriptlang.org/)
- [React 官方文档](https://react.dev/)
- [Ink 终端 UI 框架](https://github.com/vadimdemedes/ink)
- [Bun 运行时](https://bun.sh/)

### 相关概念

- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Language Server Protocol](https://microsoft.github.io/language-server-protocol/)
- [AI Agent 设计模式](https://www.anthropic.com/research)

---

## 💬 学习交流

- 在 [Issues](../../issues) 中提问
- 分享你的学习笔记
- 贡献改进建议

祝学习愉快！🎉
