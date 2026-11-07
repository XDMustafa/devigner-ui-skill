# Performance & build pitfalls (Vite 8 / Rolldown / lazy loading)

Lessons from shipping a Vite 8 + Devigner UI site to GitHub Pages. Read this
before a production build or when a lazy route shows a blank page.

## Vite 8 uses Rolldown, not Rollup

- `build.rollupOptions.output.chunkFileNames` / `assetFileNames` are not
  honored the way Rollup's are; chunk hashes can drift between local (macOS)
  and CI (Linux) builds. Trust the build output names, don't hardcode them.
- Keep root `index.html` as the Vite **source template**. If a build step
  overwrites it with `dist/index.html`, CI rebuilds the old bundle forever and
  ignores source edits. Deploy `dist/` via CI workflow only.

## `cssCodeSplit: false` leaves a ghost CSS reference

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

## Measured results (example site)

- Icons barrel alias + code-splitting: main bundle 837 KB → 433 KB
  (gzip 263 KB → 138 KB); heavy content (KaTeX) in a separate lazy chunk.
- Static prerender of the shell into `index.html` gives first paint without
  waiting for JS; React replaces it on mount.
- Fonts: convert TTF → WOFF2 with a Persian+Latin subset
  (`pyftsubset --unicodes=U+0600-06FF,…`), drop unused weights, preload only
  the body font, keep `font-display: swap`.