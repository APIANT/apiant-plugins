---
name: acquire-app-credentials
description: Get an app's credentials by driving a browser through the vendor's own developer portal — signing in, registering the application, selecting scopes, and saving the result where APIANT reads it back at connect time. Claude drives; it asks you only for what it genuinely cannot do. Use when the user says "acquire credentials", "register an oauth app", "get the client id and secret", "I need my own keys for this app", "work the credential queue", or names an app whose credentials are missing.
user_invocable: true
---

# /acquire-app-credentials — Obtain an App's Credentials by Driving the Portal

Vendor developer portals are the same chore every time: find the right page, register an
application, paste the exact redirect URI, pick the scopes, copy two values back. This skill does
that walk in a browser and asks a human only at the walls a browser cannot pass.

**This skill ships procedure. Every FACT comes from the server.** Redirect URIs, scopes, portal
URLs, credential slot names, company identity, per-app gotchas — all of it is returned by the
tools below, at call time, for the app you are actually working. None of it is written down here,
and you must never supply one from memory, from the previous app, or by pattern-matching a
hostname. A remembered fact is a wrong fact one deploy later, and this file is public.

**Call every tool yourself.** Do not delegate to subagents or spawn child agents.

---

## Prerequisite: browser control

You need to drive a browser — read the page, click, type, navigate. Use whatever browser control
this session has (the host's built-in browser, an MCP browser server, a Playwright-based CLI) and
stay on one mechanism for the whole run.

If there is none, stop and say so:

> I need to drive a browser to work the vendor's portal, and this session has no browser control
> available. Enable one and re-run the skill.

Do not proceed past this check.

---

## Step 1 — Ask the server what you can do. This is your FIRST action.

Call `credential_capabilities()`. It takes no arguments.

The answer selects the lane:

| Field | What it means |
|---|---|
| `lane` | `operator` or `customer`. **This is the lane. Do not ask the user which lane they are in, and do not infer it from which server you are connected to.** |
| `can_claim`, `can_write_shared_creds`, `can_allocate_sms` | Which parts of the flow are reachable for this caller. A `false` means the tool will refuse, whatever any procedure says. |
| `tools` | The credential-lane tools this caller may call. Load them with ToolSearch — the MCP server prefix is install-specific, so search by keyword or `select:` rather than hardcoding a prefix. |
| `playbook` | For an entitled operator, the internal procedure, inline. Empty for everyone else. |

**If `playbook` is non-empty: follow it verbatim.** It is the current operator procedure, served
fresh, and it supersedes anything you remember about how this used to work. It is internal — do
not restate it to anyone outside the operator's organization, do not write it to a file, and do
not paste it into a commit, an issue, or a message.

**If `playbook` is empty and `lane` is `customer`:** see "The customer lane" below.

The tools named in `tools` are the whole surface. If one you expected is absent, this caller is
not entitled to it — say so plainly and stop, rather than looking for another way in.

---

## Step 2 — Get the app's facts before you open a portal

An operator's playbook owns the loop; the steps below are the portal procedure it points into,
and they are what the customer lane will run when it is enabled. Read both.

Once you know which app you are working, call
`credential_registration_kit(connector_uuid: "<uuid>")`. One call returns every fact for that app.

**Branch on `kit_status` first, before anything else:**

- **`already_done`** — the credentials are already stored. The server checked the credential store
  itself; this is ground truth, not a guess. **Stop. Do not register a second application on the
  vendor's console.** Report it as already done.
- **`not_applicable`** — this app has no application for us to register. Stop and say why.
- **`start_here`** — go work the portal.

Then read `progress` — the app's history, newest first. A previous session may have gotten further
than you think, including registering an application nobody recorded. **Before you create anything
on a vendor portal, search that portal for an application that already exists.** Do this
unconditionally, on every app, whatever the history says: the history can be missing an entry the
vendor's console still remembers, and a duplicate registration on a real developer console is the
expensive mistake this whole flow exists to avoid.

Use the kit's fields exactly as returned:

- `callback_url` — paste it byte-for-byte into the redirect/callback field. It was selected by this
  connector's own auth kind and read from the server's configuration. **Never derive one.**
- `scopes` — use the strings verbatim. An empty list is a real answer: many vendors define scopes
  on their own side at registration time, so the portal's own checkboxes are then the source of
  truth. Read them off the page and tell the user what you are selecting.
- `portal_url` — where to go. Empty means it is not recorded; ask, do not guess silently.
- `profile` — the company identity to type into the registration form: legal name, brand, website,
  support address, blurb, privacy and terms URLs, logo, application-name pattern. Use the values as
  given. If the portal wants the logo as a **file upload** rather than a URL, download the URL in
  `profile` to a temporary path and upload that file.
- `keyvault_slots` + `landing_tenant_uuid` — the exact slot names the credentials must be saved
  under, and where they live.
- `credential_params` / `placement` — for a key-bearing app, the fields to collect and where they
  ride on a request.
- `validation` — the call that proves the credential actually works.
- `gotchas` — what we already learned about this vendor's portal. Read them before you start.
- `signup_address` — the address to use if this app needs an account created.

**Treat the response as forward-compatible.** You may be running a plugin older than the server.
Ignore fields you do not recognize, and degrade gracefully when an optional one is absent — never
assume a field is there because it was last week.

---

## Step 3 — Work the portal

Loop: read the page → say in one short sentence what you are about to do → act if it is
unambiguous → if it is a wall, poke (below). Repeat.

- After every navigation, submit, or click, **wait for the page to settle** before reading it
  again. Async portals will hand you a stale DOM if you chain clicks.
- **If a selector fails or the flow diverges from what you expected, stop and ask.** Do not retry
  blindly — a wrong click can submit a half-filled form or create a second application.
- Many portals show the secret **exactly once**. Capture it on the success page, immediately,
  before any navigation.
- **Record the milestone the server cannot see.** Registering the application on the portal is
  invisible to the server, so record it the moment the portal shows it, before you do anything
  else. The other milestones are recorded for you by the tools that perform them.

Then save the credentials into the slots the kit named, and run the kit's `validation` call to
prove they work.

---

## The poke protocol — how to ask for a human

When you hit something you cannot do: **ask ONE precise question and END THE TURN.** That is the
whole mechanism. The host's own notification reaches the person's phone; there is no notify tool,
no channel to configure, and nothing to set up.

**The first words of your question must say which kind of help it is**, because we do not control
the text of that notification and the first words are what the person sees:

> **Needs you at your desk** — the Google sign-in page is showing a captcha. …

> **Answerable from your phone** — which Salesforce org should this app live in, production or
> sandbox? …

Then stop. Do not narrate, do not keep working, and do not ask a second question in the same turn.

### Which wall gets a poke

| Wall | What to do |
|---|---|
| **Captcha** | Poke. Needs them at their desk. |
| **MFA / authenticator app** | Poke. |
| **SMS one-time code** | **Do NOT poke** if a pool number was allocated for this app — read the code with `get_sms_code`. Poke only if there is no number or the vendor rejects it. |
| **Credit card / card-gated trial** | **Never poke, never enter a card, never start the trial.** Park the app and move on. This one is absolute. |
| **KYC / manual account review** | Park the app. Nobody can unblock it in this session. |
| **Email verification** | Read it with the tools your lane has; poke only if you cannot. |
| **Anything ambiguous** | Poke once. If it is still ambiguous, park. |

"Park" means record the app's disposition with the reason and release it, so the next pass knows
why and does not re-attempt it forever.

---

## Browser discipline — work ONE app at a time

**The browser profile is shared across tasks, account-wide.** When you start an app, the previous
app's vendor session is very likely still signed in, and a session opened in another task can be
signed in as somebody else.

- Work exactly one app at a time. Finish it or release it before starting another.
- Before registering anything, **check which identity the portal thinks you are.** A registration
  made under the wrong signed-in account is silent, wrong, and expensive to unwind.
- Never open a second app "while this one waits". A wait is a poke, and a poke ends the turn.

---

## The customer lane

**Not enabled yet.** The customer lane ships in a later release, once the operator lane has cleared
its watched first run. If `credential_capabilities` returns `lane: "customer"`, say so and stop:

> This skill's customer flow isn't enabled yet — today it runs for APIANT operators only. Your
> account can reach the credential tools, but the guided flow is off. Contact APIANT support and
> we'll get the app connected for you.

When it does turn on, the flow **inverts**. A customer already has the vendor account; what they
lack is knowing which portal page, which scopes, and which exact redirect URI. So the handoffs
reverse:

- **The person does the login and the MFA.** Never ask for a password, and never type one.
- **You do the portal navigation, the scope selection, and copying the values back.** That is the
  part they came for.
- **A timeout is never a silent abandon.** If they walk away, record where you got to, tell them
  exactly what remains and how to resume, and end the turn. Coming back has to be cheap.

---

## Gotchas

- **A fact you did not get from the kit this run is a wrong fact.** Redirect URIs and scopes drift,
  and this plugin can be a week older than the server it is talking to.
- **The redirect URI must match byte-for-byte.** A trailing slash or a different host makes the
  vendor reject the consent redirect, and it surfaces much later as a broken connect.
- **Scopes that look alike are not alike.** `read_user` and `user:read` belong to different
  vendors. Use the exact string.
- **Sandbox is not production.** Where a vendor separates them, the person chooses — you do not.
- **Secrets never go into a note, a disposition reason, a commit, or a question you ask a human.**
  A secret read off a page lands in your context; store it and move on, and never repeat it back.
- **The credential store never hands a value back.** You can see whether a slot is filled, not what
  is in it. Replacing a credential means capturing or pasting the value again.
- **A refusal from a tool is the answer.** The gates live in the tools, not in this text. If a tool
  refuses, that caller is not entitled — report it, do not route around it.
