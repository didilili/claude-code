# Claude Code 源码深度剖析：Prompt 工程、自修复与隐藏特性

> 基于源码分析，深度解析 Claude Code 的 Prompt 系统、自修复机制和隐藏功能

## 目录

- [1. Prompt 工程最佳实践](#1-prompt-工程最佳实践)
- [2. 自修复机制详解](#2-自修复机制详解)
- [3. 隐藏功能与彩蛋](#3-隐藏功能与彩蛋)
- [4. 代码质量分析](#4-代码质量分析)
- [5. 实战技巧](#5-实战技巧)

---

## 1. Prompt 工程最佳实践

### 1.1 防御性 Prompt 设计

从 BashTool 的 Prompt 中可以学到的防御性设计模式：

**模式 1：使用强烈否定词**
```
NEVER update the git config
NEVER run destructive git commands
NEVER skip hooks
```

**为什么有效**：
- 明确的禁止比建议更有效
- 重复强调关键规则
- 使用大写增强视觉冲击

**模式 2：解释原因**
```
CRITICAL: Always create NEW commits rather than amending, unless the user 
explicitly requests a git amend. When a pre-commit hook fails, the commit 
did NOT happen — so --amend would modify the PREVIOUS commit, which may 
result in destroying work or losing previous changes.
```

**为什么有效**：
- AI 理解"为什么"后更不容易犯错
- 提供上下文帮助判断边界情况

**模式 3：提供具体示例**
```xml
<example>
git commit -m "$(cat <<'EOF'
   Commit message here.
   
   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
   EOF
   )"
</example>
```

**为什么有效**：
- 消除歧义
- 展示正确格式
- 可直接复制使用

### 1.2 条件化 Prompt 技巧

**根据用户类型动态生成**：
```typescript
if (process.env.USER_TYPE === 'ant') {
  // Anthropic 内部员工：简洁版，引用 skills
  return "Use /commit and /commit-push-pr skills"
} else {
  // 外部用户：完整指令
  return "Follow these 4 steps..."
}
```

**学习价值**：
- 不同用户需要不同详细程度
- 内部用户可以依赖约定
- 外部用户需要完整指导

---

## 2. 自修复机制详解

### 2.1 Hook 失败自修复

**Prompt 中的自修复指令**：
```
4. If the commit fails due to pre-commit hook: fix the issue and create a NEW commit
```

**实现原理**：
1. 检测 Hook 失败
2. 分析失败原因
3. 自动修复（如格式化代码）
4. 重新提交（新 commit，不是 amend）

**为什么不用 amend**：
- Hook 失败意味着 commit 没有创建
- amend 会修改上一个 commit
- 可能导致数据丢失

### 2.2 Sed 命令自动转换

**位置**：`BashTool/sedEditParser.ts`

**功能**：
- 检测 sed 编辑命令
- 自动转换为 FileEdit 工具
- 提供更好的用户体验

**示例**：
```bash
sed -i 's/old/new/g' file.txt
# 自动转换为
FileEdit(file_path="file.txt", old_string="old", new_string="new", replace_all=true)
```

### 2.3 错误降级策略

**工具失败时的降级**：
- 优先使用专用工具（FileEdit）
- 失败后降级到 Bash（sed）
- 最后提示用户手动操作

---

## 3. 隐藏功能与彩蛋

### 3.1 Undercover 模式

**位置**：`utils/undercover.js`

**触发条件**：
```typescript
process.env.USER_TYPE === 'ant' && isUndercover()
```

**功能**：
- 隐藏内部代号（如 "Kairos"）
- 隐藏模型 ID
- 剥离归属信息
- 防止在公开 commit 中泄露内部信息

**实现**：
```typescript
const undercoverSection = isUndercover() 
  ? getUndercoverInstructions() + '\n'
  : ''
```

### 3.2 Feature Flags 系统

**已发现的 Feature Flags**：

| Flag | 功能 | 状态 |
|------|------|------|
| `PROACTIVE` | 主动建议 | 条件编译 |
| `BRIDGE_MODE` | Bridge 远程控制 | 条件编译 |
| `VOICE_MODE` | 语音交互 | 条件编译 |
| `KAIROS` | Kairos 项目 | 条件编译 |
| `WORKFLOW_SCRIPTS` | 工作流脚本 | 条件编译 |
| `HISTORY_SNIP` | 历史截断 | 条件编译 |

**使用方式**：
```typescript
const proactive = feature('PROACTIVE') 
  ? require('./commands/proactive.js').default 
  : null
```

### 3.3 内部命令

**Anthropic 内部专用命令**：
- `/agents-platform` - Agent 平台管理
- `/bughunter` - Bug 追踪
- `/simplify` - 代码简化审查
- `/security-review` - 安全审查

---

## 4. 代码质量分析

### 4.1 优秀的设计

**1. 启动性能优化**
- 并行预加载（MDM + Keychain）
- 从 200ms 优化到 135ms
- 精心设计的副作用顺序

**2. 工具系统架构**
- 统一的基类设计
- 清晰的职责分离
- 良好的扩展性

**3. 安全机制**
- 6 层防御体系
- AST 解析检测危险命令
- 完善的权限系统

### 4.2 技术债务

**1. 条件编译过多**
```typescript
// 大量这样的代码
const bridge = feature('BRIDGE_MODE')
  ? require('./commands/bridge/index.js').default
  : null
```

**影响**：
- 代码分支复杂
- 测试覆盖困难
- 维护成本高

**2. 重复代码**
- 多个工具有相似的权限检查逻辑
- 文件操作有重复的编码检测

**3. 硬编码问题**
- 部分路径硬编码
- 超时时间分散在各处

### 4.3 可能的 AI 生成痕迹

**特征 1：过于规整的结构**
- 每个工具都有完全一致的文件结构
- 注释风格高度统一

**特征 2：防御性过度**
- 某些边界情况处理过于详细
- 错误信息异常完善

**特征 3：注释冗余**
- 部分注释重复代码逻辑
- 某些显而易见的代码也有注释

---

## 5. 实战技巧

### 5.1 如何编写高质量 Prompt

**从 Claude Code 学到的技巧**：

1. **使用 NEVER/ALWAYS 等强词**
2. **解释原因，不只是规则**
3. **提供具体示例**
4. **重复关键规则**
5. **结构化组织（标题、列表）**

### 5.2 如何设计工具系统

**关键要素**：

1. **统一的基类**：所有工具继承同一接口
2. **权限检查**：每个工具独立检查
3. **进度报告**：长时间操作支持进度
4. **UI 组件**：独立的渲染逻辑

### 5.3 如何实现自修复

**策略**：

1. **检测失败**：捕获错误信息
2. **分析原因**：解析错误类型
3. **自动修复**：执行修复操作
4. **重试验证**：确认修复成功

---

## 参考资料

- [ARCHITECTURE_DEEP_DIVE.md](./ARCHITECTURE_DEEP_DIVE.md)
- [PROMPTS_ANALYSIS.md](./PROMPTS_ANALYSIS.md)
- [TOOLS_SYSTEM_GUIDE.md](./TOOLS_SYSTEM_GUIDE.md)
- [KEY_FEATURES_ANALYSIS.md](./KEY_FEATURES_ANALYSIS.md)

