# Performance & build pitfalls (Vite 8 / Rolldown / lazy loading)

Lessons from shipping a Vite 8 + Devigner UI site to GitHub Pages. Read this
before a production build or when a lazy route shows a blank page.

**Versions these were tested on:** Vite 8.3.4 / Rolldown 1.2.13,
devignerui 1.9.0 (Nov 2026). These are workarounds with an expiry date —
before applying any of them, confirm the problem still exists (reproduce the
error, or check the current Vite/Rolldown changelog). None of this is
Devigner-specific; it applies to any Vite 8 project.

## Vite 8 uses Rolldown, not Rollup

- `build.rollupOptions.output.chunkFileNames` / `assetFileNames` are not
  honored the way Rollup's are; chunk hashes can drift between local (macOS)
  and CI (Linux) builds. Trust the build output names, don't hardcode them.
- Keep root `index.html` as the Vite **source template**. If a build step
  overwrites it with `dist/index.html`, CI rebuilds the old bundle forever and
  ignores source edits. Deploy `dist/` via CI workflow only.

## `cssCodeSplit: false` leaves a ghost CSS reference

**Only if you actually see this error** (and upstream hasn't fixed it yet):

`build.cssCodeSplit: false` merges emitted CSS into one file, but Rolldown
keeps a **stale per-chunk CSS name in the JS chunk manifest**. At runtime the
loader tries to preload it, gets a 404, and throws:

```text
Unable to preload CSS for https://…/assets/blackbody-XXXX.css
```

Because that throw happens inside a `React.lazy` import, React silently
unmounts the whole tree — blank page, no visible error.

**Fix:** a post-build step (Node, run between `vite build` and deploy) that
copies the real CSS bundle to every name the manifest still references:

```js
// scripts/fix-css-manifest.mjs (run after vite build)
import fs from "node:fs"; import path from "node:path";
const assets = path.resolve("dist/assets");
const realCss = fs.readdirSync(assets).find((f) => f.endsWith(".css"));
for (const f of fs.readdirSync(assets).filter((f) => f.endsWith(".js"))) {
  const src = fs.readFileSync(path.join(assets, f), "utf-8");
  for (const m of src.matchAll(/`\.\/([A-Za-z0-9_-]+\.css)`/g)) {
    const target = path.join(assets, m[1]);
    if (!fs.existsSync(target)) fs.copyFileSync(path.join(assets, realCss), target);
  }
}
```

Wire it up: `"build": "vite build && node scripts/fix-css-manifest.mjs"`.

Alternatively, avoid per-chunk CSS entirely: import global stylesheets (e.g.
`katex/dist/katex.min.css`) from the app entry, never from a lazy component.

## Nested `React.lazy` needs a Suspense at every level

A lazy component inside another lazy import (lazy route → lazy content body)
must have its **own** `Suspense` boundary. Without it, when the inner chunk
resolves there is no fallback to render and React unmounts the app — blank
screen, no console error:

```tsx
<Suspense fallback={<div>…</div>}>
  <LazyBody />
</Suspense>
```

## Measured results (example from one project)

**Example from one project** — a Persian/English blog (KaTeX, GitHub Pages,
Persian fonts). Don't treat these as a required recipe for every site; they
are what that project did, not what Devigner demands.

- **Devigner-specific (2 of these):** icons barrel alias (root entry pulls
  every icon export, doesn't tree-shake) and dropping `transition-colors`
  from menu-dock rows for compositor-friendly hover.
- **General web work (everything else):** code-splitting (main bundle
  837 KB → 433 KB, gzip 263 KB → 138 KB), prerendering the shell into
  `index.html` for first paint before JS, and font subsetting
  (TTF → WOFF2 with `pyftsubset --unicodes=U+0600-06FF,…`, drop unused
  weights, preload only the body font, keep `font-display: swap`).

The overall PageSpeed jump on that site (56 → 98) came mostly from the
general work, not from Devigner UI itself.