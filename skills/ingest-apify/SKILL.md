---
name: ingest-apify
description: Download an Instagram reel, post, or carousel via the Apify Actor, analyze its full content, and store the extracted content in the user's Notion "Social Media Posts Link" database via Composio. Use whenever the user pastes an instagram.com URL and asks to save/ingest/analyze it, or asks to set up/configure this Instagram-to-Notion ingest.
---

# Instagram → Notion ingest (Apify-backed)

Given one Instagram URL, scrape + transcribe it on Apify, analyze it, and write a single
formatted row to Notion. **Media is never uploaded to Notion** — only the post link +
extracted content. **Nothing installs locally** — no ffmpeg, no Python venv, no WSL. The only
local things this skill touches are the user's cookies file (read, never written by this
skill) and a small temp folder for images it needs to view.

## Step 0 — First-time setup check

Run this once per user, the first time this skill is invoked in a session where you haven't
already confirmed all four items below. If all four already check out, skip straight to Step 1.

There is **no local install of anything** (no ffmpeg, no Python, no WSL) — everything here is
a one-time connection or a one-time file. Tell the user this up front if any item is missing,
so it isn't a surprise partway through.

1. **Apify connected.** Try calling an Apify MCP tool (e.g. `fetch-actor-details` on
   `industrial_nightfall/instagram-ingest-scraper`, the public Actor this skill drives). If it
   resolves, move on. If not, walk the user through connecting it, then wait for confirmation:
   - **claude.ai / Claude Desktop:** `claude.ai/settings/connectors` → **+** → find/add **Apify**
     → **Connect** → sign in / create a free Apify account if they don't have one.
   - **Claude Code CLI:** get an Apify API token from
     `console.apify.com/settings/integrations`, then:
     ```bash
     claude mcp add --transport http apify https://mcp.apify.com --headers "Authorization:Bearer <their-apify-token>"
     ```
     then `exit` and restart `claude`; confirm with `claude mcp list`.
   - If this doesn't match what the user sees (connector UIs change), fetch
     `https://docs.apify.com/platform/integrations/mcp` for current steps rather than guessing.

2. **Composio → Notion connected.** Try listing Composio connections / calling a Composio MCP
   tool. If available, note the Notion account alias for step 4 below and move on. If not,
   walk them through whichever matches how they run Claude, then wait for confirmation:
   - **claude.ai / Claude Desktop:** `claude.ai/settings/connectors` → **+** → find/add
     **Composio** → **Connect** → approve the OAuth popup. Not listed? Add as a custom
     connector with URL `https://connect.composio.dev/mcp`.
   - **Claude Code CLI:** `claude mcp add --transport http composio https://connect.composio.dev/mcp`,
     then `exit` and restart `claude`; confirm with `claude mcp list`.
   - Either way, the first Notion action triggers a one-time Notion OAuth link to approve.
   - If this doesn't match what the user sees, fetch
     `https://composio.dev/toolkits/notion/framework/claude-code` for current steps.

3. **Instagram cookies exist.** Check `~/.config/ig-ingest/cookies.txt`. If it's missing or
   empty, follow `docs/COOKIES.md`: the user exports cookies from a logged-in (ideally burner)
   Instagram account via the Cookie-Editor browser extension, **Export as Netscape**, and saves
   them there. You cannot do this for them — it requires their browser session. Create the
   `~/.config/ig-ingest/` folder for them if it's missing, so they just have to paste. Wait for
   confirmation before continuing.

4. **Notion database + config exist.** Follow `docs/NOTION.md`:
   - Ensure a Notion database exists with these exact properties: `Name` (Title), `Post Link`
     (URL), `Type` (Select), `Author` (Text), `Topics` (Multi-select), `Ingested At` (Date).
   - Ensure Notion is connected in Composio and the database is shared with that connection.
   - Check `${CLAUDE_PLUGIN_ROOT}/config/notion.json` exists with real values (not the
     placeholders in `config/notion.example.json`). If missing, write it:
     ```json
     { "account": "<composio-notion-account-alias>", "database_id": "<notion-database-id>" }
     ```
     You likely already have the account alias from step 2 — if you also know the
     `database_id`, offer to write the file for them instead of making them do it by hand.

