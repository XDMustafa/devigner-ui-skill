---
name: devigner-ui
description: Use when the user explicitly asks for Devigner UI (npx devignerui, ui.devigner.cc, its components) or the project already uses it — devignerui in package.json, or components/ui/*.tsx written by its CLI. Do not introduce Devigner UI into a project that doesn't use it unless asked. It is a shadcn/ui-style library whose components install as plain .tsx files via a CLI (npx devignerui add <name>).
---

# Devigner UI

Devigner UI is a library of animated, accessible React components. It works
like shadcn/ui: components are **copied as plain `.tsx` files** into
`components/ui`, not imported from an npm package. Once added, each file is a
normal file in the repo that you can read and edit freely.

**Scope:** this skill never installs Devigner UI into a project that doesn't
already use it (`devignerui` in package.json, or CLI-generated files under
`components/ui/`) unless the user explicitly asks for Devigner UI. A
`components.json` alone is not a signal — shadcn/ui creates that too.

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

4. **Read the props — local file first, site guide second.**
   1. **Primary:** the installed `components/ui/<name>.tsx` itself. Its
      interfaces/types are the exact truth for the installed version, and
      always readable — no network needed.
   2. **Secondary:** the official markdown guide for docs the file doesn't
      spell out (keyboard behavior, troubleshooting):
      ```text
      https://ui.devigner.cc/llms/<component-name>.md
      ```
      (`https://ui.devigner.cc/llms-full.txt` has all of them in one file.)
      The site may document a **newer** version than installed — when the two
      disagree, the local file wins.
   Never guess prop names.

5. **Install dependencies.** After adding, the CLI prints the packages that
   component needs (commonly `cn`, `motion`, `react`). Install them with the
   project's package manager (e.g. `pnpm add ...` or `npm i ...`).

6. **Import and use.** Import from the local path, not from a package:
   ```tsx
   import { Slider } from "@/components/ui/slider";
   ```

7. **Customize freely.** The added file is yours. Edit class names, props, and
   behavior directly. Theming comes from CSS variables (light/dark handled via
   variables, not `dark:` classes).

## Known components (verify with `npx devignerui list`)

28 as of devignerui v1.9.0 — this list rots with every release, so always
confirm against `npx devignerui list` (what your installed version ships) or
https://ui.devigner.cc/components (what exists today). Don't guess names from
memory. Upstream also re-ships existing components — e.g. liquid-glass,
file-upload, date-range-picker, chart-card and menu-dock changed in v1.9.0.

## Rules

- Only use components that appear in `npx devignerui list` **or** on
  https://ui.devigner.cc/components. If the user wants something not in the
  library, build it with standard React + Tailwind instead.
- Always run `add` before importing a component — importing before adding will
  fail because the file won't exist yet.
- Read props from the local `components/ui/<name>.tsx` first (exact match for
  the installed version, works offline), then the site guide
  (`https://ui.devigner.cc/llms/<name>.md`); on conflict the local file wins.
  Never guess props.
- Don't import Devigner UI components from a package path; they live under
  `components/ui/`.
- Don't re-run `add` for files that exist — it overwrites manual edits. The
  only exception is when the user explicitly asks for the upstream update;
  then show them a diff first, because their customizations will be lost.

**Before a production build** (code splitting, deployment, Lighthouse scores),
read `references/performance.md` in this skill — Vite 8/Rolldown pitfalls and
lazy-loading lessons from shipped projects.

## Field notes (from real projects)

### The CLI list lags the website

`npx devignerui list` reports what the **locally installed** `devignerui`
package ships — a project pinning `^1.7.0` lists 18 components while the site
(https://ui.devigner.cc/components) has 28. If a component the user names is
missing from `list`, check the site or run `npx devignerui@latest list`
before telling them it doesn't exist. Upstream also re-ships existing
components (liquid-glass, file-upload, date-range-picker, chart-card,
menu-dock were updated in v1.9.0) — an old pin won't get those updates from
`add`.

### Icons: alias the root, subpaths pass through

`@devigner-ui/icons` ships **2,147 symbols × 4 styles = 8,588 icon exports**
(2,147 is the symbol count — that's what the site's "2,100+" means). Importing
the root entry pulls the whole set into the bundle and does not tree-shake.
Create a barrel file (`src/lib/icons.ts`) that re-exports only the icons you
use from subpaths, then alias the root to it:

```ts
// vite.config.ts — exact match only, so subpath imports pass through
{ find: /^@devigner-ui\/icons$/, replacement: path.resolve(__dirname, "src/lib/icons.ts") }
```

```ts
// src/lib/icons.ts
export { IconArrowRight } from "@devigner-ui/icons/ArrowRight";
```

Imports already written by the CLI inside generated components
(`@devigner-ui/icons/ChevronRight` in menu-dock, etc.) are subpath imports and
resolve directly — no change needed.

### Dependency map

| Import | From package | What |
|--------|--------------|------|
| `devignerui/hooks` | `devignerui` | `useReducedMotionPreference`, `useControllableState` |
| `devignerui/motion` | `devignerui` | `SPRING_LAYOUT`, `SPRING_PRESS`, `SPRING_SWAP`, `SPRING_RETRACT`, `GSAP_MAGNET` |
| `devignerui/menu-path` | `devignerui` | `menuAt`, `backFocusIndex`, `MenuNode` |
| `motion/react` | `motion` | `motion`, `AnimatePresence`, `LayoutGroup` |
| `cn` | `cn` | class merge helper (shadcn-compatible) |
| `gsap` | `gsap` | only magnetic-button needs it |

### Dark mode: non-default theme selectors

Devigner theming comes from CSS variables, so it works with any theme
switcher. But a few generated components also carry `dark:` utilities (e.g.
badge), and Tailwind v4 resolves those against its **default** selector
(`class="dark"` on `<html>`). If the app toggles theme any other way — an
attribute like `data-theme`, a `data-mode`, a class name other than `dark` —
map the variant once or those utilities stay dead:

```css
/* e.g. theme switch sets data-theme="dark" on <html> instead of class */
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

### Non-composited animation in menu-dock

`Row` elements ship with `transition-colors` (Back button, menu items). On a
scrolling page Chromium flags this as non-composited style animation. For
PageSpeed, drop `transition-colors` from the rows — hover colors still change,
just instantly, which is visually near-identical on a dock.