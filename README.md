<p align="center">
  <img src="assets/magicpath.png" alt="MagicPath" width="96" />
</p>

# MagicPath Agent Plugin

**Give your agent a multiplayer canvas for your code.** Create and edit interactive UI on the [MagicPath](https://www.magicpath.ai) canvas, bring local code and repositories onto it, install or export designs into your projects, and sketch diagrams, wireframes, and annotations — all from your coding agent, while you watch the work happen live.

This is the MagicPath plugin in the open, vendor-neutral [Agent Plugins](https://agent-plugins.org) format (spec v1.0.0). Any conformant client loads this directory and gets two components:

- **An MCP server** — the remote MagicPath MCP server over Streamable HTTP (`mcp.json`). Every MagicPath platform operation runs through its tools.
- **An Agent Skill** — `skills/magicpath/`, which teaches the agent what the server cannot: which direction a request travels, how to handle files and repositories on the host machine, and the design bar for canvas work.

## Architecture: one writer per fact

The plugin is built on a strict layering rule — **every fact has exactly one authoritative home**, chosen by who can keep it true:

| Layer | Home | Carries | Why there |
|---|---|---|---|
| Trigger | skill frontmatter (~100 tokens, always loaded) | when to activate | the only always-on cost |
| Judgment | `SKILL.md` body (loaded on trigger) | direction routing, host-side ground rules, two user gates, canvas Design Defaults | knowledge the server cannot know; stable across server releases |
| Depth | `references/` (loaded on demand) | one playbook per direction: designs → local code, code → canvas | rarely both needed in one task |
| Mechanics | the MCP server itself | tool names, schemas, session lifecycle, file boundaries, transport, auth, retry rules | delivered at call time, always current with the deployment |

The skill deliberately contains **no tool mechanics**. It points at the live tool list and at the server's own guidance resources (`magicpath://guide`, `magicpath://host-guidance`) for clients that don't surface MCP server instructions automatically. Its standing rule: *on any conflict between skill and live server, the server wins.*

This design follows directly from how agents fail. Instruction-following collapses as instruction volume grows and especially when instructions conflict ([IFScale](https://arxiv.org/abs/2507.11538), [Instruction Stacking Collapse](https://arxiv.org/abs/2608.02639), [Context Rot](https://www.trychroma.com/research/context-rot)); a static tool reference inside a skill inevitably drifts against a live server and becomes a conflict generator. Meanwhile the MCP `instructions` field alone is not a substitute, because client support for it is inconsistent — so the skill carries the judgment layer portably and bridges to server-owned mechanics by pointer, not by copy.

## Package layout

```text
magicpath-agent-plugin/
├── plugin.json                          # Agent Plugins manifest (identity + metadata)
├── mcp.json                             # MagicPath MCP server (streamable-http)
├── .codex-plugin/
│   └── plugin.json                      # ChatGPT/Codex listing metadata
├── skills/
│   └── magicpath/
│       ├── SKILL.md                     # routing, ground rules, gates, Design Defaults
│       └── references/
│           ├── using-magicpath-designs-in-local-code.md   # MagicPath → code
│           ├── bringing-code-to-the-canvas.md             # code → MagicPath
│           └── drawing-on-the-canvas.md                   # native canvas shapes
├── .claude-plugin/                      # Claude Code compatibility shim (see below)
│   ├── plugin.json                      # Claude manifest + inline MCP config
│   └── marketplace.json                 # one-plugin catalog, source "./"
├── assets/
│   ├── composer.png                     # ChatGPT/Codex composer icon
│   ├── directory.png                    # ChatGPT/Codex directory logo
│   └── magicpath.png                    # general marketplace/client logo
└── README.md
```

## Authentication

The MCP server is an OAuth-protected resource:

- Resource metadata: `https://api.magicpath.ai/.well-known/oauth-protected-resource/mcp`
- Scopes: `magicpath:read`, `magicpath:write`

Authorization is client-managed, as the Agent Plugins spec prescribes: the client discovers the authorization server, runs the OAuth flow, and stores credentials. Unauthenticated requests return `401` with a `WWW-Authenticate` challenge; an authorization failure is a connection failure for the server, not an invalid plugin. Users authenticate through their client's MCP authorization UI — there is no `login` command.

## Installation

This plugin ships in the vendor-neutral [Agent Plugins v1.0.0](https://agent-plugins.org/specification) format. Distribution and installation are client-owned under the spec: any [conformant client](https://agent-plugins.org/compatible-clients) loads the plugin from this directory (a local checkout or a clone), discovering the skill from `skills/` and the MCP server from `mcp.json`. Cursor loads the format directly; see the spec site's compatible-clients page for the full, current list.

Client-specific behavior belongs under reverse-domain namespaces in the manifest `extensions` field or a namespaced top-level directory — the portable core stays as-is.

### Cursor and other Agent Plugins clients

Install the plugin from this repository's directory using the client's plugin workflow (for example, a local plugin folder or a marketplace listing). No client-specific files are required — the portable core is the plugin.

### Claude Code (CLI)

Claude Code does not yet load the Agent Plugins format natively, so this repo also carries a `.claude-plugin/` compatibility shim — the namespaced-directory pattern the Agent Plugins spec sanctions for client-specific behavior. The shim is plumbing only (a Claude manifest with the MCP server declared inline, plus a one-plugin marketplace catalog whose `source` is the repo root); the skill and MCP definition remain single-sourced in the portable core. Install with:

```bash
claude plugin marketplace add MagicPathAI/magicpath-agent-plugin
claude plugin install magicpath@magicpath
```

Then authenticate the MagicPath MCP server via `/mcp` inside a session. If Claude Code adopts the Agent Plugins format in the future, the shim can be deleted without touching the portable core.

### Claude apps (desktop and web)

The same `.claude-plugin/` shim makes the repo installable from the Claude apps: **Settings → Plugins → Add marketplace**, enter `MagicPathAI/magicpath-agent-plugin`, then install the `magicpath` plugin from that marketplace.

### ChatGPT / Codex

The repo ships `.codex-plugin/` listing metadata (icons, descriptions, suggested prompts) for the ChatGPT plugin directory. Once listed, the plugin installs from the directory inside ChatGPT — no repo URL needed.

### Marketplace listings

As clients list this plugin in their marketplaces, it becomes installable with one click from inside the apps. Loading from this repository, as described above, always works.

## License

MIT
