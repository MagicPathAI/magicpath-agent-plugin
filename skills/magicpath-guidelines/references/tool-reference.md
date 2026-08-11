# MagicPath MCP Tool Reference

The `magicpath` MCP server (remote, Streamable HTTP) is the single integration point for the MagicPath platform. This reference describes its capability surface: what each capability does, the data you can expect back, and the conventions that apply across all of them.

> **IMPORTANT — resolve capabilities against the live tool list.** Tool names and parameter schemas are defined by the server and may evolve; the names below are MagicPath's stable capability vocabulary (a capability written here as `list-components` may surface as e.g. `list_components`). Before calling anything, read the server's current tool list and schemas. If a capability described here is missing from the list, it isn't available in this server version — say so rather than improvising with a different tool.

## Conventions

- **Structured results.** Every tool returns structured data. There are no output-format flags or interactive prompts.
- **Workspace filters.** Discovery capabilities accept a team filter (team name or ID) and a personal-only filter. Default scope is **all** workspaces (personal + every team).
- **Pagination.** List capabilities use limits, offsets, or cursors as defined by their schemas. `list-components` uses cursor-based pagination: pass the previous page's `lastId`-style cursor to fetch the next page.
- **Identifiers.** Components have a `generatedName` (e.g. `wispy-river-5234`) used by registry-style capabilities, plus a numeric component ID and revision IDs used by canvas/authoring capabilities. Projects and teams have numeric IDs; teams also resolve by case-insensitive name.
- **Preview images.** Component results include `previewImageUrl` — download and view these to understand what a component looks like.

## Authentication

The server is an OAuth-protected resource:

- Resource metadata: `https://staging.api.magicpath.ai/.well-known/oauth-protected-resource/mcp`
- Scopes: `magicpath:read`, `magicpath:write`

Authorization is entirely client-managed. When calls fail with an authorization error (the HTTP layer returns `401` with a `WWW-Authenticate` challenge), the user must authenticate the `magicpath` server through their MCP client's authorization flow; then retry. Never ask for raw tokens, never attach credential headers, and never treat an auth failure as a bug in the plugin — per the Agent Plugins spec it is a connection failure for the server.

## Identity and context

### `whoami` / `info` — Auth and workspace context

