---
name: magicpath-guidelines
description: Guidelines for working with MagicPath through the MagicPath MCP server — find, preview, inspect, install, export, create, and edit UI components (designs) and manage MagicPath skills. Trigger for MagicPath designs/components; personal or team projects; active canvases and selected components/images/revisions; themes/design systems; teams, members, ownership, attribution, installed-component audits, and share links. Use when exporting exact MagicPath components or revisions to local folders/apps; replacing, adapting, or translating local UI with 1:1 fidelity and explicit requested changes; installing React/TypeScript components from fetched component source; authoring responsive interactive canvas components with the canvas authoring tools; importing or recreating UI from local paths or GitHub/GitLab/Bitbucket repositories into MagicPath; or creating, retrieving, updating, importing, enabling, disabling, or deleting MagicPath skills. In hosts with an embedded browser, keep the project canvas open via share URLs for visual work.
compatibility: Requires the MagicPath MCP server provided by this plugin (network access to MagicPath). A browser enables OAuth sign-in and canvas viewing.
metadata:
  author: MagicPathAI
  source: https://github.com/newcompute-ai/magicpath-agent-plugin
---

# MagicPath

A platform for building, sharing, and installing UI components via AI. All MagicPath operations run through the **`magicpath` MCP server** that ships with this plugin. This skill tells you how to use its tools well; the server's live tool list tells you exactly what to call.

MagicPath canvas components can be created and edited directly through the server's canvas authoring tools — see [Edit or create canvas components](#edit-or-create-canvas-components). That path is strict: only `src/App.tsx`, `src/index.css`, files under `src/components/generated/`, and temporary image assets under `assets/` are editable surfaces of a component.

When this skill runs inside an agent host with an embedded browser, use a MagicPath project as a persistent visual canvas beside the agent when appropriate. If you create a project for canvas authoring, open that project in the embedded browser immediately after creation and before starting authoring; see [Working with embedded browsers](references/working-with-embedded-browsers.md).

## How to call MagicPath

- **Resolve tools from the live tool list.** This skill names *capabilities* using MagicPath's stable vocabulary — `search`, `list-projects`, `list-components`, `selection`, `active-project`, `get-theme`, `code start`, `code submit`, and so on. The `magicpath` server's current tool list is authoritative for the exact tool names and parameter schemas (a capability written here as `list-projects` may surface as e.g. `list_projects`). Check the tool schemas before calling; do not guess parameters.
- **Results are structured.** Tools return structured data — there are no output-format or confirmation flags. Filters described in this skill (team, personal, created-by, sort, limit, cursor, revision) are tool parameters.
- **Authentication is handled by your client.** The server is an OAuth-protected resource (scopes `magicpath:read` and `magicpath:write`). If tool calls fail with an authorization error, the user must authenticate the `magicpath` MCP server through the client's own MCP authorization flow — then retry. There is no `login` command. Never ask the user for raw tokens and never attach credentials yourself.
- **The server cannot touch local files.** Anything that lands on disk — installed component source, exported revisions, locally reconstructed skills — is written by *you*, from data the tools return. Conversely, anything sent to the canvas is content you pass *into* the tools.

> **Terminology:** Users often refer to MagicPath components as "designs" — the two terms are interchangeable. When a user says "design," "my designs," or "that design," treat it as meaning a MagicPath component. Search, inspect, and install accordingly.
>
> Users also refer to MagicPath design systems as "themes." When a user says "theme," "my themes," or "use the X theme," they mean a MagicPath design system — a set of CSS variables, fonts, and styling instructions. Use the `list-themes` and `get-theme` capabilities to work with them.
>
> Users may belong to **teams** (also called "workspaces"). When a user says "the team's designs," "our team's components," or mentions a team name like "Acme Inc," they mean the projects and components owned by that team. Use `list-teams` and team/personal filter parameters to navigate between personal and team workspaces.
>
> Users may also ask about **skills** they created in MagicPath. These are reusable instruction bundles that can be invoked from MagicPath chat and managed with the server's skills tools. Personal skills live in the user's workspace; team skills live in a MagicPath team. Public MagicPath skills are read-only unless the platform says otherwise.

## First Step

Call the server's context capability (`whoami`/`info`) to check auth status, the current user, and available projects and teams.

- If the call fails with an authorization error, tell the user to authenticate the MagicPath MCP server in their client (the client runs the OAuth flow in a browser), then verify with `whoami` again. Do not proceed with other calls until authentication succeeds.

## Guest Sessions

Guest pairing codes (`gst_…`) were a feature of the retired CLI login flow. If the server's tool list exposes a guest or pairing-code capability, follow its schema to connect, and treat the session as scoped to a single project until it expires. Otherwise, tell the user that guest access isn't available through the MCP server and that signing up (then authenticating the server through the client) unlocks the full workspace.

## Working with Teams

Users may belong to teams that own shared projects and themes. By default, `list-projects` and `search` return results from **all** workspaces (personal + every team the user belongs to). Use filter parameters to narrow scope.

