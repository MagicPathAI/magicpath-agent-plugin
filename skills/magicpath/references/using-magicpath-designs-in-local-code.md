# Using MagicPath Designs in Local Code

**Load this when** taking a design out of MagicPath: exporting it to a folder, installing it in an app, making existing local UI match it, or translating it to another framework. The MagicPath component and revision are the source of truth for **presentation**; the local application is the source of truth for **behavior and integration contracts**. Preserve both.

All MagicPath data comes from the server's read-only capabilities (selection, component inspection, install plans, code context at a revision, project export, share URLs). The server writes nothing locally — every file on disk is written by you from tool results. Resolve exact tools and parameters from the live tool list.

## The fidelity contract

"Use this design" or "make my component match this" means **parity by default**: same structure, geometry, colors, gradients, typography, radii, shadows, assets, icon characteristics, states, motion, and responsive intent — not the nearest local design-system equivalents. When requirements compete, apply this precedence:

1. The user's explicitly requested changes.
2. Required local behavior and accessibility.
3. The MagicPath revision's presentation and interactions.

If these cannot coexist, name the conflict and stop for direction. And 1:1 never means screenshot tracing — the fetched source is the implementation baseline; rebuilding from a screenshot when source is available creates avoidable drift.

## Resolve source and destination

- Canvas selection or an exact `generatedName` is already an explicit choice — use it, including the selected revision id. Otherwise search, view preview images, and pass the **target gate** (confirm with the user) before writing anything.
- Distinguish destinations before acting: a **source-only export** (fetch exact revision source, write into an empty directory), an **install into a React/TS app** (fetch the install plan, write files, install dependencies, then import and integrate), a **replacement of existing local UI** (exact source staged in a temporary directory, then parity-first swap), or a **non-React translation** (exact source as reference only — never install React files into a Swift/Vue/Python project).
- When revision fidelity matters (the user is looking at a specific revision), use the revision-aware source fetch, not the registry-style fetch. Fetching source is read-only — it creates no canvas state; never start a code session for an export.
- Keep a short internal contract before editing: exact component + revision, reference viewport, local target, behavior to retain, and a **closed list** of user-requested differences. Unplanned visual differences allowed: none.
- Exporting several designs: one empty temporary directory per component — exports share paths like `src/App.tsx` and `src/index.css` and will overwrite each other.
- Never write exported files into an app root where they could clobber entry/style files. Exported files are component source, not a runnable standalone app; say so, or build the host project if the user asked for something runnable.

## Understand the local runtime contract

Read the target before replacing anything: framework, package manager, styling system, global CSS/resets/theme providers/fonts, the parent layout and every caller, and the existing component's full contract — props, callbacks, state, API calls, routing, validation, loading/empty/error/permission states, keyboard behavior and ARIA, analytics, tests. The existing local component is the **behavioral specification**; the MagicPath component is the **visual specification**. Build the replacement under a new path and keep the old implementation until verification passes.

## Parity before adaptation

Establish a faithful baseline first — it separates source drift from integration bugs:

1. Render the MagicPath component in the target environment with minimal transformation: all generated files, required CSS, exact theme variables, fonts, icons, images, dependencies.
2. Use representative data matching the MagicPath preview; compare at the reference viewport before any refactoring or design-system mapping.
3. Fix environmental differences at their source (parent layout, resets, font loading, theme class, stacking context) — never hide a parent mismatch with child offsets.
4. Styling translation: Tailwind v4 targets keep the utilities; Tailwind v3 targets translate `@theme` values into config or concrete CSS; CSS Modules/Sass/CSS-in-JS targets translate to equivalent concrete styles after resolving what every utility computes to. Reuse a local token or primitive **only when its computed output is identical** — "close enough" violates parity.
5. Do not clean up unusual values until the render matches; then refactor only if output is unchanged.

## Integrate behavior without losing the design

Once the baseline matches: define props for real data and handlers, replace mock content while retaining layout behavior for long/short/missing/loading values, wire actions to existing application logic, preserve validation/errors/pending states/analytics/accessibility, and bridge the old component's public interface so callers barely change. Keep canvas-defined interactions (make them app-controlled rather than deleting them). Maintain the exact reference render while adapting gracefully at narrower and wider viewports — never fix overflow by shrinking type or hiding content at the reference size.

Requested changes are a **narrow delta**: implement each one in isolation, re-verify that differences appear only where expected, and report intentional deviations. "Make it production-ready" adds robustness around the design; it never redesigns it.

## Install craft (React/TS apps)

- Install only what will be imported and rendered; reading source is free and writes nothing.
- After installing, import the component directly and adapt it in place — never copy its JSX/styles into a parent and leave a dead duplicate.
- Installed files are code the user owns: edit them to accept the props, callbacks, and state the app needs.
- A design artifact becomes production code through you: fixed widths → responsive sizing, static text → props, placeholder handlers → real ones, hardcoded lists → mapped data, plus loading/error/empty states and accessibility.

## Definition of done

- The exact intended component and revision were used; the render matches MagicPath at the reference viewport and theme.
- Fonts, icons, assets, tokens, effects, and interactive states are present; narrower and wider viewports behave intentionally.
- All required local behavior still works (data, callbacks, navigation, validation, states, accessibility, analytics); the component is actually imported and rendered.
- The app builds, relevant checks pass, and browser verification shows no missing resources, console errors, or unapproved visual differences.
- Every difference from MagicPath is either invisible integration plumbing or explicitly requested — and the handoff lists source, files changed, verification done, and intentional deviations.
