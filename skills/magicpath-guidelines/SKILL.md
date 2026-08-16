---
name: magicpath-guidelines
description: Guidelines for working with MagicPath (the AI design canvas) through its MCP server. Use whenever a task involves MagicPath designs or components, projects or canvases, themes/design systems, teams, canvas selections, share links, or MagicPath skills — including installing or exporting MagicPath components into local code, making local UI match a design 1:1, recreating local files or Git repositories on the MagicPath canvas, and creating or editing interactive canvas components.
compatibility: Requires the MagicPath MCP server provided by this plugin (network access). A browser enables OAuth sign-in and canvas viewing.
metadata:
  author: MagicPathAI
  source: https://github.com/MagicPathAI/magicpath-agent-plugin
---

# MagicPath

MagicPath is a multiplayer design canvas whose components are real Vite + React + TypeScript mini-apps. All platform operations go through the `magicpath` MCP server that ships with this plugin.

**Vocabulary:** users say "design" for a MagicPath component, "theme" for a design system, and "team" or "workspace" for a MagicPath team. Treat these as interchangeable.

## Where knowledge lives

- **The server owns all mechanics.** Tool names, parameters, session lifecycle, editable-file boundaries, upload/download transport, guest access, and retry behavior are defined by the live tool list and the server's own instructions. Read a tool's current description before calling it; never work from memory of an older tool surface.
- **If your client did not surface the server's usage instructions**, read the MCP resource `magicpath://guide` before any state-changing work, and `magicpath://host-guidance` before local-file handoffs. If resources are unreachable, the tool descriptions carry the critical mechanics.
- **This skill owns what the server cannot know:** which direction a request travels, how to handle files and repositories on this machine, and the design bar for canvas work.
- **On any conflict between this skill and the live server, the server wins.**

## First step

Call `get_context` to verify auth and learn the user's identity, teams, and capability map. On an authorization error, have the user authenticate the `magicpath` server through the client's own MCP authorization flow, then retry. Never ask for raw tokens or attach credentials yourself.

## Route by direction

Direction confusion is the main failure mode. Decide the direction first; load only what that direction needs.

| The user wants | Direction | Do |
|---|---|---|
| Use, install, or export a MagicPath design in local code; make local UI match a design; translate a design to another framework | MagicPath → code | Read [Using MagicPath designs in local code](references/using-magicpath-designs-in-local-code.md) before writing any files |
| Create or edit a design on the canvas, including recreating local or repository UI in MagicPath | code → MagicPath | Apply the Design Defaults below; for recreating existing code or repos, read [Bringing code to the canvas](references/bringing-code-to-the-canvas.md) first |
| Browse, search, share, or answer questions about projects, components, teams, themes, images, or MagicPath skills | read-only | Call the tools directly; no reference needed |

Never mix directions. Install/export/inspect tools read MagicPath → code; code sessions write code → MagicPath. Never start a code session to serve an export, and never install source to serve a canvas edit.

## Ground rules

- **The server never touches this machine.** Every local file is written by you from tool results; every canvas change is content you pass into the tools.
- **Resolve "this / selected / open / current" with the selection and active-project tools** before asking for an id. Carry the returned `selectedRevisionId` into revision-aware calls so you operate on the version the user is looking at.
- **Look before you recommend.** Component results include `previewImageUrl` — download and view it to understand what a design looks like.
- **Other people's personal projects are invisible.** A teammate's work lives only in team projects; never search personal scope for it.
- **One authoring session per working directory.** Parallel sessions sharing a directory overwrite each other. Independent frames get independent directories and can be built concurrently.
- **A failed build is not a dead end.** Fix the files and resubmit the same session, following the retry guidance in the tool result. Never create a new component to escape a build failure.
- **Share one link at a time**, resolved through the share-URL tool. Never guess URL shapes.

## Two gates — stop and wait for the user

1. **Target gate (MagicPath → code).** Unless the user gave an exact `generatedName` or the target came from their canvas selection, present the matched component (name, project, preview) and stop for confirmation before exporting, installing, or replacing anything.
2. **Frame gate (code → MagicPath).** When a request spans multiple screens and the split is ambiguous, ask: one interactive component with internal navigation, or separate frames per screen? Stop until answered.

These are the only mandatory stops. Everything else proceeds autonomously.

## Design Defaults (canvas authoring)

Every component you create or edit on the canvas follows these rules unless the user explicitly overrides them. They do not apply to the install direction.

1. **No device mockups.** Never wrap a design in phone/browser/laptop chrome, status bars, or notches unless explicitly requested. The canvas is the frame; a mobile-sized canvas is not a request for a phone mockup.
2. **Responsive always.** Fluid outer sizing (`w-full`, `max-w-*`, flex/grid, breakpoints). No hardcoded pixel dimensions on outer containers; only intrinsically fixed elements (avatars, icons, media) are exempt.
3. **Centered.** The root centers itself in its frame — horizontally, and vertically when short. Never corner-stuck, never overflowing.
4. **Canvas size signals form factor, content stays fluid.** Pass width/height that fit the target (e.g. 390×844 mobile, 1440×900 desktop), but the component must adapt if placed in a different container.
5. **One screen per frame.** Multi-view flows (login → signup, wizards, tabbed settings) are ONE component with state-driven navigation. Truly independent screens are separate components in separate sessions and working directories. Never stack screens side-by-side in one frame — that artifact is broken. Unsure → frame gate.
6. **Fully interactive.** Controlled inputs, working buttons, tabs that switch, modals that open and close, forms that validate, styled hover/focus/active/disabled states, deliberate transitions. You are a design engineer, not a screenshot generator; static markup of an interactive surface is not done. Mock data locally so nothing renders empty.

Craft bar: deliberate visual hierarchy, strong typography, restrained color, semantic HTML, visible focus styles, and polished loading/empty/error states.

## Embedded browser as a canvas

When the host has an embedded browser and the user is working visually, keep the **project** canvas open beside you: resolve the URL with the share-URL tool and navigate there. Do not open individual design previews unless explicitly asked — submitted work appears on the project canvas. When creating a new project for visual work, open its canvas before authoring into it. MCP authorization and the browser pane's login are separate sessions: if the project URL redirects to sign-in, ask the user to sign in inside that pane, then reopen the same URL. Do not open a browser for background or read-only work.

## Themes

A theme returned by the theme tools contains `light`/`dark` CSS-variable maps, optional `fonts`, and an optional `prompt` holding the designer's styling instructions — follow that prompt. Apply the variables instead of hardcoded values, load the fonts, and for non-web targets translate the values into native tokens (SwiftUI colors, Android theme XML, etc.) rather than pasting CSS.

## MagicPath skills, locally

To install a MagicPath skill into a local agent: fetch it with the skill tools, create a folder named after its slug, write `SKILL.md` (frontmatter `name` + `description`, body = its instructions), and recreate any bundled files at their relative paths. Ask before writing outside the current project or into global agent configuration.
