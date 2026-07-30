# Installing the Komuta MCP Server

This document describes how to connect Claude and Codex to the Komuta MCP server.

## What can you do with the MCP?

Before installing, you can browse what the MCP server offers: provision services from a git repository, assess service health with Service Doctor, create managed Postgres / RabbitMQ / Valkey addons, inspect logs and traces, and look up packages and pricing.

Full list of tools and what they do: [Komuta MCP Tools](https://www.komuta.io/docs/mcp/mcp-tools)

## Install for Claude

### Claude Code (CLI)

```bash
claude mcp add komuta http://mcp.komuta.io/mcp
```

### Claude Desktop

`Settings` → `Connectors` → `Add` → `Custom Connector`

- **Name:** Komuta
- **Remote MCP Server URL:** `http://mcp.komuta.io/mcp`

`Add` → `Connect`

## Install for Codex

1. Install the Codex CLI:

   ```bash
   npm install -g @openai/codex
   ```

2. Add this to `~/.codex/config.toml` (on Windows: `C:\Users\<Username>\.codex\config.toml`):

   ```toml
   [mcp_servers.komuta]
   url = "http://mcp.komuta.io/mcp"
   scopes = ["openid", "email", "profile", "offline_access", "komuta:mcp"]
   ```

3. Log in:

   ```bash
   codex mcp login komuta
   ```

## Verify the connection

Ask the agent to confirm the Komuta MCP server is connected, or to list its available tools. `komuta` should appear.

## Troubleshooting

| Symptom | Cause |
|---|---|
| Browser login doesn't open or complete | Re-run `codex mcp login komuta`; for Claude Desktop, remove and re-add the connector. |
| Server not recognized | Confirm the configuration was added to the right file (`~/.codex/config.toml`) or via the correct CLI command. |
| `401 Unauthorized` / auth error | The session may have expired; repeat the login step. |
