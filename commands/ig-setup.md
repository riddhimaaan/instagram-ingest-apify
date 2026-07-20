---
description: Connect Apify + Composio, set up Instagram cookies and Notion config for instagram-ingest-apify. No local installs — this checks connections, not packages.
---

Set up the `instagram-ingest-apify` plugin for this user. Do it in order and report results plainly.

## Step 0 — Tell them what's required

Before doing anything, tell the user plainly what this plugin needs beyond Claude Code
itself — there is **no local install of anything** (no ffmpeg, no Python, no WSL); everything
below is a one-time connection or a one-time file:

1. **An Apify account**, with the Apify MCP server connected to this Claude Code session.
   Scraping and transcription run on Apify's cloud, not the user's machine.
2. **The Composio MCP server connected to Notion** — non-negotiable, it's the only way this
   plugin writes to Notion.
3. **Their own Instagram cookies** (a 2-minute browser export, not code — `docs/COOKIES.md`).

Say this up front so it isn't a surprise partway through.

## Step 1 — Check Apify

Try calling an Apify MCP tool (e.g. `fetch-actor-details` on `industrial_nightfall/instagram-ingest-scraper`,
the public Actor this plugin drives). If it resolves, Apify is connected — move on.

**If no Apify tool is available**, walk the user through connecting it, then wait for
confirmation before continuing:
- **claude.ai / Claude Desktop:** `claude.ai/settings/connectors` → **+** → find/add **Apify**
  → **Connect** → sign in / create a free Apify account if they don't have one.
- **Claude Code CLI:** the user needs an Apify API token (from
  `console.apify.com/settings/integrations`), then:
  ```bash
  claude mcp add --transport http apify https://mcp.apify.com --headers "Authorization:Bearer <their-apify-token>"
  ```
  then `exit` and restart `claude`; confirm with `claude mcp list`.
- If this doesn't match what the user sees (connector UIs change), fetch
  `https://docs.apify.com/platform/integrations/mcp` for the current steps rather than guessing.

## Step 2 — Check Composio → Notion

Same pattern as Apify: try listing Composio connections / calling a Composio MCP tool.

- **If available:** list their Notion connections now — you'll want the account alias for
  Step 4 — and move on.
- **If not available**, walk them through whichever matches how they run Claude, then wait for
  confirmation:
  - **claude.ai / Claude Desktop:** `claude.ai/settings/connectors` → **+** → find/add
    **Composio** → **Connect** → approve the OAuth popup. Not listed? Add as a custom
    connector with URL `https://connect.composio.dev/mcp`.
  - **Claude Code CLI:** `claude mcp add --transport http composio https://connect.composio.dev/mcp`,
    then `exit` and restart `claude`; confirm with `claude mcp list`.
  - Either way, the first Notion action triggers a one-time Notion OAuth link to approve.
  - If this doesn't match what the user sees, fetch
    `https://composio.dev/toolkits/notion/framework/claude-code` for the current steps.

## Step 3 — Instagram cookies 🧑

Follow `docs/COOKIES.md`: the user exports cookies from a logged-in (ideally burner) Instagram
account via the Cookie-Editor browser extension, **Export as Netscape**, and saves to
`~/.config/ig-ingest/cookies.txt`. You cannot do this for them — it requires their browser
session. Wait for them to confirm the file exists before continuing.

Create the folder for them if it's missing (`~/.config/ig-ingest/`) so they just have to paste.

## Step 4 — Notion database + config

Follow `docs/NOTION.md`:
1. Ensure a Notion database exists with these exact properties: `Name` (Title), `Post Link`
   (URL), `Type` (Select), `Author` (Text), `Topics` (Multi-select), `Ingested At` (Date).
2. Ensure Notion is connected in Composio and the database is shared with that connection.
3. Write `${CLAUDE_PLUGIN_ROOT}/config/notion.json`:
   ```json
   { "account": "<composio-notion-account-alias>", "database_id": "<notion-database-id>" }
   ```
   You likely already have the account alias from Step 2 — if you also know the database_id,
   offer to write the file for them instead of making them do it by hand.

## Step 5 — Verify

There's no `doctor` script to run — check conversationally instead:
- Apify MCP reachable (Step 1) ✓/✗
- Composio → Notion connected (Step 2) ✓/✗
- `~/.config/ig-ingest/cookies.txt` exists and is non-empty ✓/✗
- `config/notion.json` exists and has no placeholder values ✓/✗

Report all four plainly. When all four pass, tell the user they can run `/ingest <instagram-url>`.
