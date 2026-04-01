# 权限与安全系统深度解析

> 深入分析 Claude Code 的权限系统、安全机制和策略限制

## 目录

- [1. 权限系统架构](#1-权限系统架构)
- [2. 权限决策机制](#2-权限决策机制)
- [3. 权限分类器](#3-权限分类器)
- [4. 策略限制系统](#4-策略限制系统)

---

## 1. 权限系统架构

### 1.1 权限决策类型

从 `PermissionResult.ts` 提取的核心类型：

```typescript
export type PermissionDecision = 
  | PermissionAllowDecision   // 允许
  | PermissionDenyDecision    // 拒绝
  | PermissionAskDecision     // 询问用户
```

**三种行为**：
- `allow`: 自动允许
- `deny`: 自动拒绝
- `ask`: 询问用户确认

### 1.2 权限相关文件

从源码中发现的权限系统文件：

| 文件 | 功能 |
|------|------|
| `permissionsLoader.ts` | 加载权限配置 |
| `bashClassifier.ts` | Bash 命令分类 |
| `yoloClassifier.ts` | YOLO 模式分类器 |
| `permissionSetup.ts` | 权限初始化 |
| `shadowedRuleDetection.ts` | 检测被覆盖的规则 |
| `bypassPermissionsKillswitch.ts` | 紧急绕过开关 |

### 1.3 权限元数据

```typescript
export type PermissionMetadata = {
  reason?: PermissionDecisionReason
  // 其他元数据
}
```

---

## 2. 权限决策机制

### 2.1 Bash 命令分类器

从 `bashClassifier.ts` 发现的分类系统：

**分类器结果**：
```typescript
export type ClassifierResult = {
  matches: boolean           // 是否匹配规则
  matchedDescription?: string  // 匹配的描述
  confidence: 'high' | 'medium' | 'low'  // 置信度
  reason: string             // 原因
}
```

**分类器行为**：
```typescript
export type ClassifierBehavior = 'deny' | 'ask' | 'allow'
```

### 2.2 Prompt 规则

**规则格式**：
```typescript
const PROMPT_PREFIX = 'prompt:'
// 示例：prompt: delete all files
```

**功能**：
- 使用自然语言描述规则
- AI 分类命令是否匹配
- 支持模糊匹配

### 2.3 ANT-ONLY 功能

**重要发现**：
```typescript
// Stub for external builds - classifier permissions feature is ANT-ONLY
```

**含义**：
- Bash 分类器是 Anthropic 内部专用功能
- 外部版本返回 stub（存根）
- `isClassifierPermissionsEnabled()` 返回 `false`

---

## 3. 权限分类器

### 3.1 三种分类器

**1. Bash 分类器**
- 分析 Bash 命令
- 基于 Prompt 规则匹配
- 返回置信度

**2. YOLO 分类器**
- "You Only Live Once" 模式
- 可能用于一次性操作
- 文件：`yoloClassifier.ts`

**3. 通用分类器**
- 其他工具的权限判断

### 3.2 置信度系统

**三个级别**：
- `high`: 高置信度匹配
- `medium`: 中等置信度
- `low`: 低置信度

**用途**：
- 高置信度：直接执行决策
- 低置信度：可能需要用户确认

---

## 4. 策略限制系统

### 4.1 策略限制服务

**位置**：`services/policyLimits/`

**功能**：
- 企业策略管理
- 远程配置同步
- 功能开关控制

### 4.2 紧急绕过开关

**文件**：`bypassPermissionsKillswitch.ts`

**用途**：
- 紧急情况下绕过权限
- 可能用于调试
- 需要特殊授权

### 4.3 影子规则检测

**文件**：`shadowedRuleDetection.ts`

**功能**：
- 检测被覆盖的权限规则
- 避免规则冲突
- 提高配置可维护性

---

## 5. 核心发现

### 5.1 内外部差异

**外部版本**：
- Bash 分类器被禁用
- 返回 stub 实现
- 功能受限

**内部版本（ANT-ONLY）**：
- 完整的 AI 分类器
- Prompt 规则支持
- 高级权限控制

### 5.2 安全设计理念

1. **默认拒绝**：不确定时拒绝
2. **分层防御**：多层权限检查
3. **可审计**：所有决策可追溯
4. **灵活配置**：支持企业策略

---

## 参考资料

- [ARCHITECTURE_DEEP_DIVE.md](./ARCHITECTURE_DEEP_DIVE.md)
- [KEY_FEATURES_ANALYSIS.md](./KEY_FEATURES_ANALYSIS.md)