### Discovering Teams

Call `list-teams`. It returns the user's teams:

```json
{ "teams": [{ "id": "123", "name": "Acme Inc", "role": "ADMIN" }] }
```

### Filtering by Team

- **Default (no filter)**: `list-projects` and `search` include both personal and all team projects — no extra parameters needed for broad discovery.
- **Team filter** (`"Acme Inc"` or a team ID): narrows to a specific team. Available on `list-projects`, `search`, `list-themes`, and `get-theme`.
- **Personal filter**: shows only the user's personal projects/components. Available on `list-projects` and `search`.

### Structured Output

Projects and search results include `ownerType` (`"personal"` or `"team"`) and `ownerName` (user email or team name). Use these to tell the user where a component lives.

### Discovering People

Call `list-members` with the team name or ID:

```json
{ "team": { "id": "123", "name": "Acme Inc" }, "members": [{ "id": "456", "displayName": "Chloe Smith", "email": "chloe@acme.com", "role": "MEMBER" }] }
```

### Filtering by Person

- **created-by filter** on `list-components`: components a specific user has created or edited. Use it after resolving a person's name to their user ID via `list-members`.
- **`createdBy`** field on projects: each project from `list-projects` includes `createdBy: { id, displayName }` showing who created it.
- **`lastEditedBy`** field on components: each component from `list-components` includes `lastEditedBy: { id, displayName }` showing who last edited it.

**Important:** You can only see projects that the authenticated user has access to — their own personal projects and team projects they are a member of. You **cannot** access another user's personal projects. When looking for another person's work, only search **team projects** (team filter), not personal projects. Personal projects are private to their owner unless someone is explicitly invited as a member.

### Common Patterns

- **"What was Chloe working on last?"** → `list-members` for the team to find Chloe's user ID → `list-projects` filtered to the team to get **team projects only** → `list-components` per project with the created-by filter, sorted by `createdAt` descending. Report the most recent components. **Do not search personal projects for another user's work** — personal projects are private to their owner.
- **"Show me the team's designs"** or **"what has Acme Inc created?"** → `list-teams` to find the team, then `list-projects` filtered to that team, then `list-components` per project.
- **"Show me the latest design from the team"** → same as above, sorted by `createdAt` descending with a limit of 1.
- **"Who created this project/component?"** → check the `createdBy` field on projects or the `lastEditedBy` field on components.
- **"My designs"** without mentioning a team → the default (all projects) is usually correct. Only apply the personal filter if they explicitly want to exclude team projects.
- **"Use the team's theme"** → `list-themes` filtered to the team, then `get-theme` by name within that team.

## Managing MagicPath Skills

Use this flow when the user asks to create, list, inspect, update, import, delete, enable/disable, or locally install skills stored in MagicPath. The skills capabilities cover: `skills list` (with team and owned-only filters), `skills get` (by ID or slug, optionally listing or fetching bundled files), `skills create`, `skills import` (a `.zip`/`.skill` package), `skills update` (including enable/disable), and `skills delete`.

### Scope and Ownership

- Pass the team when the user says the skill belongs to a team/workspace; omit it for personal skills.
- `skills list` includes public MagicPath skills by default because those are available to the user in chat. Use the owned-only filter when the user wants skills they can edit.
- Public skills are read-only. Do not try to update or delete public skills unless the output clearly identifies them as owned/editable.
- Imported `.zip` or `.skill` packages are content-immutable in MagicPath; they can still be enabled or disabled via `skills update`.

### Creating or Updating Skills

- A MagicPath skill requires a non-empty name, description, and instructions. The description should say when the skill should be used; the instructions should say how to do the work. Pass instructions as Markdown text through the tool parameters.
- If a user is turning an observed workflow into a reusable skill, summarize the trigger, constraints, steps, examples, and any files or references the future agent should read.

### Installing a MagicPath Skill Locally

When a user wants a skill from MagicPath installed into their external coding agent, first retrieve it, then recreate it as a local Agent Skills folder:

1. Call `skills get` for the skill.
2. Create a folder named after the skill slug.
3. Write `SKILL.md` with frontmatter containing at least `name` and `description`, followed by the retrieved `instructions`:

```markdown
---
name: example-skill
description: Use when ...
---

...instructions from MagicPath...
```

4. If the skill has bundled package files, list them via `skills get` with the files option, fetch each file's content, and recreate the same relative paths in the local skill folder.
5. Install or register that folder using the current agent host's local skill workflow.

Ask before writing outside the user's current project or into a global agent configuration directory.

## Workflow

### Phase 1: Discover

