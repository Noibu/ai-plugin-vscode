# Runtime: GitHub Copilot in VS Code

Read this once per session, before the first Noibu call. It describes what this host can and can't do.
Every Noibu skill in this plugin defers to it when it mentions "this host".

## Where you're running

- GitHub Copilot's agent in the VS Code Chat view. The Noibu tools come from the `noibu` MCP server that this plugin bundles (`https://mcp.noibu.com/mcp`). Each user signs in with their own Noibu account through VS Code's sign-in prompt.
- Tools are discovered by their unqualified `noibu_…` names. Never hardcode a server prefix.

## Slash commands

- Plugin skills are namespaced in VS Code. When you tell the user to run a skill, always write it as `/noibu:<skill-name>` (e.g. `/noibu:store-pulse`), never the bare `/<skill-name>`.

## Noibu issues: always numbered and linked

- **Every time any output mentions a Noibu error issue** (a JavaScript or HTTP error, or a finding tied to one), show its number as a link: `[#209](<issueUrl>)`. That covers chat, tables, lists, follow-ups, exported HTML (`<a href="<issueUrl>">#209</a>`), tickets and PRs. Never describe an issue without its number.
- Take `issueUrl` from the tool result (`noibu_list_priority_errors`, `noibu_search_errors`, `noibu_get_error`). Never build the URL yourself. **Performance opportunities (LCP, CLS, INP findings and detected page-speed issues) never get a link**, even if a URL is available: show their number as plain text, e.g. `#6`. The same goes for detected accessibility issues.
- The only exception is a message written for a third-party vendor: give the number as plain text there, since vendors can't open the Noibu console.

## Output

- Answers are Markdown in the chat: headers, tables, bullets. There's no chart, widget or card tool, so render charts as tables.
- **Questions:** VS Code has a built-in question tool that shows clickable options (single or multi-select, with a free-text box). Use it when the answer is a choice from a short known list: which store, which optional sections, which goal, yes/no on an offer. Ask open-ended questions (numbers, narratives, names) as plain text. Ask one question at a time either way. Where a skill says "plain text" or "no picker", that came from a host without this tool; prefer the question tool for short choices here.
- Deliverable files (reports, exports, the business-context file) go into the open workspace under `noibu/`, e.g. `noibu/checkout-analysis-example.com-2026-11-13.html`. Create the folder if needed. Name the file and its path in your reply so the user can open it from the Explorer.
- If no folder is open, give the content in the chat and tell the user to open a folder if they want it saved.

## The user's code

- The open workspace is often the storefront codebase: a Shopify theme, a headless frontend, or a platform cartridge. Check it before saying no code is available. Use the workspace search and read tools to find files, selectors, scripts and templates named in Noibu stack traces or page data.
- When the workspace holds the relevant code, a fix is a concrete file edit the user can review and accept. Make the edit only when they ask for a fix. Never commit, push, merge or deploy unless the user explicitly asks, and use the terminal only with their approval.
- "Draft a pull request" means: make the edit on a new branch, then open the PR with GitHub tools if they're in the session, or with `git` and `gh` in the terminal if the user approves. If neither is possible, leave the edit on the branch and say so.
- If the workspace isn't the storefront code, say so once in one sentence, and give the fix as a code or config snippet instead.

## Web pages

- VS Code has an **integrated browser** the agent can open pages in. When its tools are in the session, use it to read live storefront pages (read-only: never log in, submit forms, or check out). Otherwise use the web-fetch tool (`#fetch`). Read only what the skill needs, such as script `src` attributes or headings, and discard the rest.
- A Playwright MCP server, if present, works too. Never ask the user to install a browser tool mid-run, except where a skill says it's required.

## Not available on this host

- **Scheduling or recurring runs.** If asked, say VS Code doesn't run scheduled tasks and suggest rerunning the same slash command (for example `/noibu:store-pulse`) when they want a fresh read.
- **Saved live dashboards.** Every run is a fresh read.
- **Email, Slack or ticket delivery**, unless those tools are already in the session. Never name a missing integration.

## Connection problems

- If a `noibu_*` call fails because the user isn't signed in, or no Noibu tools are present, send them to `/noibu:setup-noibu`. Don't retry in a loop.
- Noibu tools only run in **Agent** mode. If the user is in Ask or Edit mode, tell them to switch.
