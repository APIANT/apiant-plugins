# APIANT for Claude

[APIANT](https://apiant.ai) is an automation and integration platform. This plugin lets you
build, test and run APIANT automations from Claude by describing what you want in plain
language: "when a new order arrives in Shopify, add the customer to Mailchimp", "why did my
invoice sync fail last night", "add a step that emails me on errors".

## What it installs

One thing: a connection to the APIANT MCP server at `https://mcp.apiant.ai`. The plugin
contains no code, scripts, hooks or bundled skills.

The server offers three tools:

- **`apiant_skill_load`** loads a how-to guide (for example, building an automation or
  connecting an app) so Claude follows APIANT's own procedure.
- **`apiant_search_tools`** finds the APIANT operation that fits the task.
- **`apiant_call`** runs that operation in your APIANT account.

## Signing in

The first time Claude uses APIANT, your browser opens the APIANT sign-in page. Sign in with
your APIANT account. There is no API key or token to paste. Every call then runs as you, with
your account's permissions, and can only see and change your own APIANT data.

Don't have an account? Sign up at [apiant.ai](https://apiant.ai).

## What is sent where

Your requests, and the automation and app data they involve, go only to APIANT's server at
`mcp.apiant.ai`. APIANT then calls the third-party apps you have connected in your account,
as your automations require. See the [privacy policy](https://apiant.ai/privacy) and
[terms of service](https://apiant.ai/tos).

## Support

Email [support@apiant.com](mailto:support@apiant.com).