1. **Check auth** — call `whoami` to verify authentication.
2. **Check current selection** — if the user references "the selected component," "the selected image," "the design I have selected," or otherwise points at a *specific canvas selection*, call `selection`. If it returns components, use them directly — skip the search/confirm flow and proceed with the returned `generatedName`(s). Each returned component also includes `selectedRevisionId`, the revision currently shown for that component on the canvas. The response can also include selected `images`. When a downstream capability accepts a revision (such as fetching component source at a specific revision), pass this value through so the operation targets the version the user is looking at rather than whichever revision happens to be canonical in the database.
3. **Check the active project** — if the user references "the project I have open," "this project," "what I'm working on," or otherwise implies a working project context without naming a specific component, call `active-project`. It returns the project(s) the user currently has open in their browser, even when nothing is selected. If it returns one project, treat it as the working project and skip the project picker. If it returns multiple, list them and ask which one. If it returns an empty list, the user has no canvas open — reach for `list-projects` and ask the user. Pick the right capability for what the user said: `selection` for a referenced component, `active-project` for a referenced project, `list-projects` + ask if neither. (Note that `selection` also returns the active projects in its output, so when the user references a component you already get the project for free — no separate `active-project` call needed.)
4. **Find components** — use `search` to search across all projects, or `list-projects` then `list-components` to browse. If `active-project` already gave you a project, scope to it via `list-components` instead of searching every workspace.
5. **See a project's images** — to know which standalone images already live on a project's canvas (to reference them, avoid duplicating them, or describe them to the user), call `image list` for the project and download each `url` to view it — the same way you use `previewImageUrl` for components. (These are canvas images, separate from the `assets/` build inputs in the authoring flow.)
   For a **standalone raster request** — a photograph, illustration, texture, or transparent cutout — use the `image generate` capability rather than building a canvas design; pass reference images to edit or restyle an existing photo. Place the result on a canvas with `image add`, or feed it into an authoring session as an asset. See the [tool reference](references/tool-reference.md).
6. **Understand components visually** — `search` and `list-components` results include a `previewImageUrl` field. Download and analyze these images to understand what each component looks like before recommending it. Preview images are for your own understanding — do not navigate the embedded project canvas to an individual design preview unless the user explicitly asks to see that design there.
7. **Confirm with the user (STOP and wait)** — unless the user specified an exact generatedName, tell the user what you found (name, generatedName, project) and ask if it's the right component. When an embedded project canvas is active, keep it on the project and only open or share an individual design if the user explicitly asks. Without an embedded project canvas, get the component's URL via `share` and give it to the user as the normal confirmation fallback. If multiple matches, list them all and ask which one. **This is a STOP point — end your response here and wait for the user to reply. Do NOT proceed until the user explicitly confirms.** Do not fetch source for installation yet.

### Phase 2: Understand the Target Context

> **This phase is critical.** Before installing anything, you MUST understand where the component is going and what it needs to do there. Skipping this leads to components that look right but behave wrong.
>
> **Taking a design outside MagicPath:** Before exporting to a folder, installing into an app, replacing local UI, or translating to another framework, read and follow [Using MagicPath designs in local code](references/using-magicpath-designs-in-local-code.md). It defines how to lock the exact revision, establish 1:1 parity before adaptation, preserve local behavior, verify the rendered result, and limit differences to changes the user explicitly requested.

8. **Inspect the MagicPath component source** — fetch the component's source (the `component source` capability; `inspect`-style by `generatedName`, or the revision-aware fetch by component ID). Identify what it renders, what props it expects, and what assumptions it makes about layout (fixed widths, absolute positioning, etc.).
9. **Read the target codebase context** — before installing, read the file(s) where the component will live. Understand:
   - **Existing functionality**: If replacing a component, what does the current one do? What callbacks, state, API calls, navigation, validation, or side effects does it handle? Every piece of existing behavior must be preserved or consciously addressed.
   - **Layout context**: What is the parent layout? Is it a flex/grid container? What are the responsive breakpoints? How does spacing work? A component that looks perfect in isolation can break a layout if its sizing assumptions don't match.
   - **Data flow**: What props, context, or state does the surrounding code provide? What does it expect back (callbacks, form data, events)?
   - **Design system**: What styling patterns does the project use (Tailwind, CSS modules, theme tokens)? The MagicPath component's styles need to harmonize, not clash.

### Applying a Theme (if applicable)

If the user has a theme they want applied, or references a brand/design system by name:

1. **List available themes** — call `list-themes` to see all themes.
2. **Get the theme definition** — call `get-theme` by ID or name to fetch the full definition.
3. **Read the `prompt` field** — if present, this contains natural-language styling instructions from the designer (e.g., "use rounded corners, prefer shadows over borders, use the brand blue for CTAs"). Follow these instructions when adapting components.
4. **Apply CSS variables** — the theme's `light` and `dark` objects map CSS variable names to values (e.g., `--background: #ffffff`, `--primary: #3b82f6`). When adapting MagicPath components, use these CSS variables instead of hardcoded colors: `bg-[var(--background)]`, `text-[var(--primary)]`, etc. Ensure the component respects `defaultTheme` (light or dark).
5. **Handle fonts** — if the theme includes `fonts`, ensure the project loads these fonts (Google Fonts link or `@font-face` declarations for custom fonts) and that components reference them via the theme's font CSS variables (e.g., `font-family: var(--font-body)`).
6. **Non-React/JS projects** — theme data is a reference, not a stylesheet. Translate CSS variables into the target platform's equivalent: SwiftUI `Color` assets, Android theme XML, Python template context, etc. The `prompt` field and color/font values express platform-agnostic design intent — map them to native patterns rather than using CSS directly.

