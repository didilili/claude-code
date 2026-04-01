# 常见问题（FAQ）

[返回 README](../README.md)

---

## 📌 关于项目

### Q: 这个仓库是官方开源的吗？

**A:** 不是。这是从公开 npm 包 `@anthropic-ai/claude-code@2.1.88` 的 source map 恢复的源码镜像，仅供学习研究。官方仓库请访问 [github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)。

### Q: 可以用于商业项目吗？

**A:** 不建议。源码版权归 Anthropic 所有，本仓库仅用于研究和学习。商业使用请使用官方产品。

### Q: 与官方版本有什么区别？

**A:**

- ✅ 相同：完整的客户端源码
- ❌ 缺少：官方支持、自动更新、完整后端服务
- ✅ 额外：中文文档、学习路径、可运行配置

---

## 🔧 安装问题

### Q: 为什么必须用 Bun？

**A:** 源码使用了 Bun 特有的 API（如 `bun:bundle`），Node.js 无法直接运行。安装 Bun：

```bash
curl -fsSL https://bun.sh/install | bash
```

### Q: 安装依赖时报错怎么办？

**A:** 常见解决方案：

```bash
# 清理缓存重试
rm -rf node_modules bun.lock
bun install

# 检查 Bun 版本（需要 >= 1.3.5）
bun --version
```

### Q: macOS 以外的系统能运行吗？

**A:** 理论上 Linux 可以，Windows 可能需要 WSL。本仓库主要在 macOS 上测试。

---

## 🚀 运行问题

### Q: 启动后卡在工作区信任页面？

**A:** 这是正常的安全确认。选择"信任"后才能继续。详见 [RUNNING_SETUP.md](./RUNNING_SETUP.md#第一次启动会发生什么)。

### Q: 提示缺少 API Key？

**A:** Claude Code 需要 Anthropic API Key。设置方式：

```bash
export ANTHROPIC_API_KEY=your_key_here
bun run start
```

或使用 `--bare` 模式跳过认证测试：

```bash
bun run start --bare --print "hello"
```

### Q: 运行时报模块找不到？

**A:** 检查是否在正确目录：

```bash
cd /Users/tools/Music/cc/claude-codes
bun install
bun run start
```

---

## 📚 学习问题

### Q: 从哪里开始读源码？

**A:** 推荐路径见 [SOURCE_STUDY_GUIDE.md](./SOURCE_STUDY_GUIDE.md#推荐阅读路径)。快速入门：

1. 先读 `src/main.tsx` 了解启动流程
2. 再读 `src/QueryEngine.ts` 了解核心循环
3. 选择感兴趣的子系统深入

### Q: 代码量太大，如何快速理解？

**A:** 不要试图一次读完。建议：

- 先看 [SOURCE_STUDY_GUIDE.md](./SOURCE_STUDY_GUIDE.md#架构总览) 建立全局认知
- 选择一个具体功能（如某个工具）深入
- 使用 `rg` 或 `grep` 搜索关键词

### Q: 有没有视频教程或讲解？

**A:** 目前没有。欢迎社区贡献教程并提交 PR。

---

## 🤝 贡献问题

### Q: 可以提交 PR 吗？

**A:** 欢迎！特别是：

- 文档改进
- 错误修正
- 学习路径补充
- 翻译优化

请通过 Issue 或 PR 提交你的改进建议。

### Q: 发现文档错误怎么办？

**A:** 请提交 Issue 或直接 PR 修正。

---

## ⚖️ 法律问题

### Q: 使用这个仓库合法吗？

**A:** 源码来自公开 npm 包，用于学习研究属于合理使用。但请注意：

- 不要用于商业目的
- 不要声称为官方项目
- 尊重原始版权

### Q: Anthropic 会要求删除吗？

**A:** 可能。如果收到 DMCA 通知，仓库可能被下架。建议及时 fork 或本地备份。

---

## 🔍 其他问题

### Q: 与 nirholas/claude-code 有什么区别？

**A:**

- **nirholas/claude-code**: 更强的文档化，包含 MCP 探索工具
- **本仓库**: 中文友好，分层文档，可运行配置

两者可以互补使用。

### Q: 后续会更新吗？

**A:** 本仓库基于 `2.1.88` 版本。如果社区发现新版本的 source map，可能会更新。

### Q: 如何验证源码真实性？

**A:** 可以自行验证：

```bash
npm pack @anthropic-ai/claude-code@2.1.88
tar -xzf anthropic-ai-claude-code-2.1.88.tgz
ls -lh package/cli.js.map
```

---

## 💬 还有问题？

- 查看 [RUNNING_SETUP.md](./RUNNING_SETUP.md) 了解运行细节
- 查看 [SOURCE_STUDY_GUIDE.md](./SOURCE_STUDY_GUIDE.md) 了解源码结构
- 提交 [GitHub Issue](../../issues) 寻求帮助
