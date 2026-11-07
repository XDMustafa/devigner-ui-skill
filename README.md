# Devigner UI — AI Agent Skill

An agent skill (originally written for [Claude Agent Skills](https://docs.claude.com/en/docs/claude-code/skills), and used with Hermes and other agents too) that teaches an AI coding agent how to correctly use [**Devigner UI**](https://ui.devigner.cc/) — a shadcn/ui-style library of animated, accessible React components that install as plain `.tsx` files via a CLI.

> ⚠️ **Unofficial.** This is a community-made skill, not affiliated with the Devigner team. It just wraps their public CLI and workflow so any agent uses them the right way.

[![skills.sh](https://skills.sh/b/xdmustafa/devigner-ui-skill)](https://skills.sh/b/xdmustafa/devigner-ui-skill)

## install (Skills.sh)

```
npx skills add https://github.com/xdmustafa/devigner-ui-skill --skill devigner-ui
```

## Why this exists

Devigner UI components aren't imported from a package — they're copied into your repo as editable `.tsx` files via `npx devignerui add <name>`. Without guidance, an AI agent tends to hallucinate component names, guess prop names, or try to `import` them before adding them. This skill gives the agent the exact workflow, requirements, the real component list, official per-component reference URLs, and field notes from production builds.

## What's inside

```
devigner-ui-skill/          (rename to devigner-ui/ when installing)
├── SKILL.md                # The skill definition the agent loads
├── README.md               # You are here
└── references/
    └── performance.md      # Vite 8 / Rolldown / lazy-loading pitfalls (read before production builds)
```

## Installation

1. Clone or download this repo.
2. Copy its contents into a `devigner-ui/` folder inside your agent's skills directory:
   - **Claude:** `~/.claude/skills/devigner-ui/` (Windows: `%USERPROFILE%\.claude\skills\`)
   - **Hermes:** `~/.hermes/skills/devigner-ui/`
   - Other agents: their equivalent skills/plugins directory.
3. The folder name and the `name` field in `SKILL.md` must match (`devigner-ui`).

## How it activates

The agent loads the skill when a task matches its `description` — for example when you ask it to build a page with Devigner UI, or when a project already contains a `components.json` and `components/ui` folder created by the CLI.

## Project requirements

Devigner UI components need:

- React 18 or 19
- Tailwind CSS v4
- `motion` (Framer Motion) for animations

Compatible with Next.js (App Router), Vite, TypeScript, and shadcn/ui.

## Quick start (what the skill tells the agent to do)

```bash
# once per project
npx devignerui init

# see what's available
npx devignerui list

# add a component (writes components/ui/<name>.tsx)
npx devignerui add magnetic-button
```

Then read the component's official guide (props, keyboard, troubleshooting) before using it:

```text
https://ui.devigner.cc/llms/<component-name>.md
```

And import it locally:

```tsx
import { MagneticButton } from "@/components/ui/magnetic-button";
```

## Available components

As of Devigner UI **v1.9.0** (28 components):

**Marketing (7):** `flick-cards-slider` · `menu-notch` · `rating` · `flip-button` · `contact-lens` · `timeline` · `magnetic-button`

**Interface (21):** `command-palette` · `context-menu` · `dynamic-island` · `morphing-popover` · `tabs` · `expand-tabs` · `stardust-slider` · `badge` · `image-generation` · `kpi-chart` · `copy-button` · `nav-notch` · `liquid-glass` · `slider` · `file-upload` · `goo-switch` · `inline-time-edit` · `date-range-picker` · `chart-card` · `delete-button` · `menu-dock`

Recently updated upstream: `liquid-glass`, `file-upload`, `date-range-picker`, `chart-card`, `menu-dock`.

Always run `npx devignerui list` for what **your installed version** ships — it lags the website when a project pins an older `devignerui`. When in doubt, check https://ui.devigner.cc/components or use `npx devignerui@latest list`.

## Links

- 🌐 Website & docs: https://ui.devigner.cc
- 📄 Per-component LLM guides: `https://ui.devigner.cc/llms/<name>.md` (all-in-one: `https://ui.devigner.cc/llms-full.txt`)
- 🎨 Devigner Icons: https://icons.devigner.cc
- 💬 Discord: https://discord.gg/PSv2aFYhuC
- 🐦 X: https://x.com/devignerui

## Acknowledgements

Parts of this skill were drafted with the help of Anthropic's Claude. The field notes, performance references, and later revisions were contributed by **Hermes Agent (mimo, Nous Research)** through real production work on a Devigner UI site. Devigner UI is a trademark of its respective owners; this is an unofficial, community-made skill.

## License

MIT for this skill. Devigner UI components are free for personal and commercial use under their own terms — see their site.