### Phase 3: Install and Adapt

10. **Install into the project — you write the files.** The MCP server returns component source; installation is your job:
    - Fetch the full source (files, dependencies, `importStatement`, `usage`) via the component source capability.
    - Write each returned file under `src/components/magicpath/<name>/`, preserving the returned relative paths.
    - Install the returned npm dependencies with the project's package manager (respect the existing lockfile).
    - If this is a **non-React project** (Swift, Python, etc.), **do not install the files** — use the fetched source as a reference, then recreate the component in the target language and framework.
11. **Adapt the component for production use** — MagicPath components are design artifacts: they capture visual intent and structure, but they are often not production-ready out of the box. After installing, you MUST edit the component files to:
    - **Make it responsive**: Replace any hardcoded widths/heights (e.g., `w-[300px]`) with responsive utilities (`w-full max-w-sm`, responsive breakpoints like `md:w-64 lg:w-80`). A design may show a single viewport — your job is to make it work across all viewports.
    - **Add real interactivity**: Replace static/placeholder content with actual props, state, and event handlers. A MagicPath button that says "Submit" needs an `onClick` prop and loading state. A form needs validation and `onSubmit`.
    - **Wire up data flow**: Connect the component to the app's actual data — props from parents, context providers, API calls, router state. Don't leave mock data in place.
    - **Preserve existing functionality**: When replacing an existing component, audit every feature the old one provided (form submission, error handling, loading states, accessibility, keyboard navigation, analytics events) and ensure the new component handles all of them.
    - **Match the project's patterns without redesigning**: Use the same state management and error handling approaches. Reuse styling abstractions only when their computed output preserves the MagicPath design; do not replace exact values with merely similar local tokens or primitives.

### Phase 4: Integrate into the Page

12. **Import and render** — import the component using the `importStatement` from the fetched source. Pass the props you've defined.
13. **Verify layout fit** — after placing the component, review the parent layout to ensure it integrates cleanly. Check that the component doesn't overflow, create unexpected gaps, or break the responsive flow of the page.

## Design-to-Production Mindset

**MagicPath is a design tool.** Components from MagicPath represent what something should look like and how it should be structured — they are the design spec expressed as code. But a design comp and a production component are different things:

| Design artifact | Your job as the agent |
|---|---|
| Fixed width `w-[400px]` | Make it responsive: `w-full max-w-md` or breakpoint-based |
| Static text "John Doe" | Replace with dynamic prop: `{user.name}` |
| Placeholder `onClick={() => {}}` | Wire to real handler: `onClick={handleSubmit}` |
| Hardcoded list of 3 items | Map over real data: `{items.map(…)}` |
| No error/loading states | Add loading spinners, error boundaries, empty states |
| No accessibility attributes | Add `aria-label`, `role`, keyboard handlers, focus management |
| Desktop-only layout | Add responsive breakpoints, mobile navigation patterns |
| Decorative images with `src="/photo.jpg"` | Use real assets or proper placeholders from the project |

**The golden rule: a MagicPath component tells you WHAT to build. Your job is to make it WORK — responsively, accessibly, and fully wired into the application.**

### Common Scenarios

**Replacing an existing component** (e.g., swapping an old login form for a MagicPath design):
1. Read the old component thoroughly — list every prop, callback, validation rule, and side effect
2. Fetch the MagicPath component source through the server
3. Write the component files into `src/components/magicpath/<name>/` and install its dependencies
4. Edit the installed component to accept all the same props/callbacks
5. Ensure every feature from the old component exists in the new one
6. Swap the import in the parent — the parent code should barely change

**Building a new page from a MagicPath design library**:
1. Browse the project's components with `list-components`
2. Plan the page layout first — identify which MagicPath components map to which sections
3. Fetch and install needed components one at a time
4. Build the page layout, importing each component
5. Adapt each component: responsive sizing, real data, proper routing, state management
6. Ensure consistent spacing, typography, and color usage across all components

**Using a single MagicPath component as inspiration**:
1. Fetch the source through the server
2. Understand the design intent — colors, spacing, layout structure, typography
3. Install and adapt it, or use it as a reference to build something custom that follows the same design language

## Critical Rules

