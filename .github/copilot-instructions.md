# Copilot Instructions for OpenCode

## Project Overview

OpenCode is an open-source AI coding agent built with TypeScript and Bun. It runs as a CLI tool with a TUI (terminal UI), a headless API server, and a desktop app. The core uses the [Effect](https://effect.website/) functional programming library, [Drizzle ORM](https://orm.drizzle.team/) for SQLite, and [SolidJS](https://www.solidjs.com/) for all UI layers.

## Repository Layout

```
packages/
  opencode/       # Core business logic, CLI, TUI, and headless API server
  app/            # Shared web UI (SolidJS + Tailwind)
  desktop/        # Tauri-based native desktop app (wraps packages/app)
  desktop-electron/ # Electron-based desktop app
  sdk/js/         # JavaScript SDK (regenerate with ./packages/sdk/js/script/build.ts)
  plugin/         # @opencode-ai/plugin package
  console/        # Console app
  ui/             # Shared UI primitives
  storybook/      # Component storybook
```

Key source directories inside `packages/opencode/src/`:
- `agent/` – AI agent orchestration
- `config/` – User/project configuration (follow self-export pattern: `export * as ConfigFoo from "./foo"`)
- `provider/` – AI provider integrations (Anthropic, OpenAI, Gemini, etc.)
- `tool/` – Agent tools (file read/write, shell, etc.)
- `server/` – Hono-based HTTP + WebSocket API server
- `lsp/` – Language server protocol integration
- `mcp/` – Model Context Protocol integration
- `session/` – Session and conversation management
- `storage/` – SQLite via Drizzle ORM

## Development Setup

Requirements: **Bun 1.3+**

```bash
bun install       # Install all dependencies
bun dev           # Run TUI in packages/opencode directory
bun dev <dir>     # Run TUI in a specific directory
bun dev serve     # Start headless API server (default port 4096)
bun dev web       # Start server + open web interface
```

After changing the API or SDK, regenerate SDKs:
```bash
./script/generate.ts
# or specifically for the JS SDK:
./packages/sdk/js/script/build.ts
```

## Commands

| Command | Description |
|---|---|
| `bun install` | Install all workspace dependencies |
| `bun dev` | Start the TUI (dev mode) |
| `bun lint` | Run oxlint across the repo |
| `bun typecheck` | Run type checking via Turborepo (from repo root) |
| `bun turbo test:ci` | Run all unit tests via Turborepo |

**Important:** Do not run `bun test` from the repo root — it exits with an error by design. Run tests from package directories:
```bash
cd packages/opencode && bun test
```

Always run `bun typecheck` from a package directory (e.g. `packages/opencode`), never use `tsc` directly.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible (e.g. `Bun.file()`)
- Rely on type inference; avoid explicit type annotations unless needed for exports or clarity
- Prefer functional array methods (`flatMap`, `filter`, `map`) over `for` loops; use type guards on `filter` to maintain type inference
- In `src/config`, follow the self-export pattern: `export * as ConfigAgent from "./agent"`

Inline values that are only used once:
```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### Destructuring

Avoid unnecessary destructuring — use dot notation to preserve context:
```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment:
```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### Control Flow

Avoid `else` statements — prefer early returns:
```ts
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

### Drizzle Schema Definitions

Use snake_case for field names so column names don't need to be redefined as strings:
```ts
// Good
const table = sqliteTable("session", {
  id: text().primaryKey(),
  project_id: text().notNull(),
  created_at: integer().notNull(),
})

// Bad
const table = sqliteTable("session", {
  id: text("id").primaryKey(),
  projectID: text("project_id").notNull(),
  createdAt: integer("created_at").notNull(),
})
```

## Testing

- Avoid mocks as much as possible — test the actual implementation
- Do not duplicate logic into tests
- Tests **cannot** run from repo root; run from package directories like `packages/opencode`
- Use `bun test` in the package directory, or `bun turbo test:ci` from the root for CI

## Branch & Git Conventions

- Default branch is `dev`
- Local `main` ref may not exist; use `dev` or `origin/dev` for diffs
- PRs target `dev`

## What Gets Merged

- Bug fixes
- Additional LSPs / formatters
- Improvements to LLM performance
- Support for new providers (prefer a PR to [anomalyco/models.dev](https://github.com/anomalyco/models.dev) first)
- Fixes for environment-specific quirks
- Missing standard behavior
- Documentation improvements

Any UI or core product feature requires design review with the core team before implementation.
