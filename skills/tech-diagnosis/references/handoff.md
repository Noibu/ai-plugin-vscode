<!-- overlay: add — derived-from: tech-diagnosis/references/share.md @ f57ec16dd2f2 -->
<!-- overlay: add — derived-from: tech-diagnosis/references/ticket.md @ 077ad4dfdc24 -->

# Handoff (ticket + report paths)

Reached only if the operator chose to share/ticket from the closing offer.

### First question (conversational)

> Share as tickets for follow-up, or as a report to send someone?

### Ticket path

Full content always included (Summary + Technical details + Cause + Fix). No content-selection step.

1. **Detect the connected ticket connector** (Linear / Jira / GitHub Issues / Notion task database).
   - **One connected** → use it directly.
   - **None present** → this path should not have been offered (see SKILL.md Step 5). If reached anyway, say the ticket path isn't available in this session and offer the report path instead. Never tell them to add a connector.
   - **Multiple connected** → conversational question: which one?
2. **Ask for project/repo** if not cached. Save to config.
3. **Create tickets** — one per eligible finding (fixable JS errors and performance issues only — not vendor issues, which are third-party origin and not actionable by the operator's team):
   - Title: `[severity] [summary] (affecting [segment])`.
   - Body in markdown: Summary → Technical details → Cause → Fix.
   - Priority mapping (where supported): critical → high; standard → medium; third-party-origin → default/low.
   - Labels (only if pre-existing): `tech-diagnosis`, `error` or `performance`, severity, `noibu`.
   - Assignee: unset unless operator specifies.
4. **Surface ticket URLs** after creation. Report partial failures honestly.


#### Automatically draft a pull request (PR)

Only available when **both** conditions are met:
- The code is reachable, either in the open workspace (a git repo holding the affected file) or through repo tools (e.g. GitHub) in the session. If you only have repo tools and don't know which repo to target, ask once ("What's the repo? e.g. `mycompany/my-store`"). One question, not a setup flow.
- A specific file and line was identified from code enrichment.

If either condition isn't met, do not offer or mention this option.

When both are met:
1. **Confirm the fix** — briefly restate the specific change in plain language and ask the operator to confirm before touching their code.
2. **Create a branch** — name it descriptively (e.g. `fix/[plain-English-description]`).
3. **Apply the fix.** Write the exact code change to the correct file. In the open workspace, use a normal file edit the user can review.
4. **Open a review request.** Title: a plain-English summary of what was fixed. Body: Summary → Cause → Fix in Markdown, with a link to the Noibu finding by issue ID. In the workspace, commit the change and open the PR with GitHub tools if they're present, or with `git push` + `gh pr create` in the terminal if the user approves each command. If neither works, leave the commit on the branch and tell them the branch name so they can push it themselves.
5. **Confirm it's ready** — surface the link and tell them someone needs to review and approve it before it goes live.

### Report path

Ask as plain conversational text:

> Where would you like this sent? Options: a document (saved in your workspace)[, plus each of Email / Teams / Slack / Notion whose tools are in this session]. And what should it include beyond the summary — cause, fix, technical details? Reply with your choices.

- **Destinations** (operator can pick more than one): document — always available. Email, Teams, Slack, Notion — list one only if its tools are in the session. Never name a destination that isn't reachable, and never explain the omission.
- **Include** (beyond the always-included Summary): Cause and Fix — included by default unless the operator says otherwise. Technical details — only if requested.

**Conversational follow-up** (after the operator replies with their choices, for each selected destination that needs recipient details):
- Email → "What email address?"
- Teams → "Which channel or chat?"
- Slack → "Which channel?"
- Document → no follow-up; save to the workspace and state the filename.
- Notion → "Which page or database?"

Multiple destinations are processed in parallel — write the document, draft the email, post to the channel, etc. — and follow-up asks come as needed per destination.

**Generate the share artifact** with the Summary always present plus any selected add-ons, in this order: Summary → Technical details → Cause → Fix.

**Static-document rewrite required.** Strip all chat-only constructs:
- No "let me know if" / "want me to walk through" hooks.
- No pause-for-direction text.
- Alternative-fix mentions become static framing ("If the primary fix doesn't resolve, the alternative is [X]") not conversational.
- The reasoning-vs-measurement disclaimer, if included, becomes a single-line static caveat at the top.

**Per-channel:**
- **Document:** write it to the open workspace as `noibu/tech-diagnosis-[domain]-[date].html` (see `vscode-runtime.md`). State the path. Tell the operator plainly it's an HTML file, not a PDF, and that they can open it in a browser and print to PDF if they need that format. If no folder is open, put the content in the chat instead.
- **Email** — subject `Tech findings for [domain] — [N] items to investigate`. HTML body, scannable.
- **Teams** — lead-in `**Tech findings for [domain]** — [N] items, last [window].` One section per finding.
- **Slack** — lead-in `*Tech findings for [domain]* — [N] items, last [window].` One block per finding.
- **Notion** — title `Tech findings for [domain] — [date]`. Sectioned page.

### Issue links in shared content (both paths)

Same rule as in-chat: every issue shows its number linked to `issueUrl`, in tickets, PR descriptions and reports. Replay and other console links only on explicit request. Vendor-facing messages: number as plain text, no link.
