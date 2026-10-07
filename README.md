# Devigner UI — Claude Agent Skill

A [Claude Agent Skill](https://docs.claude.com/en/docs/claude-code/skills) that teaches Claude how to correctly use [**Devigner UI**](https://ui.devigner.cc/) — a shadcn/ui-style library of animated, accessible React components that install as plain `.tsx` files via a CLI.

> ⚠️ **Unofficial.** This is a community-made skill, not affiliated with the Devigner team. It just wraps their public CLI and workflow so Claude uses them the right way.

[![skills.sh](https://skills.sh/b/xdmustafa/devigner-ui-skill)](https://skills.sh/xdmustafa/devigner-ui-skill)

## Why this exists

Devigner UI components aren't imported from a package — they're copied into your repo as editable `.tsx` files via `npx devignerui add <name>`. Without guidance, an AI agent tends to hallucinate component names or try to `import` them before adding them. This skill gives Claude the exact workflow, requirements, and the real list of available components.

## What's inside

```
devigner-ui/
├── SKILL.md      # The skill definition Claude loads
└── README.md     # You are here
```

## Installation (Claude Desktop)

1. Clone or download this repo.
2. Copy the `devigner-ui/` folder into your Claude skills directory:
   - **macOS/Linux:** `~/.claude/skills/`
   - **Windows:** `%USERPROFILE%\.claude\skills\`
3. Restart Claude Desktop.

The folder name and the `name` field in `SKILL.md` must match (`devigner-ui`).

> Also works with Claude Code and the Claude Agent SDK — drop it in the same `skills/` location.

## How it activates

Claude loads the skill automatically when a task matches its `description` — for example when you ask it to build a page with Devigner UI, or when a project already contains a `components.json` and `components/ui` folder created by the CLI.

## Project requirements

Devigner UI components need:

- React 18 or 19
- Tailwind CSS v4
- `motion` (Framer Motion) for animations

Compatible with Next.js (App Router), Vite, TypeScript, and shadcn/ui.

## Quick start (what the skill tells Claude to do)

```bash
# once per project
npx devignerui init

# see what's available
npx devignerui list

# add a component (writes components/ui/<name>.tsx)
npx devignerui add magnetic-button
```

Then import it locally:

```tsx
import { MagneticButton } from "@/components/ui/magnetic-button";
```

## Available components

As of Devigner UI **v1.7.0** (18 components):

`stardust-slider` · `badge` · `image-generation` · `kpi-chart` · `contact-lens` · `copy-button` · `nav-notch` · `liquid-glass` · `slider` · `file-upload` · `goo-switch` · `inline-time-edit` · `date-range-picker` · `chart-card` · `delete-button` · `menu-dock` · `timeline` · `magnetic-button`

Always run `npx devignerui list` for the current set — the library adds components regularly.

## Links

- 🌐 Website & docs: https://ui.devigner.cc
- 🎨 Devigner Icons: https://icons.devigner.cc
- 💬 Discord: https://discord.gg/PSv2aFYhuC
- 🐦 X: https://x.com/devignerui

## Acknowledgements

Parts of this skill were drafted with the help of Anthropic's Claude. Devigner UI is a trademark of its respective owners; this is an unofficial, community-made skill.

## License

MIT for this skill. Devigner UI components are free for personal and commercial use under their own terms — see their site.
