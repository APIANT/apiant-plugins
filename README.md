# APIANT plugins for Claude

[APIANT](https://apiant.ai) is an automation and integration platform your AI agent drives
directly: describe a workflow in plain language and the agent builds, tests and runs it.

- Website: [apiant.ai](https://apiant.ai)
- Sign up: [app.apiant.ai/register](https://app.apiant.ai/register)
- MCP server: `https://mcp.apiant.ai` (Streamable HTTP, OAuth sign-in)

## Install

```
/plugin marketplace add APIANT/apiant-plugins
/plugin install apiant@apiant
```

Or use **Settings → Plugins → Add from a repository**.

Any other MCP client: add `https://mcp.apiant.ai` as a remote HTTP server.

Agent skill (skills.sh): `npx skills add APIANT/apiant-plugins`

## Plugins

- **[`apiant`](plugins/apiant)** — build, test and run APIANT automations and app
  integrations from Claude. Connects Claude to the APIANT MCP server; you sign in with your
  APIANT account.

Questions: [support@apiant.com](mailto:support@apiant.com)