Report all four plainly (✓/✗) before moving on. Fix anything that's a ✗ first — don't attempt
Step 1 with a missing piece, it will just fail partway through.

## Step 1 — Get the URL

From the instagram.com URL in the user's message, or ask for one if there isn't one yet.
Accept `/reel/`, `/reels/`, `/p/`, `/tv/` links.

## Step 2 — Read the user's Instagram cookies

Read `~/.config/ig-ingest/cookies.txt` (Netscape format). If it's missing or empty, go back to
Step 0 item 3 — don't call the Actor without cookies, it will fail immediately.

## Step 3 — Run the Apify Actor

Call it via the Apify MCP (dispatch async, then poll — the transcription step can take under
a minute, close to a single sync-wait window, so don't rely on one blocking call):

1. `call-actor` with `actor: "industrial_nightfall/instagram-ingest-scraper"`, `waitSecs: 0`,
   `input: {"instagramUrl": "<url>", "cookies": "<contents of cookies.txt>"}`. This returns a
   `runId` and the run's `datasetId`/`keyValueStoreId` immediately.
2. `get-actor-run` with that `runId`, `waitSecs: 30`. If status isn't terminal yet
   (SUCCEEDED/FAILED/ABORTED/TIMED-OUT), call it again — don't give up after one poll.
3. **On FAILED/ABORTED/TIMED-OUT:** tell the user the ingest failed. The most common cause by
   far is expired/blocked Instagram cookies — point them at `docs/COOKIES.md` to re-export.
   Stop here.
4. **On SUCCEEDED:** `get-dataset-items` with that `datasetId`, `limit: 1` — this is the post's
   metadata: `type`, `author`, `caption`, `page_count`, `has_video`, `transcript` (already a
   plain string, or `null`), `pages` (list of `{index, kind, kvsKey}`), and `frames` (only
   populated in the silent-reel fallback case).

## Step 4 — Materialize any images you need to view

`transcript` is already usable as-is — no file needed for it. Images are different: Claude's
`Read` tool needs a local file path, it can't view a remote URL. For each `pages` entry with
`kind: "image"` (carousel/post case), and for each `frames` entry (silent-reel fallback,
only when `transcript` is empty/null):
1. `get-key-value-store-record` with that run's `keyValueStoreId` and the item's `kvsKey`.
   This returns a **signed URL** good for direct download — no Apify token needed.
2. Download it to a local temp folder (e.g. `~/ig_ingest/<shortcode>/<kvsKey>`) with a plain
   HTTP GET (curl or equivalent).
3. `Read` the local file.

Never do this for `kind: "video"` pages — videos aren't read directly, only their transcript
(Step 3) or fallback frames (this step) matter.

## Step 5 — Analyze (your own vision)

- **Carousel/post:** `Read` each materialized image in `pages` order.
- **Reel:** use the `transcript` string directly — that's the content. Only if `transcript`
  was empty/null (the fallback case), `Read` the materialized `frames` in order to recover
  text off the video.
**Goal: extract the VERBATIM text, not descriptions of the imagery.** Do NOT describe what a slide *looks like* (no "a person on a couch", no "a16z mockup"). Only capture the actual words that appear.

Produce:
- `page_text` — **carousels/posts only.** For each page in order, the **main textual content**
  of that slide — the headline/hook, the body copy, and any substantive lines, examples, or
  code that carry the message. Use the slide's own words (don't paraphrase into a
  description), but keep only what matters.
  **Omit accessory / chrome text**, e.g.:
  - copyright & watermark lines like `© 2026 · BRODY AUTOMATES · INTRO`
  - the repeated author handle / brand name / logo text that appears on every slide
  - section/label tags (`INTRO`, `THE TOOL`), page numbers, swipe hints, UI furniture
  Do **not** add your own annotations or brackets (never write things like `[on-screen in mockup]`). If a slide has no meaningful content, write `(no text)`. Label page 1 as the cover.