- **Fetching source for installation means install-to-use.** Only write component files into the project when you intend to import and render them. If you just want to read the source code, fetch it and read it — don't write files.
- **After installing, always import the component.** The whole point of installing is to get source files you then import. Never install a component and then copy its styles/markup into another file — import and render the component directly.
- **Installed MagicPath components are source code the user owns.** After installation, the component files live in the project at `src/components/magicpath/<name>/`. You can and should edit them directly to add props, change behavior, adjust styles, or integrate with the app's state.
- **When a component needs integration:** (1) install the component files, (2) edit the component file to accept the props you need (e.g., `onSubmit`, `placeholder`, `className`), (3) import it from the parent and pass those props. Do NOT copy the component's JSX/styles into the parent file.
- **Never just drop a component in.** Always read the surrounding code, understand the layout constraints, and adapt the component to fit. A MagicPath component placed without adaptation is a bug, not a feature.
- **Reading source is read-only.** Fetching component source writes nothing by itself. Use it to decide whether a component fits before committing to install.
- **Installation is for React/TypeScript projects only.** MagicPath components are React/TS source with npm dependencies. For non-JS projects (Swift, Python, etc.), fetch the source as a reference, then translate the design and behavior into the project's language and framework.
- **Share one thing at a time.** When giving the user links, resolve and present one target at a time; don't flood the conversation or the browser with multiple previews at once.
- **Keep an embedded browser on the project canvas.** Do not navigate it to individual design previews unless the user explicitly asks; return a design share link instead when that is sufficient.
- **Open a newly created project before authoring into it.** When an embedded browser is available and the request includes work inside a new project, show the project canvas immediately after `create-project` and before starting canvas authoring.
- **Do not mix directions.** Fetching source and installing (MagicPath → app) and canvas authoring (`code start` → `code submit`, app → MagicPath) are separate flows. Never use authoring tools for an export, and never use source-fetch tools to author.

## Creating a project

A **project** is the workspace that holds designs/components. Use this when the user explicitly asks to create a project ("make a new project called …", "create a project for …"), or when they ask for a new design but no project context exists yet and a fresh project is the right home for it.

### Picking the workspace

Before creating, decide whether the project is **personal** or belongs to a **team**:

- If the user names a team ("create a project in Acme Inc"), resolve that team and pass it through.
- If the user says "create a personal project" or doesn't mention a team and has no teams, default to personal.
- If the user is ambiguous and belongs to one or more teams, call `list-teams` and ask which workspace — personal or one of the teams. Don't guess. **STOP and wait for the user to reply.**

### Running the capability

Call `create-project` with a name (and the team, if any):

- The name is optional — without one the project gets an auto-generated placeholder name. Always pass the name when the user told you what to call the project.
- The team parameter accepts a team name or team ID. Resolve the user's intent to one of the teams returned by `list-teams`.
- The result is `{ project: { id, name, ownerType, ownerName, ... } }`. The `id` is what subsequent capabilities need. Visibility is set automatically: personal projects default to `PRIVATE`, team projects to `SHARED`.

### After the project exists

