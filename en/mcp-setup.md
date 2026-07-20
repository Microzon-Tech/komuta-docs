# Installing the Komuta MCP Server

This document describes how to connect an AI coding agent to the Komuta MCP server.

| Property | Value |
|---|---|
| Server name | `komuta` |
| Transport | HTTP |
| URL | `https://mcp.komuta.io/mcp` |
| Authentication | Bearer token |

## Step 1: Add the server configuration

Paste the prompt below into your AI coding agent's chat, unmodified. The agent will register the server using its own standard method — a CLI command, a configuration file, or its settings interface, whichever applies. Keep `<YOUR_API_KEY>` as a literal placeholder; it is replaced in Step 2.

```
Add the Komuta MCP server to whatever AI coding client you are currently running in, using your standard method for registering an MCP server.

- Server name: komuta
- Transport: HTTP
- URL: https://mcp.komuta.io/mcp
- Auth: Bearer token, value: <YOUR_API_KEY>

Keep <YOUR_API_KEY> as a literal placeholder — it is replaced separately. After adding the server, report exactly where <YOUR_API_KEY> needs to be replaced (which file, or which field in a settings screen).
```

For clients without a chat interface to prompt (for example Claude Desktop or claude.ai), add the server manually: open the connector or MCP settings, add a custom server with the URL above, and leave the authentication field empty for now.

## Step 2: Configure the API key

Retrieve the API key from the Komuta account dashboard, then apply it wherever the agent reported inserting `<YOUR_API_KEY>` in Step 1 — replacing the placeholder with the real value.

## Step 3: Verify the connection

Ask the agent to confirm the Komuta MCP server is connected, or to list its available tools. `komuta` should appear.

## Troubleshooting

| Symptom | Cause |
|---|---|
| `401 Unauthorized` | The key is missing, mistyped, or expired. Regenerate it from the Komuta account. |
| Server not recognized | Confirm the agent actually applied the configuration — ask it to show what it added. |
| Key exposure risk | Do not commit the key to version control or share it in plain text; it grants direct access to the Komuta account. |
