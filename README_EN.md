# Claude Code Source Mirror

> Unofficial, unsupported, and provided for research/documentation purposes only.
>
> This repository is not affiliated with Anthropic. For normal usage, installation, updates, and support, please use the official Claude Code product and documentation.

[中文主文档](./readme.md) | [English README](./README_EN.md)

---

### Overview

This repository is an **unofficial Claude Code source mirror**. Based on direct npm verification, it appears to be reconstructed from the publicly shipped source map in the npm package, rather than being Anthropic's full private development repository.

More precisely:

- the public npm package `@anthropic-ai/claude-code@2.1.88` includes both `cli.js` and `cli.js.map`
- I verified directly from the npm tarball that `cli.js.map` is present and is about `59.8MB`
- after unpacking the tarball, I confirmed the source map contains `4756` `sources`, with non-empty `sourcesContent`
- this local repository preserves the recovered `src/` tree and is useful for architecture study, feature analysis, command mapping, and product archaeology

So this is best described as a **near-complete client-side source mirror recoverable from the published package**, not the full internal Anthropic monorepo.

### Quick Facts

| Item | Value |
| --- | --- |
| Repository type | Unofficial source mirror |
| Public package | `@anthropic-ai/claude-code@2.1.88` |
| Exposure vector | `cli.js.map` shipped in npm package |
| Tarball size | `31.2MB` |
| Unpacked size | `102.8MB` |
| Files in tarball | `20` |
| Local files under `src/` | `1902` |
| Local `ts/tsx` files | `1884` |
| Local source lines | `512,685` |
| Visible in this mirror | CLI, commands, tools, Ink/React TUI, MCP, plugins, skills, bridge, remote, voice, permissions, telemetry, feature flags |
| Not included | Claude model weights, training data, Anthropic private backends, full internal CI/ops assets |

### What This Mirror Contains

The recovered source tree covers the major client-side layers of Claude Code:

```text
src/
├── main.tsx
├── QueryEngine.ts
├── Tool.ts
├── tools.ts
├── commands.ts
├── cli/
├── commands/
├── tools/
├── components/
├── services/
├── bridge/
├── remote/
├── plugins/
├── skills/
├── voice/
├── state/
└── ...
```

Several entry files are large enough to show that this is a substantial product codebase, not a trivial wrapper:

- `src/main.tsx`: `4683` lines
- `src/QueryEngine.ts`: `1295` lines
- `src/Tool.ts`: `792` lines
- `src/commands.ts`: `754` lines
- `src/tools.ts`: `389` lines

### What The Source Reveals

At minimum, the recovered client code shows:

- a full terminal-agent workflow for reading, editing, searching, and executing
- a `React + Ink` terminal UI
- a structured tool registry for file, search, web, task, MCP, planning, agent, and team workflows
- a large slash-command surface including `review`, `mcp`, `permissions`, `tasks`, `plugins`, `resume`, and `status`
- deep MCP integration under `services/mcp/`
- bridge / remote / direct-connect / SSH-remote / IDE-related code
- plugins, skills, hooks, worktrees, team/swarm, memory, and teleport-style mechanisms
- voice, telemetry, policy limits, permissions, managed settings, and sync systems

The codebase also exposes many feature flags or internal codenames, including:

- `KAIROS`
- `ULTRAPLAN`
- `COORDINATOR_MODE`
- `BRIDGE_MODE`
- `DIRECT_CONNECT`
- `SSH_REMOTE`
- `WEB_BROWSER_TOOL`
- `WORKFLOW_SCRIPTS`
- `BUDDY`

Their presence is useful for research, but **a codename in source does not mean the feature shipped publicly**.

### What It Is Not

This repository is **not**:

- the Claude model itself
- training code, datasets, or inference weights
- Anthropic's full backend stack
- a guaranteed buildable official development repository
- an officially supported open-source release

The most accurate description is:

> a Claude Code client source mirror reconstructed from the public npm package's source map

### Public Timeline

#### 2025

- Anthropic publicly documented and marketed Claude Code as an agentic coding tool that lives in the terminal
- Anthropic's `Introducing Claude 4` announcement explicitly said Claude Code was generally available

#### 2026-03-31

