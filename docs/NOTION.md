# Notion + Composio setup

The plugin writes each ingested post as one row in a Notion **database**, using
[Composio](https://composio.dev)'s `NOTION_INSERT_ROW_DATABASE` tool. You need three things:

1. A Notion database with the right columns.
2. Notion connected inside Composio.
3. Those two identifiers saved to `config/notion.json`.

---

## 1. Create the Notion database

Make a new **full-page database** (Table view) in Notion — e.g. **"Social Media Posts"**.
Add these properties **exactly** (names and types are case-sensitive):

| Property name | Type          |
|---------------|---------------|
| `Name`        | Title         |
| `Post Link`   | URL           |
| `Type`        | Select        |
| `Author`      | Text          |
| `Topics`      | Multi-select  |
| `Ingested At` | Date          |

`Name` already exists as the default title column — just rename it if needed. You don't need
to predefine Select/Multi-select options; they auto-create on first use.

### Find the `database_id`

Open the database as a full page, copy the URL. It looks like:

```
https://www.notion.so/yourworkspace/3a163972c7eb8080a003d9531bf5eac9?v=...
                                    └────────── database_id ──────────┘
```

The 32-character hex string (dashes optional) is your `database_id`.

---

## 2. Connect Notion in Composio

1. Make sure the **Composio MCP server is enabled in Claude** (Claude Desktop/Code → MCP
   servers → Composio). Ask Claude: *"list my Composio connections"* to confirm it's live.
2. Authorize Notion: ask Claude *"connect my Notion account in Composio"* (it will use
   `COMPOSIO_MANAGE_CONNECTIONS`), or add the Notion connection from the Composio dashboard.
3. **Share the database with the connection.** In Notion, open the database → `•••` →
   *Connections* → add the Notion integration Composio uses, so it has write access.

### Find your `account` value

`account` is the Composio **connected-account alias** for your Notion login. To find it, ask
Claude: *"list my Composio Notion connected accounts"* — it will show an alias/id like
`notion_yourname`. Use that string. (If you only have one Notion connection, Claude can
usually resolve it automatically, but setting it explicitly is more reliable.)

---

## 3. Save `config/notion.json`

Copy the template and fill in your two values:

```bash
cp config/notion.example.json config/notion.json
```

```json
{
  "account": "notion_yourname",
  "database_id": "3a163972c7eb8080a003d9531bf5eac9"
}
```

That's it. Run `python3 scripts/doctor.py` — the Notion line should turn green. This file is
`.gitignore`d, so your identifiers never get committed.

> Prefer a ready-made database? Duplicate this public template and use its `database_id`:
> **(add your own Notion template share link here before publishing).**
