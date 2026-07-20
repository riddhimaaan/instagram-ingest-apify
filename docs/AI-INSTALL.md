# Install runbook (for an AI coding agent)

> **You are an AI agent (Claude Code, Cursor, Codex, etc.) installing the
> `instagram-ingest-apify` Claude Code plugin for a user on their machine. Follow these steps
> in order. Run one command at a time, show output, and stop to ask the user only where
> marked 🧑.**

This plugin turns an Instagram URL into a clean, formatted row in the user's Notion database.
Scraping and transcription run on **Apify's cloud** — there is nothing to install locally (no
ffmpeg, no Python venv, no WSL). It needs: an Apify account + MCP connection, a
Composio→Notion connection, and the user's own Instagram cookies.

---

## Step 0 — Set expectations, then gate on Apify and Composio

Before doing anything else, tell the user plainly what this install requires beyond Claude
Code itself: **an Apify account with its MCP server connected** (this is what runs the
scraper — non-negotiable), **the Composio MCP server connected to Notion** (non-negotiable,
the only way this plugin writes to Notion), and **their own Instagram cookies** (a 2-minute
browser export, not code — Step 4). Say this up front so it isn't a surprise later.

Then, **before Step 1**, check both of these are connected — a hard stop if either is
missing, so the user isn't left mid-install with a plugin that can't run anything:

- **Apify** — try calling an Apify MCP tool (e.g. `fetch-actor-details` on
  `industrial_nightfall/instagram-ingest-scraper`). If unavailable, walk the user through
  `docs/APIFY.md` (create a free Apify account if needed, connect the MCP server), then wait
  for confirmation.
- **Composio → Notion** — try listing Composio connections. If unavailable, walk the user
  through connecting it (claude.ai/Desktop Connectors UI, or `claude mcp add --transport http
  composio https://connect.composio.dev/mcp` for the CLI), then wait for confirmation.

If either doesn't match what the user sees (these connector UIs move fast), don't guess —
fetch the current docs (`https://docs.apify.com/platform/integrations/mcp` or
`https://composio.dev/toolkits/notion/framework/claude-code`) before improvising.

## Step 1 — Install the plugin into Claude Code
Tell the user to run these in Claude Code (these are Claude Code slash commands — the user
runs them, you cannot):
```
/plugin marketplace add riddhimaaan/instagram-ingest-apify
/plugin install instagram-ingest-apify@instagram-ingest-apify
```
The plugin files land under the user's Claude plugins directory — this is
`${CLAUDE_PLUGIN_ROOT}` when the plugin runs. Call it `$PLUGIN` below.

## Step 2 — Instagram cookies 🧑
Follow `docs/COOKIES.md`:
- Ask the user to export cookies from a **burner** Instagram account with the Cookie-Editor
  browser extension, **Export as Netscape**, and save to `~/.config/ig-ingest/cookies.txt`.
- You cannot do this for them (it requires their browser session). Wait for them to confirm
  the file exists, then continue.

## Step 3 — Notion database + Composio 🧑
Follow `docs/NOTION.md`:
1. Ensure a Notion database exists with these exact properties: `Name` (Title), `Post Link`
   (URL), `Type` (Select), `Author` (Text), `Topics` (Multi-select), `Ingested At` (Date).
2. Ensure Notion is connected in Composio and the database is shared with that connection.
3. Write `$PLUGIN/config/notion.json`:
   ```json
   { "account": "<composio-notion-account-alias>", "database_id": "<notion-database-id>" }
   ```
   You can discover the account alias by asking the Composio MCP to list Notion connections,
   and the database_id from the Notion database URL (32-hex chars).

## Step 4 — Verify
There's no install script to run. Check conversationally:
- Apify MCP reachable (Step 0) ✓/✗
- Composio → Notion connected (Step 0) ✓/✗
- `~/.config/ig-ingest/cookies.txt` exists and is non-empty (Step 2) ✓/✗
- `$PLUGIN/config/notion.json` exists with no placeholder values (Step 3) ✓/✗

Fix anything that's a ✗ before continuing.

## Step 5 — First run
There is no slash command to run — the plugin is a single skill (`ig-ingest`) that Claude
invokes automatically whenever a message contains an instagram.com URL. Tell the user to just
paste a link in Claude Code, e.g.:
```
Ingest this: https://www.instagram.com/reel/XXXXXXXXX/
```
Expected result: one formatted row appears in their Notion database. Report the Notion page
URL back to the user. This run happens on Apify's infrastructure and bills the user's own
Apify account — see `docs/APIFY.md`'s cost note.

---

### Troubleshooting quick table
| Symptom | Fix |
|---|---|
| Actor run FAILED, no obvious reason | Cookies expired → re-export `~/.config/ig-ingest/cookies.txt` (`docs/COOKIES.md`). This is the #1 cause. |
| `fetch-actor-details`/`call-actor` not available | Apify MCP isn't connected — see `docs/APIFY.md`. |
| Notion write errors on a property | Property names/types must match the table in `docs/NOTION.md` exactly. `Type` is a Select. |
| `config/notion.json missing` | Copy `config/notion.example.json` → `config/notion.json` and fill it in. |
| Run succeeds but images won't display | The `get-key-value-store-record` signed URL expired or wasn't re-fetched — always download fresh, don't cache URLs across turns. |
