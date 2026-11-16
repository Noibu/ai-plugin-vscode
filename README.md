# Noibu AI Plugin for VS Code

Official Noibu plugin for GitHub Copilot in VS Code. Ask about your store's sessions, conversion, errors and page speed in Copilot Chat. Because Copilot is working in your store's code, you can go straight from a Noibu finding to a fix in your own files.

## Install

**[Install in VS Code](https://vscode.dev/redirect?url=vscode%3A%2F%2Fchat-plugin%2Finstall%3Fsource%3DNoibu%252Fai-plugin-vscode%2526plugin%3Dnoibu)**

Then open Copilot Chat in **Agent** mode and type:

```
/noibu:setup-noibu
```

It signs you in to Noibu, picks your store and checks that data is flowing. That's the whole setup: no config files, no API keys.

**If the install link doesn't open VS Code:** open the Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`), run **Chat: Install Plugin from Source**, and enter `Noibu/ai-plugin-vscode`.

Installing adds the **noibu** marketplace, so VS Code picks up new versions of the plugin automatically.

### Requirements

- VS Code with GitHub Copilot, signed in to GitHub.
- A Noibu login with access to your store.
- **Copilot Business or Enterprise:** your GitHub org admin must turn on the **MCP servers in Copilot** policy (off by default), and allow agent plugins if your org restricts them.

## What's included

| Skill | What it does |
|---|---|
| `/noibu:setup-noibu` | Sign in to Noibu, pick your store, confirm data is flowing |
| `/noibu:store-pulse` | At-a-glance store health for the last 24 hours |
| `/noibu:tech-diagnosis` | Errors and Core Web Vitals, with root cause, and a fix in your code when it's open in VS Code |
| `/noibu:checkout-analysis` | Checkout funnel drop-off, payment and delivery mix, checkout errors |
| `/noibu:product-analysis` | Product and collection performance, views without sales |
| `/noibu:segment-analysis` | Channel, device and country performance |
| `/noibu:accessibility-audit` | WCAG 2.1 AA audit of your top pages, with optional fix PRs |
| `/noibu:ab-test-opportunity-finder` | Evidence-backed A/B test ideas, created as Noibu drafts on request |
| `/noibu:build-business-context` | Teaches Noibu about your business so every answer gets sharper |
| `querying-noibu-data` | Shared query rules the other skills load automatically (hidden from the `/` menu) |

Plugin skills are namespaced, so type `/noibu:` to see them all in the chat menu. You don't have to use a slash command, though: ask in plain English ("What's my mobile conversion rate this week?") and Copilot picks the right skill.

Reports and exports are saved in your workspace under `noibu/`.

## Good to know

- Noibu tools only run in **Agent** mode.
- Each teammate signs in with their own Noibu login.
- `/noibu:accessibility-audit` scans pages in VS Code's built-in browser, so there's nothing extra to install. If your VS Code doesn't offer browser tools, it will ask you to add the Playwright MCP server.
- The Noibu skills tell Copilot never to commit, push, merge or deploy unless you ask, and VS Code asks before running terminal commands.

## Troubleshooting

- **Noibu tools missing:** run **MCP: List Servers** → **noibu** → **Start Server**, sign in, then start a new Agent-mode chat.
- **No sign-in prompt:** on Copilot Business or Enterprise, confirm the MCP policy is on for your org.
- **Anything else:** **MCP: List Servers** → **noibu** → **Show Output**, and send that to your Noibu contact or support@noibu.com.

## Maintaining the install link

The button uses VS Code's install link wrapped in a `vscode.dev/redirect` link (GitHub and most help centers strip `vscode://` links). The redirect page drops everything after the first `&`, so the `&` before `plugin=` is **double-encoded** (`%2526`). Build it with:

```python
from urllib.parse import quote
inner = "vscode://chat-plugin/install?source=" + quote("Noibu/ai-plugin-vscode", safe="") + quote("&", safe="") + "plugin=noibu"
print("https://vscode.dev/redirect?url=" + quote(inner, safe=""))
```

## Repository layout

```
plugin.json   # Agent Plugins 1.0 manifest
.github/plugin/marketplace.json   # one-plugin catalog; the install link needs it
mcp.json      # Noibu MCP server (https://mcp.noibu.com/mcp); sign-in handled by VS Code
skills/       # One folder per skill: SKILL.md + references/
assets/       # Logos
```

The skills are derived from the Microsoft Copilot Cowork variant ([Noibu/ai-plugin-copilot-cowork](https://github.com/Noibu/ai-plugin-copilot-cowork)), which in turn derives from the Claude plugin ([Noibu/ai-plugin](https://github.com/Noibu/ai-plugin)). VS Code-specific changes are marked with `<!-- overlay: vscode … -->` comments, and the host rules all skills share live in `skills/querying-noibu-data/references/vscode-runtime.md`. Noibu calls are tagged `skillVersion: "vscode:<version>"` so VS Code usage can be told apart from other clients.

## License

MIT
