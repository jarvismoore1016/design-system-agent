---
description: Set up and maintain Storybook for your design system — scaffold from scratch, sync missing stories, or enrich existing story documentation
allowed-tools: Read, Write, Bash, Glob, Grep, AskUserQuestion
argument-hint: init | sync | docs
---

# Storybook

## Mode Detection

If `$1` is `init` → go to Mode 1 (Init).
If `$1` is `sync` → go to Mode 2 (Sync).
If `$1` is `docs` → go to Mode 3 (Docs).

If `$1` is blank, use AskUserQuestion:
- question: "What would you like to do with Storybook?"
- header: "Storybook mode"
- options:
  - Init — scaffold Storybook from scratch (config + story files)
  - Sync — generate missing stories for components that don't have them yet
  - Docs — enrich existing stories with full controls, descriptions, and variant coverage

---

## Mode 1 — Init (scaffold from scratch)

### Step 1 — Read context and check existing state

Read `CLAUDE.md` for:
- Framework (React, Vue, Svelte, Angular, vanilla HTML)
- Build tool (Vite, Webpack, etc.)
- Token file path
- Component directory paths

Glob for `.storybook/main.ts` and `.storybook/main.js`.

If `.storybook/` already exists, warn the user:

```
.storybook/ already exists in this project.
```

Then use AskUserQuestion:
- question: "A .storybook/ directory already exists. How would you like to proceed?"
- header: "Existing Storybook"
- options:
  - Overwrite — replace existing config files with fresh generated versions
  - Add missing files only — keep existing files, only create what's absent
  - Cancel — stop here, I'll handle this manually

If "Cancel": stop.

### Step 2 — Confirm framework and builder

Show the framework detected from CLAUDE.md. Use AskUserQuestion:
- question: "I detected [framework] with [build tool]. Is that correct?"
- header: "Stack check"
- options:
  - Yes, that's right
  - No, let me correct it

If "No, let me correct it", ask:

**Framework**:
- question: "What framework are you using?"
- header: "Framework"
- options:
  - React
  - Vue 3
  - Svelte
  - Angular
  - Vanilla HTML

**Builder**:
- question: "What build tool are you using?"
- header: "Builder"
- options:
  - Vite
  - Webpack

Derive the correct Storybook adapter:
- React + Vite → `@storybook/react-vite`
- React + Webpack → `@storybook/react-webpack5`
- Vue 3 + Vite → `@storybook/vue3-vite`
- Svelte + Vite → `@storybook/svelte-vite`
- Angular → `@storybook/angular`
- Vanilla HTML + Vite → `@storybook/html-vite`
- Vanilla HTML + Webpack → `@storybook/html-webpack5`

### Step 3 — Ask configuration preferences

Use three separate AskUserQuestion calls:

**Addons** (multiSelect: true):
- question: "Which addons would you like to include? (Essentials is always included.)"
- header: "Addons"
- options:
  - Accessibility (`@storybook/addon-a11y`) — WCAG checks on every story
  - Interactions (`@storybook/addon-interactions`) — test user flows in stories
  - Measure & Outline — spacing visualizer toolbar

**Story file location**:
- question: "Where should story files live?"
- header: "Story location"
- options:
  - Co-located with components (`Button.stories.tsx` next to `Button.tsx`)
  - Separate `/stories` folder at the project root

**Theme support**:
- question: "Which color themes should Storybook support?"
- header: "Theme"
- options:
  - Light only
  - Dark only
  - Both (adds a theme switcher background toggle)

### Step 3b — Pre-write approval gate

After collecting all preferences, build the complete list of files that will be created. Show it:

```
Here's what I'll create:

  .storybook/main.ts            Storybook config (framework, addons, story globs)
  .storybook/preview.ts         Global preview (imports token file, controls, themes)
  [path/to/ComponentName.stories.tsx for each component]
```

Use AskUserQuestion:
- question: "Ready to write these files?"
- header: "Proceed?"
- options:
  - Yes, let's go
  - Show me each file before writing
  - Cancel