- public community discussion surfaced around the npm source map exposing Claude Code source
- one of the earliest widely shared public links was Chaofan Shou's X post:
  `https://x.com/Fried_rice/status/2038894956459290963?s=20`
- I independently verified that `@anthropic-ai/claude-code@2.1.88` contains `cli.js.map`
- the map includes full `sourcesContent`, making human-readable TypeScript/TSX recovery possible

### How To Verify And Explore

#### 1. Check the package metadata

```bash
npm view @anthropic-ai/claude-code@2.1.88 version dist.tarball homepage
```

#### 2. Download and inspect the tarball

```bash
npm pack @anthropic-ai/claude-code@2.1.88
tar -xzf anthropic-ai-claude-code-2.1.88.tgz
ls package
```

You should see at least:

- `package/cli.js`
- `package/cli.js.map`
- `package/package.json`

#### 3. Explore this local mirror

```bash
rg --files src | wc -l
rg --files src -g '*.ts' -g '*.tsx' | wc -l
find src -maxdepth 1 -type d | sort
```

#### 4. Recommended entry points

- `src/main.tsx`
- `src/QueryEngine.ts`
- `src/tools.ts`
- `src/Tool.ts`
- `src/commands.ts`
- `src/services/mcp/`
- `src/bridge/`
- `src/plugins/`
- `src/skills/`

### Limitations

Even though this mirror is highly useful, keep in mind:

- the root-level official build context is not fully preserved in this mirror
- many code paths are gated by feature flags, environment variables, or internal user types
- some flows depend on Anthropic-private APIs, services, or runtime infrastructure
- internal names and experimental branches should not be treated as a confirmed public roadmap

### Good Use Cases

- studying Claude Code's client architecture
- learning how an agentic CLI composes tools
- analyzing permission and state-management patterns for terminal coding agents
- studying MCP, plugins, skills, hooks, and remote bridge integration
- creating command maps, feature inventories, and architecture notes

### Poor Use Cases

- treating this as an official or supported baseline
- redistributing proprietary material without considering IP and usage terms
- assuming internal names in source equal public features
- treating this mirror as Anthropic's entire internal system

### Community-Enhanced References

If you want something beyond a raw source mirror, one useful community reference is:

- `nirholas/claude-code`: <https://github.com/nirholas/claude-code>

That repository is helpful because it adds:

- a more elaborate README and architecture-oriented explanation
- extra navigation and documentation layers for researchers
- an MCP-server-based exploration workflow for the recovered source

Important distinction:

- it is a community-enhanced derivative, not an official Anthropic repository
- it represents a documented/explainer layer on top of the recovered source
- if your goal is preserving a closer-to-raw mirror, this repository should stay more conservative in scope

### Public References

- Official product page: <https://www.anthropic.com/claude-code>
- Official overview docs: <https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview>
- Official security docs: <https://docs.anthropic.com/en/docs/claude-code/security>
- Official MCP docs: <https://docs.anthropic.com/en/docs/claude-code/mcp>
- Official GitHub repository: <https://github.com/anthropics/claude-code>
- npm package page: <https://www.npmjs.com/package/@anthropic-ai/claude-code>
- `2.1.88` tarball: <https://registry.npmjs.org/@anthropic-ai/claude-code/-/claude-code-2.1.88.tgz>
- Anthropic `Introducing Claude 4`: <https://www.anthropic.com/news/claude-4>
- Chaofan Shou on X:
  <https://x.com/Fried_rice/status/2038894956459290963?s=20>
- Community-enhanced mirror `nirholas/claude-code`:
  <https://github.com/nirholas/claude-code>
- Sample Reddit discussions from March 31, 2026:
  - <https://www.reddit.com/r/ClaudeAI/comments/1s8lkkm/i_dug_through_claude_codes_leaked_source_and/>
  - <https://www.reddit.com/r/claude/comments/1s8p012/claude_code_source_code_has_been_leaked_via_a_map/>
  - <https://www.reddit.com/r/LocalLLaMA/comments/1s8ijfb/claude_code_source_code_has_been_leaked_via_a_map/>

### Disclaimer

All code, names, prompts, implementation details, and related intellectual property belong to their original rights holders. This README and mirror are intended for:

- research
- documentation
- architecture study
- security and product analysis

If you want to actually use Claude Code, prefer Anthropic's official product, documentation, and repository.
