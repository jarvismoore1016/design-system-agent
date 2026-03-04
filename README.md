# Design System Agent

**Give any web project an AI-powered design system assistant in 15 minutes — using Claude Code.**

One command. No dependencies. Works with whatever stack you already have.

---

## What You Get

After running setup, your project will have four permanent tools:

| What | Where | How to use |
|------|-------|------------|
| Design token file | Project root | Reference in your CSS/JS |
| `/audit-ui` command | `.claude/commands/` | Scan for inconsistencies anytime |
| `/generate-component` command | `.claude/commands/` | Build new components on demand |
| `/generate-docs` command | `.claude/commands/` | Document components as you go |
| `CLAUDE.md` | Project root | Persistent design context for every Claude session |

---

## Quick Start

**1. Clone this repo** (or download it) somewhere on your machine:

```bash
git clone https://github.com/jarvismoore1016/design-system-agent.git ~/design-system-agent
```

**2. Copy the setup command into your project:**

```bash
mkdir -p /path/to/your/project/.claude/commands
cp ~/design-system-agent/commands/setup-design-system.md /path/to/your/project/.claude/commands/
```

**3. Open Claude Code from your project root:**

```bash
cd /path/to/your/project
claude
```

**4. Run the setup command:**

```
/setup-design-system
```

Claude will scan your project and walk you through the rest — no additional input needed until it asks you to confirm choices.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated
- Any web project directory
- No build tools, package managers, or extra dependencies required

---

## How It Works

The setup wizard scans your project, then builds everything around what it finds:

```
Your project
    │
    ▼
1. Detect stack          → React, Vue, Svelte, vanilla, etc.
2. Extract design values → every color, spacing, font, border-radius in your code
3. Build token file      → named, organized, ready to reference
4. Create /audit-ui      → scan for inconsistencies on demand
5. Create /generate-*    → build and document components using your tokens
6. Write CLAUDE.md       → persistent context so Claude always knows your system
```

Setup takes 15–20 minutes. Future command runs are fast because they use `CLAUDE.md` instead of rescanning.

---

## Supported Stacks

### Web

- Vanilla HTML + CSS
- React (JavaScript or TypeScript)
- Vue 2 and Vue 3
- Svelte / SvelteKit
- Angular
- Next.js
- Astro
- Nuxt
- Any project with CSS, SCSS, Less, or Stylus

### Mobile Apps

- React Native (JavaScript or TypeScript)
- Flutter (Dart)
- SwiftUI (Swift)
- Kotlin Compose (Jetpack Compose)

---

## Mobile Quick Start

**1. Clone this repo** if you haven't already:

```bash
git clone https://github.com/jarvismoore1016/design-system-agent.git ~/design-system-agent
```

**2. Copy the mobile setup command into your project:**

```bash
mkdir -p /path/to/your/mobile/project/.claude/commands
cp ~/design-system-agent/commands/setup-mobile-design-system.md /path/to/your/mobile/project/.claude/commands/
```

**3. Open Claude Code from your project root:**

```bash
cd /path/to/your/mobile/project
claude
```

**4. Run the setup command:**

```
/setup-mobile-design-system
```

Claude detects your platform (React Native, Flutter, SwiftUI, or Kotlin Compose), extracts design values in the right format for that platform, and creates the same three slash commands and CLAUDE.md as the web setup.

> **Web and mobile use separate setup commands** but produce the same workflow: one token file, three slash commands, one CLAUDE.md. Copy the web command, the mobile command, or both — depending on your project.

---

## After Setup

**`/audit-ui`** — Scan for design system issues

```
/audit-ui                    # Scan entire project
/audit-ui src/components     # Scan a directory
/audit-ui src/Button.tsx     # Audit a single file
```

**`/generate-component`** — Build a new component using your tokens

```
/generate-component button
/generate-component card
/generate-component modal
/generate-component input
```

**`/generate-docs`** — Document a component

```
/generate-docs button
/generate-docs src/components/Card
/generate-docs all           # Document all components
```

---

## Examples

The `examples/` folder shows real output from each command:

**Web**
- `sample-audit.md` — what an audit report looks like
- `sample-component.html` — a generated component
- `sample-docs.md` — generated component documentation

**Mobile**
- `sample-audit-mobile.md` — mobile audit report (React Native)
- `sample-component-rn.tsx` — React Native Button (Pressable, tokens, accessibility)
- `sample-component-flutter.dart` — Flutter AppButton (Semantics, ThemeData access)
- `sample-component-swiftui.swift` — SwiftUI AppButton (ButtonStyle, accessibilityLabel)
- `sample-component-compose.kt` — Compose AppButton (semantics, token references)
- `sample-docs-mobile.md` — mobile component documentation (React Native Button)

The `templates/` folder contains the reference versions of each slash command if you want to understand, customize, or rebuild them.

---

## Troubleshooting

**Setup finds no files** — Make sure Claude Code is opened from your project root, not a subdirectory.

**Tech stack not detected** — Claude will ask you to confirm. Choose "Something's off" and correct it when prompted.

**Token file format is wrong** — Edit the generated file directly, or re-run setup and choose a different naming convention.

**A command produces unhelpful output** — Edit `.claude/commands/[command-name].md` directly. The files are plain markdown — adjust the instructions to match what you need.

**Working in a monorepo** — Point Claude Code at the specific package directory. Run setup once per package that needs its own design system.

**Large codebase takes a long time** — Normal. Token extraction scans every file. Future runs are fast.

---

## License

MIT — Copyright (c) 2025 Jarvis Moore
