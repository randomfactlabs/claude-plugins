# claude-plugins

Claude Code plugins for the RandomFact product suite.

This repository is a Claude Code **plugin marketplace**: the catalog lives at
[`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) and each
plugin sits in its own subdirectory under [`plugins/`](plugins/).

## Plugins

| Plugin | Description |
| :----- | :---------- |
| [`can`](plugins/can) | Slash commands for the Can kanban workflow (`/can:pr`, …), bundling the hosted Can MCP server. |

## Install

Add this marketplace, then install the plugins you want:

```
/plugin marketplace add randomfactlabs/claude-plugins
/plugin install can@randomfactlabs
```

The `can` plugin registers and auto-starts the hosted Can MCP server. That
server uses OAuth 2.1, so the first time you use it, run `/mcp` and complete the
one-time Can sign-in in the browser; tokens refresh automatically thereafter.
Installing the plugin does **not** authenticate you — that interactive step is
per-user and unavoidable.

## Develop locally

Test a plugin without installing it:

```
claude --plugin-dir ./plugins/can
```

Then `/reload-plugins` to pick up edits, and `/can:pr` to try the command.
Validate the manifests before committing:

```
claude plugin validate ./plugins/can
```

## Layout

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json        # marketplace catalog (lists every plugin)
└── plugins/
    └── can/
        ├── .claude-plugin/
        │   └── plugin.json     # plugin manifest
        ├── .mcp.json           # bundled Can MCP server registration
        └── skills/
            └── pr/
                └── SKILL.md     # /can:pr
```

Manifests live in `.claude-plugin/`; everything else (`skills/`, `.mcp.json`)
sits at the plugin root.
