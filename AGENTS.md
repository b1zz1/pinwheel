# Pinwheel: agent instructions

This is a monorepo. Each top-level folder is a self-contained package with its own dependencies
and tooling. Run commands from inside the package you're working on, not from the repo root.

| Folder      | What it is                                        |
| ----------- | ------------------------------------------------- |
| `frontend/` | Next.js 16 app (React 19, Tailwind v4, shadcn/ui) |
| `backend/`  | Python backend (planned; not created yet)         |
| `docs/`     | Documentation (planned; not created yet)          |

Before working in a package, read its own `AGENTS.md` (for example `frontend/AGENTS.md`). The
more specific file wins when instructions conflict.

## Where things go

- Repo-wide config stays at the root: git hooks (`.husky/`), editor settings (`.vscode/`),
  `.editorconfig`, the root `.gitignore`, `.mcp.json`, and agent skills (`.agents/skills/`,
  linked into `.claude/skills/`).
- Language-specific config stays inside its package: dependencies, linters, formatters, and
  ignores for build output. Don't add a root `package.json` or root Prettier/ESLint config.

## Checks

- Frontend: `cd frontend && npm run check` (typecheck, lint, format check).
- The pre-commit hook (`.husky/pre-commit`) runs each package's lint-staged step. When adding a
  package, add its step there.