Returns auth status, the current user, teams (with the user's role in each), and accessible projects. Call this first in any session; if it fails with an authorization error, stop and have the user authenticate the server in their client.

### `selection` — Current canvas selection

Returns the component(s) and image(s) currently selected in the MagicPath web app canvas, along with the project(s) the user has open. Empty `components`/`images` means nothing of that type is selected; empty `projects` means no canvas is open.

Shape:

```json
{
  "projects": [{ "id": "...", "name": "...", "ownerType": "personal|team", "ownerName": "..." }],
  "components": [{ "id": "...", "name": "...", "generatedName": "...", "clientId": "...", "projectId": "...", "projectName": "...", "selectedRevisionId": "..." }],
  "images": [{ "id": "...", "shapeId": "...", "name": "...", "projectId": "...", "projectName": "...", "width": 0, "height": 0 }]
}
```

Notes:

- `projects` is the same shape returned by `active-project` — calling `selection` gives you both signals in one round-trip.
- `selectedRevisionId` identifies the revision currently displayed for that component. Pass it to revision-aware capabilities (such as the revision-aware source fetch) so an export or inspection cannot drift to another revision.
- `components` may be empty while `images` or `projects` are non-empty. Use that to decide whether to start an authoring session with selected image context, or fall back to listing/searching components.
- If only the open project is needed (not the selection), prefer `active-project` — it is lighter.

### `active-project` — The project(s) the user has open

Returns the project(s) currently open in the MagicPath web app: `{ projects: [{ id, name, ownerType, ownerName }] }`. Multiple entries can be returned if the user has multiple tabs open; an empty list means no active canvas session. If a project is open but cannot be resolved against the user's accessible projects, the entry has only `id` populated and `name`/`ownerType`/`ownerName` set to `null`.

## Teams and people

### `list-teams`

Lists all teams the user belongs to, with their role in each: `{ teams: [{ id, name, role }] }`.

### `list-members`

Lists all members of a specified team (required team parameter, name or ID): `{ team: { id, name }, members: [{ id, displayName, email, role }] }`. Use it to resolve a person's name to their user ID, then use the created-by filter on `list-components` to find their work.

## Discovery

### `search` — Search components across projects

Case-insensitive substring match on component names across all accessible projects. Supports a result limit and team/personal filters. Each result includes project and workspace context (`ownerType`, `ownerName`) and `previewImageUrl`.

### `list-projects`

Lists accessible projects (personal + team by default; filterable). Returns `{ projects, pagination: { total, limit, offset, hasMore } }`. Each project includes `ownerType`, `ownerName`, and `createdBy` (`{ id, displayName }` or null).

### `list-components`

Lists a project's components with cursor-based pagination: `{ components, pagination: { limit, hasNext, lastId } }`. Supports sort by `name` or `createdAt` (asc/desc) and a created-by user filter. Each component includes `previewImageUrl` (string or null) and `lastEditedBy` (`{ id, displayName }` or null).

## Projects

### `create-project`

Creates a project in the user's personal workspace, or in a team when a team is passed (the user must be a member). The name is optional (auto-generated placeholder if omitted) — always pass it when the user named the project. Returns `{ project }` with `id`, `name`, `ownerType`, `ownerName`, `visibility`, etc. Visibility defaults: personal → `PRIVATE`, team → `SHARED`. Use `project.id` for `code start` when adding the first component.

## Links

### `share` — Shareable URL for a component or project

Returns the canonical URL without opening anything: `{ type: "component", url, generatedName }` or `{ type: "project", url, projectId }` (project URLs have the shape `https://www.magicpath.ai/files/<projectId>`). Accepts a component `generatedName` or a project ID.

There is no capability that opens the user's browser — the server is remote. To show something, give the user the URL or navigate the host's embedded browser to it (see [Working with embedded browsers](working-with-embedded-browsers.md)). Present one target at a time.

## Component source (read-only)

### `inspect`-style fetch — Source by `generatedName`

Returns the component's full source without side effects: `{ component, generatedName, files: [{ path, name, content }], dependencies, importStatement?, usage? }`. Works regardless of the target project type; for non-React targets it is the reference for translation.

### Revision-aware fetch — Exact source at a revision (`code context`)

Returns the exact editable source (`src/App.tsx`, `src/index.css`, `src/components/generated/**`) for a component at a specific revision, defaulting to the component's currently selected revision. Pass the `selectedRevisionId` from `selection` whenever the user is looking at a specific revision so the export cannot drift.

This fetch is intentionally read-only: it does not create a pending revision, does not show canvas presence, and does not start an authoring session. Use it as the revision-safe export path when taking a design out of MagicPath; use `code start` instead when the goal is to edit the component on the canvas.

### Installing fetched source (agent-side)

Installation is not a server capability — **you** perform it from fetched source, and only in React/TypeScript projects where the component will be imported and rendered:

1. Fetch the source (registry fetch by `generatedName`, or the revision-aware fetch when an exact revision matters).
2. Write each returned file under `src/components/magicpath/<component>/`, preserving relative paths. Do not overwrite existing files without reviewing them first.
3. Install the returned `dependencies` with the project's package manager.
4. Import the component using `importStatement` / `usage` and integrate it.

To check what's already installed, scan `src/components/magicpath/` in the project (there is no server-side record of local installs).

## Themes (design systems)

### `list-themes`

Lists design systems for the current user, or for a team: `{ themes: [{ id, name, isPublic, createdAt, updatedAt }] }`.

### `get-theme`

Fetches a full theme definition by numeric ID or case-insensitive name (team lookup supported):

```json
{ "id": 1, "name": "...", "theme": { "light": { "--background": "#ffffff" }, "dark": { "--background": "#0a0a0a" } }, "defaultTheme": "light", "prompt": "...", "fonts": [], "version": 1 }
```

Key fields for agents:

- `theme.light` / `theme.dark` — CSS variable maps to apply to components
- `prompt` — natural-language styling instructions from the designer (e.g., "use rounded corners, prefer shadows over borders")
- `fonts` — font metadata with source (`google` or `custom`) and weight URLs
- `defaultTheme` — whether the theme defaults to `"light"` or `"dark"`

## Canvas images

Standalone images that live directly on a project's canvas (alongside components). This is distinct from the `assets/` staging surface in the authoring flow, which holds build inputs for a single component — these capabilities operate on the project canvas itself.

### `image list`

Lists the images on a project's canvas: `{ images: [{ id, name, url, position: { x, y, z, width, height }, createdAt, updatedAt }] }`. Each `url` is a public image URL — download it to visually inspect the image, the same way `previewImageUrl` is used for components.

### `image add`

Places an image on the project canvas from an image source (an uploaded file or an http(s) URL, per the tool schema), with optional name and canvas position/size. Width and height default to the image's real pixel dimensions so it isn't stretched. Requires editor access to the project. Returns `{ image: { id, name, url, position } }`.

### `image generate`

Generates (or edits, given reference images) a raster image server-side. Supports a prompt (required), an aspect ratio (`1:1`, `16:9`, `9:16`, `4:3`, `3:4`), and up to three reference images for editing/restyling. Returns the hosted result: `{ image: { url, mimetype, size, revisedPrompt, ... } }` — the `url` is a permanent public hosted URL; download it locally if you need a file on disk.

Use it for standalone raster requests — photographs, illustrations, textures, transparent cutouts — not for editable canvas designs (use `code start`/`code submit` for those). Follow up with `image add` to place the result on a canvas, or stage it into an authoring session's `assets/` to use it inside a design. If generation is not configured on the server, the tool returns an error — report that plainly rather than claiming an image was created.

## MagicPath skills

Reusable instruction bundles invocable from MagicPath chat. All capabilities accept a team parameter for team-owned skills; omit it for personal skills.

### `skills list`

Lists skills in the user's workspace or a team. Public MagicPath skills are included by default (they are invocable in chat); an owned-only filter hides them. Each skill includes `id`, `name`, `slug`, `description`, `instructions`, `sourceFormat` (`EDITOR` or `ARCHIVE`), `enabled`, `isPublic`, and owner fields.

### `skills get`

Fetches one skill by ID, slash-command slug, or exact name. Options list bundled files (for imported package skills) and fetch a single bundled file's content.

### `skills create`

Creates an editable personal or team skill. Requires non-empty name, description (when to use it), and instructions (how to do the work), passed as Markdown text.

### `skills import`

Imports a `.zip` or `.skill` package into the personal workspace or a team. Package skills are content-immutable after import, but can still be enabled or disabled.

### `skills update`

Updates an editable skill's name, description, or instructions, and enables/disables any owned skill. `ARCHIVE` package skills only support enable/disable. Public MagicPath skills are read-only.

### `skills delete`

Deletes an editable personal or team skill. Public MagicPath skills cannot be deleted.

### Installing a MagicPath skill locally

1. Fetch the skill with `skills get`.
2. Create a local folder named with the returned `skill.slug`.
3. Write `SKILL.md` using the returned `skill.name`, `skill.description`, and `skill.instructions`.
4. If the skill has bundled files, list them, fetch each path's content, and write it under the local skill folder at the same relative path.
5. Register that folder with the current agent host using the host's supported local skill install flow.

Ask before writing into global agent configuration directories.

## Canvas authoring (`code` capabilities)

The authoring capabilities let an external agent author or edit a MagicPath canvas component's source, then publish it back to the platform. This is unrelated to the read-only source fetches above, which serve installation and export.

### Editable file boundary

The authoring flow accepts full-file contents only for:

- `src/App.tsx`
- `src/index.css`
- `src/components/generated/**`
- `assets/**` for temporary image assets only

Image files staged under `assets/` are build inputs: the backend uploads them to stable public asset URLs, rewrites TSX/CSS references, and removes the staging area before build. Reference assets from component code or CSS with paths such as `../../../assets/hero.png`, `/assets/hero.png`, or `url("../../assets/hero.png")`. Do not inline `data:image/...;base64,...` in source files.

When image shapes are selected on the canvas before `code start`, the session includes `selectedImages`. Each selected image has a short-lived `accessUrl` plus a stable `assetPath` under `assets/selected/**`. Use `assetPath` in source, never the expiring `accessUrl`.

The flow does **not** accept dependency installation, `package.json` edits, `src/main.tsx`, Vite config changes, lockfile edits, raw patches, or arbitrary repo files.

### Tailwind v4 requirements

- Keep `@import 'tailwindcss';` in `src/index.css`.
- Do not use `@tailwind base;`, `@tailwind components;`, or `@tailwind utilities;`.
- Do not remove `@theme inline { ... }`, `:root`, or `.dark` token blocks.
- Append custom utilities or theme additions to `src/index.css`; do not replace the whole file.
- There is no `tailwind.config.js`; configuration lives in `src/index.css`.

### `code start` — Begin a create or edit session before writing code

The stateful entrypoint for both create and edit. Exactly one target: a **project** (create a new component) or a **component** (edit an existing one).

For creates: registers the component and a pending revision on the canvas immediately, enables external-agent canvas presence (live cursor), and provides the scaffolded starting structure — a pre-wired slim `src/App.tsx` that renders the top-level component, the Component Forge Tailwind v4 `src/index.css` (`@import 'tailwindcss';`, `@theme inline`, token definitions, base layer), and a stub `src/components/generated/<ComponentName>.tsx` named after the PascalCase form of the component name ("Hero Card" → `HeroCard.tsx`). Pass width and height to set the canvas frame for the component you plan to build.

For edits: creates or reuses one pending edit revision, enables canvas presence, and provides the component's current editable files. Defaults to the component's selected revision; pass a specific revision when the user is viewing a non-current one (carried through from `selection`).

If selected canvas images were available, the session also includes them under `assets/selected/` and reports `selectedImages`.

Keep parallel sessions isolated — one working directory per component. Sessions that share a working area overwrite each other's files.

### `code submit` — Publish the session's files

Submits the session's changed editable files as full-file replacements, plus any deletions since the session began, and triggers a build. Pass width and height together when the canvas size should change. Wait for (or poll) the build so failures can be fixed in the same turn; failed jobs include sanitized build diagnostics.

To delete a file in edit mode, remove it from the session — the deletion is detected and propagated (reported as `deletedPaths`). Deletion applies to source files only; staged assets are temporary inputs and are not deleted server-side. If nothing changed and no dimensions were provided, the submit reports `unchanged` without creating a revision.

On a conflict or stale-base error, call `code start` again for the component to refresh the session, then re-apply the edits. Never create a new component to work around a build failure.

### One-shot create (if offered)

If the server exposes a convenience capability that creates a component from complete files in one call, prefer the explicit `code start` → `code submit` split anyway: it shows the pending component on the canvas while you are still writing code, and it gives you the scaffold to fill in.

### `code status` — Poll a build job

Returns `pending`, `processing`, `completed`, `failed`, or `cancelled` for a job ID. Failed jobs include sanitized diagnostics when available.
