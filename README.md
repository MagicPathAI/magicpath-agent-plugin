# MagicPath Agent Plugin

The MagicPath plugin in the open, vendor-neutral [Agent Plugins](https://agent-plugins.org) format (spec v1.0.0). Any conformant client loads this directory and gets two components:

- **An MCP server** — the remote MagicPath MCP server over Streamable HTTP (`mcp.json`). Every MagicPath platform operation runs through its tools.
- **An Agent Skill** — `skills/magicpath-guidelines/`, which teaches the agent what the server cannot: which direction a request travels, how to handle files and repositories on the host machine, and the design bar for canvas work.

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
├── skills/
│   └── magicpath-guidelines/
│       ├── SKILL.md                     # routing, ground rules, gates, Design Defaults
│       └── references/
│           ├── using-magicpath-designs-in-local-code.md   # MagicPath → code
│           └── bringing-code-to-the-canvas.md             # code → MagicPath
├── assets/
│   └── magicpath.png                    # logo for marketplaces / client listings
└── README.md
```

## Authentication

The MCP server is an OAuth-protected resource:

- Resource metadata: `https://api.magicpath.ai/.well-known/oauth-protected-resource/mcp`
- Scopes: `magicpath:read`, `magicpath:write`

Authorization is client-managed, as the Agent Plugins spec prescribes: the client discovers the authorization server, runs the OAuth flow, and stores credentials. Unauthenticated requests return `401` with a `WWW-Authenticate` challenge; an authorization failure is a connection failure for the server, not an invalid plugin. Users authenticate through their client's MCP authorization UI — there is no `login` command.

## Installation

Distribution and installation are client-owned under the Agent Plugins spec. Any conformant client can load this plugin from a directory path. Client-specific branding or behavior, if ever needed, belongs under reverse-domain namespaces in the manifest `extensions` field or a namespaced top-level directory — the portable core stays as-is.

## License

MIT