- `transcript` — **reels only.** The verbatim spoken words. It comes back as one run-on
  block per video — **break it into readable paragraphs** grouped by natural pauses / topic
  shifts (a few sentences each), the way the speaker would if writing it out. Keep the words
  verbatim (don't paraphrase or trim); only add the paragraph breaks. Never store it as a
  single wall of text.
- `onscreen_text` — **reels only, and only in the frames-fallback case.** When you read frames
  because no transcript was available, capture the verbatim text overlaid on them. When a
  transcript exists you skip frames, so there is no `onscreen_text` — omit it.
- `topics` — 3–6 short tags for the Topics property (e.g. "design", "AI tools", "claude"). This is the only interpretive field.

Do not produce a visual summary or scene description — those are intentionally dropped.

## Step 6 — Write to Notion (single Composio call)

First read the user's Notion target from `${CLAUDE_PLUGIN_ROOT}/config/notion.json` (a JSON file with `account` and `database_id`). Use those two values for the call below.
- If `config/notion.json` doesn't exist or still has placeholder values, go back to Step 0 item 4 — do not guess a database.

Use `COMPOSIO_MULTI_EXECUTE_TOOL` → tool_slug `NOTION_INSERT_ROW_DATABASE` with:
- `account`: the `account` value from `config/notion.json` (your Composio Notion connection alias)
- `database_id`: the `database_id` value from `config/notion.json`

### properties (array of {name, type, value} — names/types are exact & case-sensitive)
| name | type | value |
|---|---|---|
| `Name` | `title` | concise title: `@<author> — <topic>` |
| `Post Link` | `url` | the original URL |
| `Type` | `select` | `reel`, `post`, or `carousel` (from meta) |
| `Author` | `rich_text` | `@<author>` |
| `Topics` | `multi_select` | comma-separated tags, no commas inside a tag |
| `Ingested At` | `date` | current time, ISO 8601 |

### child_blocks (the formatted body — array of {block_property, content})
Build in this order, omitting any section with no data. Each `content` ≤ 2000 chars — **split longer text across multiple `paragraph` blocks**. Keep it faithful to the source text; no image descriptions or summaries.

**For a carousel / post:**
1. `heading_2` "Caption" → `paragraph` with the original caption (verbatim)
2. `heading_2` "Content by Page" → for each page in order:
   - `heading_3` `Page N — Cover` (page 1) or `Page N` (rest)
   - `paragraph`(s) with that slide's **verbatim** text (split if >2000 chars)

**For a reel:**
1. `heading_2` "Caption" → `paragraph` (verbatim)
2. `heading_2` "Transcript" → **multiple** `paragraph` blocks — one per paragraph you grouped above, so the transcript reads cleanly. Do not dump the whole transcript into a single `paragraph`.
3. `heading_2` "On-screen Text" → `paragraph`(s) (verbatim overlays) — **only in the frames-fallback case**; omit this section entirely when a transcript was used.

`child_blocks` items use the simplified shape, e.g.:
`{"block_property": "heading_2", "content": "Transcript"}`,
`{"block_property": "paragraph", "content": "..."}`,
`{"block_property": "bulleted_list_item", "content": "..."}`.
(>100 blocks is fine — the tool appends the overflow automatically.)

## Step 7 — Report

Tell the user: the created **Notion page URL** (from the tool response), the post `type`, and page count. Keep it to a few lines. There's no local folder to mention this time — everything ran on Apify.

## Notes
- Never upload media to Notion. Never store any local path in Notion.
- If the Composio call errors on a property, re-check name/type against the table above (do not invent properties). `Type` is a `select`, not a `status`.
- Topics/Type options auto-create in Notion on first use.
- Signed KVS URLs expire — don't cache/reuse one across sessions; always fetch a fresh one via `get-key-value-store-record` if you need the same image again later.