If "Cancel": stop.
If "Show me each file before writing": display each file's content in a code block before writing it, then use AskUserQuestion: "Write this file?" (Yes / Skip) for each file.

### Step 4 — Generate `.storybook/main.ts`

Write `.storybook/main.ts` with:
- `stories` glob matching the chosen story file location:
  - Co-located: `'../src/**/*.stories.@(ts|tsx|js|jsx|mdx)'`
  - Separate `/stories`: `'../stories/**/*.stories.@(ts|tsx|js|jsx|mdx)'`
- `addons` array beginning with `'@storybook/addon-essentials'` followed by selected addons
- `framework` set to the derived adapter
- `docs.autodocs: 'tag'`

Example output (React + Vite, co-located, a11y selected):
```ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(ts|tsx|js|jsx|mdx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

### Step 5 — Generate `.storybook/preview.ts`

Write `.storybook/preview.ts` with:
- Import of the project's token file (path from CLAUDE.md) so tokens apply globally
- `controls.matchers` for color and date detection
- `docs.toc: true` for table of contents on docs pages
- `backgrounds` values — include both light and dark entries regardless of theme choice (the theme preference controls which is `default`)

Example (React, both themes):
```ts
import type { Preview } from '@storybook/react';
import '../tokens.css'; // path from CLAUDE.md

const preview: Preview = {
  parameters: {
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
    docs: {
      toc: true,
    },
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#0f172a' },
      ],
    },
  },
};

export default preview;
```

For "Light only": set `default: 'light'`.
For "Dark only": set `default: 'dark'`.
For "Both": set `default: 'light'` (user can toggle in the toolbar).

Adjust the import type and export signature for Vue, Svelte, Angular, or vanilla HTML as appropriate.

### Step 6 — Scan components and ask which to generate stories for

Glob for component files in the directories listed in CLAUDE.md. Common patterns:
- `src/components/**/*.{tsx,jsx,vue,svelte}`
- `components/**/*.{tsx,jsx,vue,svelte}`
- `src/ui/**/*.{tsx,jsx,vue,svelte}`

Build a deduplicated list of component names (strip path and extension).

Filter out any component that already has a `.stories.{ts,tsx,js,jsx}` file alongside it or in the `/stories` folder.

Present the filtered list via AskUserQuestion (multiSelect: true):
- question: "Which components should I generate stories for?"
- header: "Components"
- options: [one option per component name found]

### Step 7 — Generate story files (one at a time with approval)

For each selected component:

1. Read the component source file.
2. Extract from the file:
   - Component name and import path (relative from story file location)
   - All props (from TypeScript interface, type alias, or PropTypes)
   - Default values for props
   - String union types (e.g., `'primary' | 'secondary' | 'ghost'`) → use `control: 'select'`
   - Boolean props → use `control: 'boolean'`
   - String props → use `control: 'text'`
   - Number props → use `control: 'number'`
   - Whether a `loading` prop exists
   - Named variants from the union types

3. Generate the story file content.

4. Display the generated content in a code block.

5. Use AskUserQuestion:
   - question: "Write this story file?"
   - header: "[ComponentName].stories"
   - options:
     - Yes
     - Skip
     - Stop generating

   Only write the file if "Yes". If "Stop generating": stop iterating and go to Step 8.

**Story file format** (TypeScript React example — adapt for Vue, Svelte, etc.):

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { ComponentName } from './ComponentName'; // adjust import path

const meta = {
  title: 'Components/ComponentName',
  component: ComponentName,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost'],
    },
    disabled: { control: 'boolean' },
    children: { control: 'text' },
    // one entry per prop
  },
} satisfies Meta<typeof ComponentName>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Button',
  },
};

// ... one named export per visual variant

export const Disabled: Story = {
  args: {
    variant: 'primary',
    disabled: true,
    children: 'Button',
  },
};

// Include Loading story only if the component has a loading prop
export const Loading: Story = {
  args: {
    variant: 'primary',
    loading: true,
    children: 'Button',
  },
};

export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '8px', flexWrap: 'wrap' }}>
      <ComponentName variant="primary">Primary</ComponentName>
      <ComponentName variant="secondary">Secondary</ComponentName>
      <ComponentName variant="ghost">Ghost</ComponentName>
      <ComponentName disabled>Disabled</ComponentName>
    </div>
  ),
};
```

