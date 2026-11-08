---
name: apiant
description: Build, test and run app automations and integrations on APIANT (apiant.ai) — "when a new Shopify order arrives, add the customer to Mailchimp", "why did my invoice sync fail", "connect my CRM to my billing app". Use when the user wants to connect apps, automate a workflow between SaaS tools, sync data between systems, or fix an automation that failed.
---

# APIANT

[APIANT](https://apiant.ai) is an automation and integration platform that AI agents drive
directly. You describe the workflow; the agent builds, tests and runs it in the user's APIANT
account.

## Connect

Add the APIANT MCP server (Streamable HTTP, OAuth sign-in — no API key):

```
https://mcp.apiant.ai
```

- **Claude Code:** `claude mcp add --transport http apiant https://mcp.apiant.ai`
- **Claude plugin:** `/plugin marketplace add APIANT/apiant-plugins` then `/plugin install apiant@apiant`
- **Any other MCP client:** add `https://mcp.apiant.ai` as a remote HTTP server.

The first call opens the APIANT sign-in page in the browser. No account yet? Sign up at
[app.apiant.ai/register](https://app.apiant.ai/register).

## How to work

The server exposes three tools:

1. **`apiant_skill_load`** — call it with no name first for the guide index, then load the guide
   the task needs (building an automation, editing one, testing, connecting an app, adding an
   app that is not in the catalog yet) and follow it. The guides carry the platform's rules; the
   tool descriptions alone do not.
2. **`apiant_search_tools`** — find the operation for the step you are on.
3. **`apiant_call`** — run it in the user's account.

## Rules

- Never turn automations on or off, publish, deploy, or halt runs without the user's explicit
  approval of the exact change.
- The user connects their own apps; when an app needs credentials, the connect step returns a
  link for them to open.
- If an app or operation is missing, the guides cover adding it — do not tell the user it is
  impossible.

Support: [support@apiant.com](mailto:support@apiant.com)
