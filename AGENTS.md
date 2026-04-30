# Repository Guidelines

## Project Structure & Module Organization
The root package is the Bun-based CLI and local server. Main code lives in `src/`: `entrypoints/` for startup paths, `screens/` and `components/` for the Ink TUI, `commands/` for slash commands, `services/` for API/MCP/OAuth logic, and `tools/` for agent tool implementations. `bin/claude-haha` is the executable entrypoint. The desktop app is isolated in `desktop/` with React UI code in `desktop/src/` and Tauri glue in `desktop/src-tauri/`. Documentation is in `docs/` and builds with VitePress. Treat root screenshots and `docs/images/` as reference assets, not source code.

## Build, Test, and Development Commands
Install root dependencies with `bun install`, then install desktop dependencies in `desktop/` if you are touching the app UI.

- `./bin/claude-haha` or `bun run start`: run the CLI locally.
- `SERVER_PORT=3456 bun run src/server/index.ts`: start the local API/WebSocket server used by `desktop/`.
- `bun run docs:dev` / `bun run docs:build`: preview or build the VitePress docs.
- `cd desktop && bun run dev`: run the desktop frontend in Vite.
- `cd desktop && bun run build`: type-check and produce a production web build.
- `cd desktop && bun run test`: run Vitest suites.
- `cd desktop && bun run lint`: run TypeScript no-emit checks.

## Coding Style & Naming Conventions
Use TypeScript with 2-space indentation, ESM imports, and no semicolons to match the existing code. Prefer `PascalCase` for React components, `camelCase` for functions, hooks, and stores, and descriptive file names like `teamWatcher.ts` or `AgentTranscript.tsx`. Keep shared UI in `desktop/src/components/`, API clients in `desktop/src/api/`, and avoid adding new dependencies unless the existing utilities cannot cover the change.

## Testing Guidelines
Desktop tests use Vitest with Testing Library in a `jsdom` environment. Name tests `*.test.ts` or `*.test.tsx`; colocate focused tests near the file or place broader coverage in `desktop/src/__tests__/`. No coverage gate is configured, so add regression tests for any behavior you change and run the relevant suites before opening a PR.

## Local Config Isolation (强制)
本项目（cc-haha）任何修改都不得影响用户本机的 Claude Code 全局配置。具体边界：

- **禁止写入**：`~/.claude/settings.json`、`~/.claude.json`、`~/.claude/CLAUDE.md`、`~/.claude/agents/`、`~/.claude/commands/`、`~/.claude/plugins/`、`~/.claude/oauth.json`、macOS Keychain 中的 Claude Code 凭据。这些是用户主 Claude Code 的私有领域，cc-haha 必须把它们当只读。
- **可读可写**：`~/.claude/cc-haha/` 整个子目录是 cc-haha 的 sandbox，包括 `settings.json`、`providers.json`、`oauth.json` 等都在这里维护。所有 provider env、auth token、model 映射只往这里写。
- **运行时**：CLI/Server 通过 `applySafeConfigEnvironmentVariables()` 把 `~/.claude/cc-haha/settings.json` 的 env 叠加在 `~/.claude/settings.json` 之后，因此 cc-haha 想覆盖什么 env，写到 cc-haha 的 settings 就够了，不需要也不允许动主 settings。
- **桌面端 UI**：`Settings → Providers` 等页面在保存时只能调用 `/api/providers/*`（落到 `~/.claude/cc-haha/`），绝不允许把 `ANTHROPIC_*`、顶层 `model` 等字段直接 `PUT` 到 `~/.claude/settings.json`。
- **诊断/排错**：调试时如需观察 `~/.claude/settings.json` 是只读读取，不能为了"修复 cc-haha 的问题"去清理或改写主 settings；要清理也只在 `~/.claude/cc-haha/` 内操作。
- **Agent 规则**：未经用户明确同意，任何代码改动、命令、脚本都不得修改 `~/.claude/` 下除 `cc-haha/` 子目录以外的任何文件。

## Commit & Pull Request Guidelines
Recent history follows Conventional Commit prefixes such as `feat:`, `fix:`, and `docs:`. Keep subjects imperative and scoped to one change. PRs should explain the user-visible impact, list verification steps, link related issues, and include screenshots for desktop or docs UI changes. Keep diffs reviewable and call out any follow-up work or known gaps.
