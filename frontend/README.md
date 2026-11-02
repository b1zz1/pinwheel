# Pinwheel frontend

A Next.js app in its early setup stage. The tooling and UI libraries are in place; application
features have not been built yet (`app/page.tsx` is still the create-next-app starter).

All commands below run from this `frontend/` folder.

## Stack

- **Framework:** [Next.js 16](https://nextjs.org) (App Router, Turbopack) with React 19 and
  TypeScript
- **Styling:** [Tailwind CSS v4](https://tailwindcss.com) with `tw-animate-css`
- **UI components:** [shadcn/ui](https://ui.shadcn.com) (`base-luma` style, built on
  [Base UI](https://base-ui.com)), with icons from [Lucide](https://lucide.dev)
- **Data:** [AG Grid](https://www.ag-grid.com) and [AG Charts](https://www.ag-grid.com/charts/)
  (installed, not used yet)
- **Fonts:** Inter (sans) and Geist Mono, loaded with `next/font`

> **Note:** Next.js 16 has breaking changes compared to earlier versions. Check the bundled docs
> in `node_modules/next/dist/docs/` before relying on older APIs or conventions.

## Getting started

```bash
npm install
npm run dev
```

Then open [http://localhost:3000](http://localhost:3000).

## Scripts

| Script                 | What it does                                   |
| ---------------------- | ---------------------------------------------- |
| `npm run dev`          | Start the development server                   |
| `npm run build`        | Build for production                           |
| `npm run start`        | Serve the production build                     |
| `npm run lint`         | Run ESLint (`lint:fix` to auto-fix)            |
| `npm run format`       | Format everything with Prettier                |
| `npm run format:check` | Check formatting without writing               |
| `npm run typecheck`    | Type-check with `tsc --noEmit`                 |
| `npm run check`        | Run typecheck, lint, and format check together |

## Project structure

```text
app/              Routes, root layout, and global styles (globals.css holds the theme tokens)
components/ui/    shadcn/ui components (currently: button)
lib/utils.ts      The cn() class-name helper
public/           Static assets
```

To add shadcn components:

```bash
npx shadcn@latest add <component>
```

## Code quality

- **Prettier:** no semicolons, double quotes, trailing commas, 100-column lines. Tailwind classes
  are sorted by `prettier-plugin-tailwindcss`, including inside `cn()` and `cva()`.
- **ESLint:** `eslint-config-next`, with `eslint-config-prettier` so it doesn't conflict with
  Prettier.
- **Pre-commit:** the root `.husky/pre-commit` hook runs `lint-staged` here, which applies ESLint
  and Prettier to staged frontend files.
