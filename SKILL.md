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

28 as of devignerui v1.9.0. Marketing: flick-cards-slider, menu-notch,
rating, flip-button, contact-lens, timeline, magnetic-button. Interface:
command-palette, context-menu, dynamic-island, morphing-popover, tabs,
expand-tabs, stardust-slider, badge, image-generation, kpi-chart,
copy-button, nav-notch, liquid-glass, slider, file-upload, goo-switch,
inline-time-edit, date-range-picker, chart-card, delete-button, menu-dock.

Recently updated by upstream: liquid-glass, file-upload, date-range-picker,
chart-card, menu-dock — re-`add` them (or diff against components/ui/) when a
project pins an older version.

## Rules

- Only use components that appear in `npx devignerui list`. If the user wants
  something not in the library, build it with standard React + Tailwind instead.
- Always run `add` before importing a component — importing before adding will
  fail because the file won't exist yet.
- Don't import Devigner UI components from a package path; they live under
  `components/ui/`.
- Reuse already-added components; don't re-run `add` for files that exist.

## Field notes (from real projects)

Practical lessons from shipping sites built with these components. Read these
before wiring a Devigner component into a production build.

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

`@devigner-ui/icons` root entry bundles all 6,000+ icons (~6 MB) and does not
tree-shake. Create a barrel file (`src/lib/icons.ts`) that re-exports only the
icons you use from subpaths, then alias the root to it:

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

### Dark mode: this project's theming convention

Devigner theming comes from CSS variables, not `dark:` Tailwind classes. When
the app switches theme by setting `data-theme` on `<html>` (not `class`), Tailwind
v4 needs one custom variant line or `dark:` utilities in generated components
stay dead:

```css
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

### Non-composited animation in menu-dock

`Row` elements ship with `transition-colors` (Back button, menu items). On a
scrolling page Chromium flags this as non-composited style animation. For
PageSpeed, drop `transition-colors` from the rows — hover colors still change,
just instantly, which is visually near-identical on a dock.

### Vite 8 / Rolldown build pitfalls

Vite 8 uses Rolldown instead of Rollup. What we hit:

- `build.rollupOptions.output.chunkFileNames` / `assetFileNames` are not
  honored the way Rollup's are; chunk hashes can drift between local (macOS)
  and CI (Linux) builds. Trust the build output names, don't hardcode them.
- `build.cssCodeSplit: false` merges emitted CSS but leaves a **stale
  per-chunk CSS reference in the JS manifest**. The runtime then throws
  `Unable to preload CSS for ...` — and because the throw happens during a
  `React.lazy` import, the whole tree unmounts silently (blank page). Fix: a
  post-build script that copies the real CSS file to every name the manifest
  still references (see `scripts/fix-css-manifest.mjs` pattern).
- Keep root `index.html` as the Vite **source template**. If a build step
  overwrites it with `dist/index.html`, CI rebuilds the old bundle forever and
  ignores source edits. Deploy `dist/` via CI workflow only.

### Lazy pages with Devigner components

A `React.lazy` component nested inside another lazy import (e.g. a lazy route
containing a lazy content body) needs its **own** `Suspense` boundary at each
level. Without it, when the inner chunk resolves React has no fallback to
render and silently unmounts the app — blank screen, no console error. Wrap
every lazy boundary:

```tsx
<Suspense fallback={<div>…</div>}>
  <LazyBody />
</Suspense>
```