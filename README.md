# MagicPath Agent Plugin (Staging)

The MagicPath plugin packaged in the open, vendor-neutral [Agent Plugins](https://agent-plugins.org) format (spec v1.0.0). Any Agent Plugins–conformant client can load this directory and get both components:

- **An MCP server** — the remote MagicPath MCP server over Streamable HTTP, declared in `mcp.json`. All MagicPath platform operations (search, projects, components, themes, teams, canvas authoring, images, MagicPath skills) are performed through its tools.
- **An Agent Skill** — `skills/magicpath-guidelines/` teaches the agent *how* to use those tools well: workflow selection, fidelity rules, canvas design defaults, and the boundaries between the install, authoring, and import directions.

> **Staging:** This is the testing-phase package, published as `magicpath-staging`. It targets the staging MCP endpoint at `https://staging.api.magicpath.ai/mcp`. The production release will be published as `magicpath` against the production endpoint.

## What changed from the previous plugin

This package replaces [MagicPathAI/agent-skills](https://github.com/MagicPathAI/agent-skills), which drove MagicPath through the `magicpath-ai` CLI (`npx -y magicpath-ai ...`). Two things are new:

1. **Format:** the client-specific manifests (`.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.agents/`) are replaced by one portable package: a root `plugin.json`, components in fixed locations (`skills/`, `mcp.json`), per the Agent Plugins v1.0.0 specification.
2. **Transport:** the CLI approach is retired. The plugin now connects to the remote MagicPath MCP server; the skill is renamed from `magicpath` to `magicpath-guidelines` because the MCP server *is* the MagicPath integration — the skill provides the guidelines for using it.

## Package layout

```text
magicpath-agent-plugin/
├── plugin.json                          # Agent Plugins manifest (identity + metadata)
├── mcp.json                             # MagicPath MCP server (streamable-http)
├── skills/
│   └── magicpath-guidelines/
│       ├── SKILL.md                     # core workflow guidance
│       └── references/
│           ├── tool-reference.md        # MCP tool surface, capability by capability
│           ├── using-magicpath-designs-in-local-code.md
│           ├── working-with-repositories.md
│           └── working-with-embedded-browsers.md
├── assets/
│   └── magicpath.png                    # logo for marketplaces / client listings
└── README.md
```

## Authentication

The MCP server is an OAuth-protected resource:

- Resource metadata: `https://staging.api.magicpath.ai/.well-known/oauth-protected-resource/mcp`
- Scopes: `magicpath:read`, `magicpath:write`

Authorization is client-managed, as the Agent Plugins spec prescribes: the client discovers the authorization server, runs the OAuth flow, and stores credentials. Unauthenticated requests return `401` with a `WWW-Authenticate` challenge; an authorization failure is a connection failure for the server, not an invalid plugin. There is no `login` command anymore — users authenticate through their client's MCP authorization UI.

## Installation

Distribution and installation are client-owned under the Agent Plugins spec. Any conformant client can load this plugin from a directory path. Client-specific branding or behavior, if ever needed, belongs under reverse-domain namespaces in the manifest `extensions` field or a namespaced top-level directory — the portable core stays as-is.

## License

MIT