If the user also asked for a design inside the new project, take the `id` from the response and continue with the canvas authoring flow described under [Edit or create canvas components](#edit-or-create-canvas-components). Do not re-create the project per design — one project holds many components.

When the task includes creating or editing designs inside a newly created project, treat that project as the canvas. In an embedded-browser host, the order is mandatory: immediately after `create-project` returns an `id`, call `share` for the project to get its URL, open that URL in the embedded browser, and only then begin authoring. Keep that project canvas visible while work appears there; do not navigate to the generated design preview after submission unless the user explicitly asks to see that design alone. If no embedded browser exists, give the user the share URL when user-facing navigation is needed.

## Use a MagicPath project as an embedded canvas when available

Some agent hosts, including Codex and some Cursor workflows, can provide an embedded or in-app browser. When that capability is available, a MagicPath **project** can remain open beside the agent as a persistent canvas: the agent works from local code and context while the user sees and selects work on the canvas.

Use this for opening a newly created project, reconnecting to the user's active project, or beginning canvas authoring in a named project. Do not automatically navigate the embedded browser to an individual design or component; only show one there when the user explicitly asks to open that specific design.

Call `share` for the project to get its URL without opening anything, then navigate the host's embedded browser to the returned `url`. If no embedded browser exists, it cannot be controlled reliably, or the user explicitly wants their normal browser, give the user the URL to open themselves.

MCP authentication and embedded-browser authentication are separate. A successful `whoami` or `create-project` call does not mean the visible browser pane is signed into MagicPath. If opening the returned `/files/<projectId>` URL redirects to the home or sign-in experience, keep the task focused on that project: tell the user to sign into MagicPath in the embedded browser, then navigate back to the same project URL after sign-in. Do not substitute a public individual-design preview just because it loads without the project-canvas session.

Do not open a project automatically for background work such as `whoami`, listing/searching data, retrieving themes, reading component source, or installing a component into an application. Full decision guidance and recipes live in [Working with embedded browsers](references/working-with-embedded-browsers.md).

## Bring an existing repository into MagicPath

When the user wants to take UI that already exists in a Git repository — local or online — and reproduce it on their MagicPath canvas (e.g. "bring the sidebar of my app into MagicPath", "render this project in MagicPath", "recreate my landing page here"), recreate it as a canvas component via the `code start` → `code submit` authoring flow.

This is the **inverse** of installing: the source of truth is the user's repository and the destination is the canvas. Do **not** use the source-fetch/install capabilities for this. The short version:

1. **Get the code** — read a local path directly, or `git clone --depth 1 <url>` an online repo into a scratch directory (kept separate from your authoring working directory). Private repos need the user's credentials — ask, don't guess.
2. **Read the design foundation first** — global CSS (`globals.css`/`index.css`/`app.css`), design tokens (`tailwind.config.*`, CSS variables, token files), fonts, theming strategy, and shared UI primitives. This is what makes the recreation faithful rather than approximate.
3. **Resolve the target** — for a single element (e.g. the sidebar), open its file and follow all its imports (child components, icons, styles, data) plus the layout parent that gives it size and position. For a whole page/project, identify the entry and decide one interactive frame vs. separate frames per screen (Design Default rule 5) — ask if ambiguous and **stop and wait**.
4. **Recreate on the canvas** — `code start` against the project with a name and canvas dimensions, write the component faithfully into the editable surfaces (translate the repo's framework and styling into React + Tailwind v4), match colors/spacing/typography exactly, wire real interactivity, mock data locally, then `code submit` and wait for the build. Honor the Design Defaults (responsive, centered, no device mockups, single screen, fully interactive).
5. **Verify** the result against the source app on the project canvas or via its share URL.

Full step-by-step guidance — the styling-translation table, edge cases (monorepos, non-React sources, server components, Tailwind v3→v4), and quick recipes for "bring the sidebar of my app" and "render this project" — lives in [Working with repositories](references/working-with-repositories.md).

## Edit or create canvas components

Use this workflow when the user wants you to author or modify a MagicPath canvas component itself — not install an existing component into a separate application. The authoring capabilities operate as a session against one component: `code start` begins it, `code submit` publishes it, `code status` polls the build.

> **When authoring on the MagicPath canvas, you are an expert design engineer who builds beautiful, *functional, interactive* React components.** Components you produce on the canvas should be real working mini-apps, not static design comps: state-driven, hover / focus / active states wired up, buttons that do something, forms that validate, transitions that feel deliberate. A pretty but lifeless component is a failed component. (This persona applies only to the authoring flow — when you're installing components into a user's project, follow the [Design-to-Production Mindset](#design-to-production-mindset) instead.)

### SUPER IMPORTANT — Design Defaults

These rules apply to every canvas component you create or edit with the authoring tools, unless the user **explicitly** overrides them in the request. They do **not** apply to the install flow — for that, see the [Design-to-Production Mindset](#design-to-production-mindset). These rules override anything else in this skill for canvas authoring.

#### 1. NEVER add device mockups

Do NOT wrap components in iPhone / Android / laptop / desktop / browser frames, status bars, notches, home indicators, address bars, or any other device chrome. Only add a device mockup if the user **explicitly** asks for one ("show this inside a phone frame", "wrap it in an iPhone mockup", "make it look like a Mac window"). Designing for a mobile viewport is **not** a request for a mockup — the canvas itself is the device frame. Never draw a second device inside it.

#### 2. Everything is responsive — always

Every component must work at any width, including small primitives like buttons, inputs, badges, and cards. Use `w-full`, `max-w-*`, percentage widths, flex/grid sizing, and breakpoint utilities (`sm:`, `md:`, `lg:`). Do not hardcode pixel widths/heights on outer containers. The only exceptions are intrinsically fixed elements (avatars, icons, fixed-size media).

#### 3. Always centered inside the canvas

The root of the component should center itself in its frame — horizontally, and vertically when the design is short. Use `min-h-screen flex items-center justify-center`, `mx-auto`, or grid centering on the root. The design must never stick to a corner when the canvas is larger than the content, and must never overflow when it's smaller.

#### 4. Canvas size ≠ device mockup

You may (and should) pass width/height on `code start` / `code submit` to reflect the target device — e.g. `390×844` for a mobile design, `1440×900` for desktop. That's how you signal "this is a mobile design." But the content inside must remain fluid: if the same component is dropped into a wider or narrower container later, it should adapt — not stay locked to the original pixel size.

#### 5. NEVER stack multiple screens inside one frame

A MagicPath component is **one** frame. Do not draw "Screen 1 / Screen 2 / Screen 3" side-by-side, vertically stacked, or as a slideshow inside a single canvas. That output is broken — it doesn't render, it doesn't navigate, and it wastes the user's canvas.

When the user wants something that spans multiple views, pick one of these two patterns and **stick to it**:

**A. Self-contained app in ONE frame (preferred when the views belong to the same flow).** A single component can hold many views, screens, modals, tabs, steps, or routes by using React state, conditional rendering, tab components, client-side routing, or `useState`-driven view switching. A login → signup → forgot-password flow, a multi-step wizard, a settings page with tab navigation, a dashboard with a slide-out detail panel — all of these belong in **one** component with internal state, not several frames glued together.

**B. Multiple frames (one component per screen) when the screens are truly independent.** If the user is asking for distinct deliverables — "design the login screen, the dashboard, and the settings page" — each one is its own MagicPath component. Author them as separate `code start` sessions, **each with its own working directory** (parallel sessions that share a working directory will overwrite each other's files). Build them concurrently — if your environment supports parallel sub-agents, spawn one per frame; otherwise run the sessions in parallel however your runner allows. Do not try to render them all in a single canvas to "save time" — it produces a broken artifact.

If you're unsure which pattern fits, ask the user: "Should this be one interactive component with internal navigation, or separate frames for each screen?" — and **stop and wait** for the answer.

#### 6. Build interactive components, not static markup

You are an engineer, not a screenshot generator. Every canvas component must be **fully interactive** — buttons trigger real actions, inputs are controlled, forms submit and validate, hover / focus / active / disabled states are styled, modals open and close, tabs switch, drawers slide, dropdowns expand, toggles flip, accordions collapse. Use `useState` / `useReducer` for local state, real event handlers (`onClick`, `onChange`, `onSubmit`, `onKeyDown`, `onBlur`), `aria-*` attributes for accessibility, and meaningful transitions (Tailwind `transition-*`, Framer Motion, or CSS animations) where they add polish. A component left with placeholder `onClick={() => {}}` or static markup of an interactive surface is **not done** — wire it up before `code submit`. If the component represents a multi-view flow, make the navigation between views work via state (see rule 5.A).

**Editable file boundary.** The authoring flow only accepts full-file contents for:

- `src/App.tsx`
- `src/index.css`
- `src/components/generated/**`
- `assets/**` for temporary image assets only

Never submit `package.json`, `vite.config.*`, `src/main.tsx`, lockfiles, or any other path — they will be rejected.

**Image assets.** Stage local image files under the session's `assets/` surface and reference them from code or CSS, for example `../../../assets/hero.png`, `/assets/hero.png`, or `url("../../assets/hero.png")`. MagicPath uploads these temporary assets, rewrites references to stable public asset URLs, and removes the `assets/` staging area before build. Do not inline `data:image/...;base64,...`; if you encounter base64 image data, move it into an asset file instead.

**Selected canvas images.** When the user has selected image shapes on the canvas before `code start`, the session includes them as `selectedImages`, each with a short-lived `accessUrl` and a stable `assetPath` under `assets/selected/**`. Use the `assetPath` in imports or CSS. Do not use `accessUrl` directly because it expires.

**Deleting and renaming source files is supported in edit mode.** Remove an editable source file from the session and `code submit` propagates the deletion. A rename is a delete + a write in the same submit. Assets are temporary staging inputs and are not deleted from the server by removing them locally. In create mode, there's nothing to delete; just don't write the file.

**Do not use source-fetch/install capabilities for this workflow.** They read MagicPath-side source for installing into another app. The authoring flow edits components on the user's MagicPath canvas — they are separate flows and must not be mixed.

### Edit an existing component

1. Call `code start` for the component. This creates or reuses a pending edit revision, shows agent presence on the canvas, and gives you the component's editable files. By default, the session starts from the component's currently selected revision. To start from a specific revision instead, pass the revision — useful when the user is viewing or referring to a non-current revision (e.g. a value carried through from `selection`).
2. Edit, add, or delete allowed files within the session (see the boundary above). Put any new images under the `assets/` surface and reference them from the generated component or CSS. When you remove the last usage of a sub-component file, delete its source file too — don't leave orphan files in the revision. Renames are delete-plus-write.
3. Call `code submit` and wait for the build. If your edit changes the intended canvas size, pass both width and height on submit.
4. If the job result is `failed`, read the returned sanitized diagnostics, fix only allowed files, and submit again. Do not create a new component to work around a build failure.
5. If the submission reports a conflict or stale base, call `code start` for the component again to refresh the edit session before re-applying your edits.

### Create a new component

**Important experiential rule:** always call `code start` *before* writing component files. This registers the pending component on the canvas so the user sees your work-in-progress presence, not a silent agent.

**Expected file structure.** A MagicPath component has a slim `src/App.tsx` that imports and renders a top-level component from `src/components/generated/`. The actual implementation lives in `src/components/generated/<ComponentName>.tsx` (PascalCase filename, named export). Larger components should be split into additional sibling files under `src/components/generated/`, each importing what it needs. This is how every existing MagicPath component is structured.

**`code start` scaffolds this structure for you.** After it returns, the session already contains a pre-wired `src/App.tsx` and a stub `src/components/generated/<ComponentName>.tsx` (the filename is the PascalCase form of the name you passed, e.g. "Hero Card" → `HeroCard.tsx`). Your job is to fill in the stub — **do not rewrite `App.tsx`**, it's already correct. The only reason to edit `App.tsx` is to change the `theme` (`'light'`/`'dark'`) value at the top.

Steps:
1. Call `code start` with the project ID, a component name, and width/height that fit the component you plan to build instead of relying on the default canvas size. **Reminder:** the component must be responsive, centered, free of device mockups, a **single screen** (use internal state for multi-view flows, or parallel sessions for separate screens), and **fully interactive** (real handlers, controlled inputs, state-driven views, hover/focus/active states). See the [Design Defaults](#super-important--design-defaults) above.
2. Fill in `src/components/generated/<ComponentName>.tsx` with the component implementation. Split into additional files in the same directory if the component is substantial.
3. Optionally edit `src/index.css` for custom styles (see the Tailwind v4 rules below). Stage image files under `assets/` and reference them from TSX or CSS instead of embedding base64.
4. Call `code submit` and wait. If the final implementation needs a different canvas size than you chose at start, pass both width and height here.
5. If the build fails, fix the component files and re-submit. Do not start a second component unless the user explicitly asks.

> If the server offers a one-shot create capability that combines start and submit, prefer the explicit two-step flow anyway — it makes your progress visible on the canvas while files are still being written, and it gives you the scaffolded starting point to work from.

#### Tailwind v4 Rules

The MagicPath template uses Tailwind v4. Style this way:

- `src/index.css` must contain `@import 'tailwindcss';`, not `@tailwind base;`, `@tailwind components;`, or `@tailwind utilities;`.
- Theme tokens (`bg-background`, `text-foreground`, `border-border`, `bg-primary`, etc.) are wired via the `@theme inline { ... }` block in `index.css`. Do not remove it.
- The `:root` and `.dark` blocks define the actual token values. Do not remove them.
- To add custom utility classes, append them to `index.css` instead of replacing existing content.
- There is no `tailwind.config.js`. Configuration lives in `index.css` via Tailwind v4's `@theme` directive.

### Polling a job separately

If you need to check job status after the fact (for example, after submitting without waiting), call `code status` with the job ID. It returns one of `pending`, `processing`, `completed`, `failed`, or `cancelled`.

## Capability Quick Reference

Resolve each capability to the current tool on the `magicpath` server; full details in the [tool reference](references/tool-reference.md).

| Area | Capability | Purpose |
|---|---|---|
| Identity | `whoami` / `info` | Auth status, user, teams, projects |
| Canvas context | `selection` | Selected components (with `selectedRevisionId`), selected images, open projects |
| Canvas context | `active-project` | Project(s) the user has open (lighter than `selection`) |
| Discovery | `search` | Find components across workspaces (team/personal filters) |
| Discovery | `list-projects` / `list-components` | Browse projects, then components (`previewImageUrl`, `lastEditedBy`, created-by filter, cursor pagination) |
| Teams | `list-teams` / `list-members` | Workspaces and people (resolve names to user IDs) |
| Projects | `create-project` | New personal or team project |
| Links | `share` | URL for a component or project (never opens anything itself) |
| Source out | component source (`inspect` / revision-aware fetch) | Read full source, dependencies, `importStatement` — installation and export input |
| Themes | `list-themes` / `get-theme` | Design systems: CSS variable maps, fonts, designer `prompt` |
| Images | `image list` / `image add` / `image generate` | Canvas images and AI raster generation/editing |
| MagicPath skills | `skills list/get/create/update/import/delete` | Manage skills stored in MagicPath |
| Authoring | `code start` / `code submit` / `code status` | Create or edit canvas components; build lifecycle |

## Key Concepts

- Each component has a **generatedName** (e.g., `wispy-river-5234`) — this is the identifier for registry-style operations — and a numeric ID used by canvas/authoring capabilities. Revisions have their own IDs; `selection` tells you which revision the user is looking at.
- Installed components live as source code at `src/components/magicpath/<name>/` — written by you from fetched source, then owned and edited like any project code.
- Fetched component source includes `importStatement` and `usage` — use these when wiring the component in.
- Reading source is free of side effects — don't write files just to read code.
- MagicPath components are React/TypeScript source — install in JS/TS projects; fetch + translate for other languages.
- **Themes** (design systems) contain CSS variables (`light`/`dark` maps), optional `fonts`, and an optional `prompt` with styling instructions for agents. "Theme" and "design system" are interchangeable.
- Use `code start` + `code submit` to publish source edits back to the MagicPath canvas. The read-only revision-aware source fetch is the exact-revision retrieval path for outward exports. Source-fetch + install remains the application-integration path.

## References

- [Tool Reference](references/tool-reference.md) — the MagicPath MCP server's capability surface, expected inputs/outputs, and conventions
- [Using MagicPath designs in local code](references/using-magicpath-designs-in-local-code.md) — export an exact component or revision, replace local UI, or translate a MagicPath design while preserving 1:1 fidelity and explicitly requested differences
- [Working with repositories](references/working-with-repositories.md) — bring an existing local or online Git repository's UI onto the MagicPath canvas (e.g. "render this project in MagicPath", "bring the sidebar of my app into MagicPath")
- [Working with embedded browsers](references/working-with-embedded-browsers.md) — use a MagicPath project as the persistent canvas inside Codex, Cursor, or another host with an in-app browser
