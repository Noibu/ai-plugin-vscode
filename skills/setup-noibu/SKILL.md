---
name: setup-noibu
description: "Connect Noibu in VS Code and confirm your site is sending data. Run this first after installing the Noibu plugin: it signs you in to Noibu, picks your store, checks that sessions are flowing, and shows what to ask next. Use for 'set up Noibu', 'connect Noibu', 'is Noibu working', or sign-in problems."
argument-hint: "[your store domain, e.g. www.example.com]"
---
<!-- overlay: vscode — rewritten for VS Code: sign-in through the bundled MCP server's prompt; derived from the Cowork setup-noibu skill -->

You're running the Noibu onboarding flow in VS Code: confirm Noibu is signed in, pick the store, then confirm the site is actually sending data.

Run every step on every invocation. Don't assume prior state.

Read `../querying-noibu-data/references/vscode-runtime.md` for what this host can do.

---

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "setup-noibu"`, `skillVersion: "vscode:1.0.0"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## 1. Check the sign-in

If you're not in Agent mode (no tools available), say: "Switch the chat to **Agent** mode, then run `/noibu:setup-noibu` again." Stop.

Call `noibu_get_current_user`.

**If it succeeds:** say Noibu is connected and greet them by first name if the result has one. Go to step 2.

**If the Noibu tools aren't in the session, or the call fails because they aren't signed in:** explain in one or two sentences what connecting unlocks: plain-English questions about their store's errors, checkout drop-off, page speed and conversion. Then give these steps exactly:

> To sign in:
> 1. Open the Command Palette (**Cmd+Shift+P** on Mac, **Ctrl+Shift+P** on Windows) and run **MCP: List Servers**.
> 2. Select **noibu**, then **Start Server**.
> 3. When VS Code asks to allow the sign-in, select **Allow** and sign in to Noibu in the browser window that opens.
>
> Then reply **done** and I'll pick up from here.

When they reply, call `noibu_get_current_user` once more.
- If it succeeds, go to step 2.
- If the Noibu tools still aren't in the session, ask them to open a **new chat** in Agent mode and run `/noibu:setup-noibu` again. VS Code sometimes only picks up a newly started server in a new chat.
- If it still fails, go to **Still not connecting** below. Don't keep retrying.

## 2. Pick the store

If the user named a domain (for example in the slash command), resolve it with `noibu_get_domain`.

Otherwise call `noibu_list_domains` (it returns up to 50 stores by default; `flatLimit` raises that). Each row carries its own `companyId`.
- Exactly one domain: use it.
- 2–10: ask in plain text which store to set up, listing them by name. Don't guess.
- More than 10: don't list them. Ask them to type the store's domain (e.g. `www.example.com`) and resolve it with `noibu_get_domain`.
- None: say their Noibu login isn't linked to any store yet, and they should ask their Noibu admin or Noibu contact to add them. Stop.

Keep the domain's UUID `id` and its `companyId`. The next step needs both.

## 3. Confirm data is flowing

Call `noibu_check_data_connection`:

```
noibu_check_data_connection({
  domainId: "<resolved-domain-uuid>",
  companyId: <companyId from the domain lookup>,
  rationale: "Onboarding — confirming the customer's site is sending session data to Noibu",
  skill: "setup-noibu",
  skillStep: "check-data-connection"
})
```

**If `isReceivingData` is true:** confirm it plainly, then finish with step 4.

**If `isReceivingData` is false:** say the connection works but you're not seeing recent sessions from [domain]. That usually means the Noibu script isn't on the live site yet, or it was just added and hasn't caught up. Suggest they check the install with their dev team and run `/noibu:setup-noibu` again later. Stop.

Keep this step short. It's a connectivity check, not a diagnosis.

## 4. What to try next

End with a short list. Keep it to these, with their domain filled in:

> You're all set. Try:
> - `/noibu:store-pulse` — how [domain] did in the last 24 hours
> - `/noibu:tech-diagnosis` — what's broken and what to fix first
> - `/noibu:checkout-analysis` — where shoppers drop off in checkout
> - `/noibu:build-business-context` — teach me about your business so answers get sharper
>
> Or just ask in plain English, like "What's my mobile conversion rate this week?"

If the open workspace looks like their storefront code (for example a Shopify theme with `layout/theme.liquid`, or a frontend app with a `package.json`), add one line: "Because your store's code is open here, I can also suggest fixes directly in your files."

## Still not connecting

Give these checks in order, briefly:

1. **Company plan (Copilot Business or Enterprise):** your GitHub admin has to turn on the **MCP servers in Copilot** policy, and allow plugins, for Noibu to load.
2. **Server status:** run **MCP: List Servers**, select **noibu**, then **Show Output** to see the error.
3. **Sign-in:** make sure you're signing in with the same email you use for console.noibu.com.

If none of that helps, they should contact their Noibu account team or support@noibu.com with the server output.

## Tone

- This skill is for Noibu's customers. Write as a friendly Noibu product expert helping someone get set up for the first time.
- Use their actual domain name. One clear action at a time.
- Avoid the words "OAuth", "MCP" and "registry" in your own sentences. Say "connect" and "sign in". The VS Code command names in the steps above are the exception, because the user has to click them.
