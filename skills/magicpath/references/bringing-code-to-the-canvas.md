# Bringing Code to the Canvas

**Load this when** the user wants UI that already exists — a local path, a component in their app, or an online Git repository — reproduced on their MagicPath canvas ("bring my sidebar into MagicPath", "render this project in MagicPath", "recreate my landing page here"). This is the **inverse** of installing: the repository is the source of truth, the canvas is the destination, and the work goes through the code-session authoring tools. Never use install/export/inspect tools for this direction.

A MagicPath canvas component is a single, self-contained, interactive React + Tailwind v4 mini-app. The goal is fidelity: the canvas result should look and behave like the same product, not a reinterpretation. The skill's Design Defaults apply throughout.

## Get the code

- **Local path:** confirm it if not explicit; read files directly. Detect the repo root (`package.json`, `.git/`, framework config) rather than assuming.
- **Online repo:** `git clone --depth 1 [--branch <branch>] <url>` into a scratch directory. For a link to one file or folder, fetch just that slice plus its imports. Private repos need the user's credentials — ask, never guess.
- **Monorepos:** scope to the package/app the user means; don't recreate the whole workspace.
- Keep the cloned repo (read-only input) separate from the authoring working directory (your draft).

## Read the design foundation first

Faithful recreation comes from the global design layer, not the target file alone:

1. Stack and styling approach from `package.json` (framework; Tailwind / CSS Modules / styled-components / Sass / plain CSS).
2. Global stylesheets (`globals.css`, `index.css`, `app.css`, …) — resets, base typography, CSS custom properties.
3. Design tokens with their **actual values**: `tailwind.config.*` theme/extend, `:root` and `.dark` variables, token files.
4. Fonts (how they load, which family maps to body vs. headings) and the theming strategy (dark-mode handling, providers).
5. Shared UI primitives (`components/ui`, `design-system/`) that the target composes.

If the user has a MagicPath theme matching this brand, fetch it and reconcile against its variables, fonts, and `prompt`; otherwise derive tokens straight from the repo.

## Resolve the target

- **A single element** ("the sidebar"): locate the file, trace all imports (children, icons, styles, data), and read the layout parent that gives it width, position, and spacing — reproduce that container behavior inside the frame, or the element will look wrong. Resolve every class and variable to concrete values; match the rendered result, not class strings.
- **A whole page/project:** identify the entry screen (ask and stop if ambiguous), walk the component tree, and decide the frame split per the Design Defaults — one cohesive flow becomes one stateful component; genuinely independent screens become separate sessions, each with its own working directory, built concurrently. Confirm the split at the **frame gate** before building many frames.

## Build

1. Resolve a project (active project if one is open; otherwise ask or create one). Start the code session before writing files so the user sees agent presence — follow the server's session mechanics exactly as its tool descriptions and instructions state them. If the project wasn't already open, give the user its canvas link now (see the skill's canvas-first rule) so they watch the build live.
2. Fill in the scaffold the session returns. Implementation lives in `src/components/generated/`, split into sibling files for sub-components. Do not rewrite `src/App.tsx` beyond its theme value.
3. Translate the source's styling into the canvas's Tailwind v4 conventions: CSS variables move into the existing `:root`/`.dark`/`@theme` blocks of `src/index.css` and are referenced as utilities; CSS-in-JS and CSS Modules flatten into utilities or appended plain CSS; a source Tailwind v3 config becomes `@theme` tokens (the canvas has no `tailwind.config.js`). Add font `@import`/`@font-face` declarations to `src/index.css`.
4. Match visual details precisely — colors, spacing, radii, shadows, type family/size/weight/line-height, transitions — using the concrete values you resolved above.
5. **Reproduce behavior, not a screenshot:** collapsible things collapse, active states are state-driven, dropdowns work. Replace server data and app context with realistic local mock data and stubs so the component is self-contained and looks populated.
6. Assets are files staged for the session (per the server's asset rules) and referenced from code or CSS — never hotlinked repo blobs, never inline base64.
7. Non-React sources (Vue, Svelte, Angular, HTML, SwiftUI): translate the markup semantics, rendered output, and behavior into React + Tailwind — never the original framework's constructs.
8. Submit through the session and handle build failures by fixing files and resubmitting the same session.

## Verify

Compare the canvas result against the source side by side (project canvas or share URL): tokens resolved with no stray defaults, fonts loaded, dark mode matches where relevant, interactions behave like the original, and the component stays centered and intact at the target width while degrading sensibly when resized. Then hand off per the skill's canvas-links rule: the project URL plus a named share link for each component created.

## Boundaries

- Don't dump a whole repo onto the canvas — scope to the ask.
- Don't submit files outside the editable surfaces the server defines.
- Don't pixel-lock or add device chrome even when the source has it — same look, fluid sizing.
- Drop nothing silently: if a piece of the source UI can't be reproduced, say so.
