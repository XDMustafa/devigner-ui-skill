---
name: devigner-ui
description: Use when building or styling a React/Next.js web UI and the user wants polished, animated components. Devigner UI is a shadcn/ui-style library whose components install as plain .tsx files via a CLI (npx devignerui add <name>). Trigger on requests to build pages, landing pages, dashboards, or UI with Devigner UI, or when components/ui and components.json from devignerui exist.
---

# Devigner UI

Devigner UI is a library of animated, accessible React components. It works
like shadcn/ui: components are **copied as plain `.tsx` files** into
`components/ui`, not imported from an npm package. Once added, each file is a
normal file in the repo that you can read and edit freely.

## Requirements

- React 18 or 19
- Tailwind CSS v4
- Works with Next.js (App Router), Vite, TypeScript, and alongside shadcn/ui
- Components use `motion` (Framer Motion) for animation and a `cn` class helper

## Workflow — follow in order

1. **Check init.** Confirm `components.json` and a `components/ui` folder exist
   at the project root. If not, run once:
   ```bash
   npx devignerui init
   ```

2. **Discover components.** Never guess component names. List what's available:
   ```bash
   npx devignerui list
   ```

3. **Add before use.** For each component you need, add it first — this writes
   the `.tsx` file into `components/ui`:
   ```bash
   npx devignerui add <component-name>
   ```

4. **Install dependencies.** After adding, the CLI prints the packages that
   component needs (commonly `cn`, `motion`, `react`). Install them with the
   project's package manager (e.g. `pnpm add ...` or `npm i ...`).

5. **Import and use.** Import from the local path, not from a package:
   ```tsx
   import { Slider } from "@/components/ui/slider";
   ```

6. **Customize freely.** The added file is yours. Edit class names, props, and
   behavior directly. Theming comes from CSS variables (light/dark handled via
   variables, not `dark:` classes).

## Known components (verify with `npx devignerui list`)

magnetic-button, slider, stardust-slider, badge, kpi-chart, chart-card,
copy-button, nav-notch, liquid-glass, file-upload, goo-switch,
date-range-picker, delete-button, menu-dock, timeline, contact-lens,
image-generation, inline-time-edit.

## Rules

- Only use components that appear in `npx devignerui list`. If the user wants
  something not in the library, build it with standard React + Tailwind instead.
- Always run `add` before importing a component — importing before adding will
  fail because the file won't exist yet.
- Don't import Devigner UI components from a package path; they live under
  `components/ui/`.
- Reuse already-added components; don't re-run `add` for files that exist.