For JavaScript projects, omit TypeScript syntax and use PropTypes inference instead of interface extraction.

Story file location follows the preference set in Step 3:
- Co-located: write alongside the component file
- Separate `/stories`: write to `/stories/ComponentName.stories.tsx`

### Step 8 — Update CLAUDE.md (with approval)

Compose the exact text to be added to CLAUDE.md under a `## Storybook` heading. Show it:

```
I'll add the following section to CLAUDE.md:

---
## Storybook
- **Config**: `.storybook/`
- **Run**: `npm run storybook` (port 6006)
- **Story files**: `src/components/**/*.stories.tsx` [or `/stories/`]
- **Framework**: [derived adapter]
- **Addons**: essentials[, a11y][, interactions][, measure]
- **Token integration**: token file imported in `.storybook/preview.ts`
---
```

Use AskUserQuestion:
- question: "Add this Storybook section to CLAUDE.md?"
- header: "Update CLAUDE.md"
- options:
  - Yes
  - Skip

Only write if "Yes." Append the section to the end of CLAUDE.md.

### Step 9 — Print install instructions and summary

Print the install command for all selected packages — do NOT run it. The user runs this in their terminal.

```
Storybook scaffolded. Here's what was created:

  .storybook/main.ts          Storybook config (framework, addons, story globs)
  .storybook/preview.ts       Global setup (token import, controls, themes)
  [list each story file written]

To get started:

  1. Install packages:
     npm install --save-dev [derived adapter] @storybook/addon-essentials [selected addons]

  2. Add scripts to package.json:
     "storybook": "storybook dev -p 6006",
     "build-storybook": "storybook build"

  3. Run:
     npm run storybook

Storybook will open at http://localhost:6006
```

---

## Mode 2 — Sync (generate missing stories)

### Step 1 — Verify Storybook exists

Read `CLAUDE.md`.

Glob for `.storybook/main.ts` and `.storybook/main.js`.

If neither exists, print:

```
No .storybook/ directory found in this project.

Run /storybook init to scaffold Storybook from scratch, then run /storybook sync
to keep stories in step with your components going forward.
```

Stop.

### Step 2 — Compare components to existing stories

Glob for component files in the component directories from CLAUDE.md.

Glob for existing story files: `**/*.stories.{ts,tsx,js,jsx}`.

For each component, check whether a corresponding story file exists (match by component name, case-insensitively).

**Outdated story detection**: For each component that does have a story file:
1. Read the component file and extract its prop names.
2. Read the story file and extract the keys in `argTypes`.
3. If any prop from the component is absent from `argTypes`, flag the story as "possibly outdated".

### Step 3 — Present the sync report

```
Storybook Sync Report

Missing stories ([N] components have no story file):
  - ComponentA
  - ComponentB
  - ...

Possibly outdated (component has props not in the story's argTypes):
  - Button — `loading` prop not in argTypes
  - Input — `error` prop not in argTypes

Stories that look current: [list]
```

### Step 4 — Ask which to generate or update

Use AskUserQuestion (multiSelect: true):
- question: "Which components would you like to generate or update stories for?"
- header: "Components to sync"
- options: [one per missing or outdated component]

### Step 5 — Generate selected stories (one at a time with approval)

For each selected component, follow the same generation process as Init Mode Step 7:

1. Read the component source file.
2. Generate the story content (using the same format and approval gate).
3. Display in a code block.
4. Use AskUserQuestion: "Write this story file?" (Yes / Skip / Stop generating).
5. Only write if "Yes."

