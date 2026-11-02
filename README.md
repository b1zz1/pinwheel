# Pinwheel

Pinwheel is a monorepo. Each top-level folder is a self-contained package with its own
dependencies, tooling, and README.

| Package                    | What it is                                         | Status          |
| -------------------------- | -------------------------------------------------- | --------------- |
| [`frontend/`](./frontend/) | Next.js 16 web app (React 19, Tailwind v4, shadcn) | Early setup     |
| `backend/`                 | Python API                                         | Not started yet |
| `docs/`                    | Project documentation                              | Not started yet |

## Repository layout

```text
pinwheel/
├── frontend/          Next.js app: its own package.json, ESLint, Prettier, lint-staged
├── backend/           (planned) Python backend: its own pyproject.toml and tooling
├── docs/              (planned) Documentation
├── .agents/skills/    Agent skills, tracked in skills-lock.json
├── .claude/skills/    Symlinks into .agents/skills/ so Claude Code picks them up
├── .husky/            Git hooks for the whole repo
├── .vscode/           Shared editor settings (open the repo root in VS Code)
├── .editorconfig      Whitespace rules for every language
├── .gitignore         Repo-wide ignores (packages add their own .gitignore)
└── .mcp.json          MCP servers for coding agents
```

**What goes where:** anything that affects the whole repo (git hooks, editor settings, agent
setup) lives at the root. Anything language-specific (dependencies, linters, formatters, build
output ignores) lives inside its package.

## Getting started

```bash
cd frontend
npm install   # also installs the repo's git hooks
npm run dev
```

See [`frontend/README.md`](./frontend/README.md) for scripts and conventions.

## Git hooks

[Husky](https://typicode.github.io/husky/) is installed from `frontend/` but runs the hooks in the
root [`.husky/`](./.husky/) folder. The pre-commit hook runs each package's checks on its staged
files. When the backend lands, add its step (for example `(cd backend && ruff check --fix)`) to
`.husky/pre-commit`.

## AI tooling

- `.agents/skills/` holds agent skills (frontend design, React/Next.js best practices, codebase
  auditing, and others). They are linked into `.claude/skills/` and tracked in
  `skills-lock.json`.
- `.mcp.json` configures the [shadcn MCP server](https://ui.shadcn.com/docs/mcp). It points at a
  local `npx` path, so other machines may need to adjust it.
- `frontend/AGENTS.md` holds agent instructions for the Next.js app. `next dev` regenerates its
  Next.js section.
