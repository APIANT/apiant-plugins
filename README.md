# APIANT plugins for Claude

A Claude Code plugin marketplace from [APIANT](https://apiant.ai).

## Install

In Claude, open the plugin manager and add this repository as a marketplace:

```
/plugin marketplace add APIANT/apiant-plugins
/plugin install apiant-credentials@apiant
```

Or use **Settings → Plugins → Add from a repository** and pick `apiant-credentials` from the
Discover tab.

Installing the plugin registers the APIANT MCP server (`https://app.apiant.ai/mcp`). The first
time a skill calls it, Claude signs you in with your APIANT account through the browser. **There
is no token to paste and no client ID to configure** — the server publishes OAuth metadata and
registers the client automatically.

## What's here

### `apiant-credentials`

One skill, `acquire-app-credentials`. It drives a browser through a vendor's developer portal to
obtain an app's credentials — signing in, registering the application, pasting the exact redirect
URI, selecting scopes, and saving the result where APIANT reads it back at connect time.

Claude does the walk. It stops and asks you only at the walls a browser cannot pass: a captcha, an
MFA prompt, a decision only you can make. Each of those is one precise question that ends the turn,
so your device's own notification reaches you and you can answer and walk away again.

It never enters a credit card, and it never starts a card-gated trial.

**Available to APIANT operators today.** The customer flow — where you already have the vendor
account and Claude finds the portal page, the scopes and the exact redirect URI for you — is
coming in a later release. Until then the skill will tell you so and point you at support.

## How this plugin stays correct

**The plugin ships procedure. The server ships facts.**

Redirect URIs, OAuth scopes, portal URLs, credential slot names, company identity and per-app
notes are not written into this repository. They are returned by the APIANT MCP server, at call
time, for the app being worked. A fact baked into a plugin can only be corrected by a release, and
every install stays wrong until it is updated — so there are none here to go stale.

That also means the skill discovers what *you* are entitled to from the server on its first call,
rather than asking you or guessing.

## Support

[support@apiant.com](mailto:support@apiant.com) · [apiant.ai](https://apiant.ai)