For outdated stories: generate a complete replacement of the story file using all current props.

### Step 6 — Summary

```
Sync complete.

  [N] stories generated
  [N] stories updated
  [N] stories skipped
```

---

## Mode 3 — Docs (enrich existing stories)

### Step 1 — Verify Storybook exists

Read `CLAUDE.md`.

Glob for `.storybook/main.ts` and `.storybook/main.js`.

If neither exists:

```
No .storybook/ directory found in this project.

Run /storybook init to scaffold Storybook, then /storybook docs to enrich the stories.
```

Stop.

### Step 2 — Find all existing story files

Glob for `**/*.stories.{ts,tsx,js,jsx}`.

For each story file, read it and check:
- Is `tags: ['autodocs']` set on the meta object?
- Do all props in `argTypes` have a `description` field?
- Does a `Playground` or `AllVariants` story exist?
- Is `parameters.docs.description.component` set in the meta?

Also check `docs/components/[name].md` if it exists — extract any description text to reuse.

### Step 3 — Present enrichment opportunities

For each story file, note what's missing:

```
Enrichment Opportunities

Button.stories.tsx
  ✗ Missing: autodocs tag
  ✗ Missing: argType descriptions (3 props undescribed)
  ✗ Missing: component description in meta
  ✗ Missing: AllVariants story

Card.stories.tsx
  ✓ autodocs tag present
  ✗ Missing: argType descriptions (2 props undescribed)
  ✗ Missing: Playground story

Badge.stories.tsx
  ✓ All enrichments already present — skipping
```

### Step 4 — Ask which stories to enrich

Use AskUserQuestion (multiSelect: true):
- question: "Which component stories should I enrich?"
- header: "Enrich stories"
- options: [one per story file that has at least one missing enrichment]

### Step 5 — Enrich selected stories (one at a time with approval)

For each selected story file:

1. Read the component source file to get TypeScript comments or JSDoc descriptions for each prop.
2. Check `docs/components/[name].md` for an existing component description to use.
3. Compose the proposed changes:
   - Add `tags: ['autodocs']` to the meta if missing
   - Add `description` to each `argTypes` entry (from TypeScript comments, JSDoc, or the docs markdown)
   - Add or update `parameters.docs.description.component` with 2–3 sentences describing the component
   - Add an `AllVariants` story rendering all variant combinations if it doesn't exist
   - Add a `Playground` story with all controls exposed if it doesn't exist

4. Show the proposed changes in a diff-style code block:

```
Proposed changes for Button.stories.tsx:

+ tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost'],
+     description: 'Visual style of the button.',
    },
    disabled: {
      control: 'boolean',
+     description: 'When true, the button is non-interactive and visually muted.',
    },
  },
+ parameters: {
+   docs: {
+     description: {
+       component: 'The Button component triggers actions. Use Primary for the main call to action, Secondary for supporting actions, and Ghost for low-emphasis actions.',
+     },
+   },
+ },

+ export const Playground: Story = {
+   args: {
+     variant: 'primary',
+     children: 'Button',
+   },
+ };
+
+ export const AllVariants: Story = {
+   render: () => (
+     <div style={{ display: 'flex', gap: '8px', flexWrap: 'wrap' }}>
+       <Button variant="primary">Primary</Button>
+       <Button variant="secondary">Secondary</Button>
+       <Button variant="ghost">Ghost</Button>
+       <Button disabled>Disabled</Button>
+     </div>
+   ),
+ };
```

5. Use AskUserQuestion:
   - question: "Apply these changes to [ComponentName].stories.tsx?"
   - header: "Apply enrichments"
   - options:
     - Yes
     - Skip
     - Stop enriching

   Only write if "Yes." When writing, apply the changes to the existing story file (do not replace the whole file — add only the missing pieces).

### Step 6 — Summary

```
Docs enrichment complete.

  [N] stories enriched
  [N] stories skipped

Properties documented: [total count across all enriched files]
```
