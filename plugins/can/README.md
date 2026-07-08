# can

Slash commands for the [Can](https://can.randomfact.com) kanban workflow,
bundling the hosted Can MCP server so the tools and the commands install
together.

The plugin **name** (`can`) drives the `/can:` slash-command namespace — don't
rename it without updating that expectation.

## Commands

| Command | What it does |
| :------ | :----------- |
| `/can:pr [CAN-123]` | Opens a pull request for the current Can ticket. |

## Bundled MCP server

[`.mcp.json`](.mcp.json) registers the hosted Can server (`can`) over HTTP. It
carries **no credentials**: the server is OAuth 2.1 and discovers auth on first
connect. After installing the plugin, run `/mcp` once and complete the Can
sign-in; tokens refresh automatically afterward.

> **Headless caveat:** in `claude -p` / Agent SDK runs there is no `/mcp` panel
> to complete OAuth, so the Can tools report as unavailable until you have
> authenticated once in an interactive session.

## Install

```
/plugin marketplace add randomfactlabs/claude-plugins
/plugin install can@randomfactlabs
```

Or, for local development:

```
claude --plugin-dir ./plugins/can
```
