# 📥 instagram-ingest-apify

A single **Claude Code skill** that turns any Instagram **reel, post, or carousel** into a
clean, formatted row in your **Notion** database — with **zero local installs**. Scraping and
transcription run on [Apify](https://apify.com)'s cloud; Claude does the reading and writes
to Notion via [Composio](https://composio.dev).

Paste a link and ask, or use the explicit command:

```
/ingest https://www.instagram.com/reel/XXXXXXXXX/
```

> ### ⚠️ Before you install, you'll need three things
> 1. **An [Apify](https://apify.com) account, with its MCP server connected.** This is what
>    actually downloads and transcribes the post — non-negotiable, and every `/ingest` runs on
>    *your* Apify account's compute (Apify's free tier covers casual use — see
>    [`docs/APIFY.md`](docs/APIFY.md)).
> 2. **The [Composio](https://composio.dev) MCP server, connected to Notion.** This is how the
>    plugin writes rows to your Notion database — also non-negotiable.
> 3. **Instagram cookies from a logged-in (ideally burner) account.** Instagram blocks
>    anonymous downloads, so you'll export cookies from your browser once — see
>    [`docs/COOKIES.md`](docs/COOKIES.md). A 2-minute copy/paste, not code.
>
> Unlike most scrapers, there is **nothing else to install** — no ffmpeg, no Python
> environment, no WSL. If you don't want to deal with Apify or Composio, this plugin isn't for
> you yet — both are hard requirements, not optional integrations.

---

## ✨ What it captures

- **Carousels / posts** → the verbatim text of every slide, in order, with the marketing
  chrome (watermarks, page numbers, handles) stripped out.
- **Reels** → a clean, paragraph-formatted transcript (faster-whisper, running on Apify).
  Falls back to reading video frames only when a reel has no usable audio.
- **Every row** → title, post link, type, author, auto-tagged topics, and timestamp.

---

## 🚀 Install

### Option A — Do it yourself

**1. Add the plugin** (in Claude Code):
```
/plugin marketplace add riddhimaaan/instagram-ingest-apify
/plugin install instagram-ingest-apify@instagram-ingest-apify
```

**2. Connect Apify** → [`docs/APIFY.md`](docs/APIFY.md) (free account + MCP connection).

**3. Add your Instagram cookies** → [`docs/COOKIES.md`](docs/COOKIES.md)
(export from a burner account, save to `~/.config/ig-ingest/cookies.txt`).

**4. Point it at your Notion** → [`docs/NOTION.md`](docs/NOTION.md)
(create the database, connect Notion in Composio, fill `config/notion.json`).

**5. Run `/ingest <url>`** in Claude Code — the skill checks all of the above itself the
first time it runs, and tells you what's still missing, if anything. 🎉

### Option B — Let your AI install it for you

Paste this to your AI assistant (Claude Code, Cursor, Codex, …):

> Install the Claude Code plugin at **https://github.com/riddhimaaan/instagram-ingest-apify**
> for me. Follow its `docs/AI-INSTALL.md` runbook step by step.

The [`docs/AI-INSTALL.md`](docs/AI-INSTALL.md) runbook is written for an AI agent to execute:
it connects Apify and Composio, then walks you through the two things only you can do
(cookies + the Notion database).

---

## 🧩 How it works

| Step | Where it runs | What happens |
|---|---|---|
| Download + transcribe | **Apify** (`industrial_nightfall/instagram-ingest-scraper`) | gallery-dl (yt-dlp fallback) downloads the media; faster-whisper transcribes any video — all inside one Apify Actor run |
| Frames *(fallback)* | Apify | ffmpeg samples frames, only when a reel has no transcript |
| Extract | *(the `ig-ingest` skill, local)* | Claude reads the transcript / materialized images and pulls the verbatim text |
| Save | Composio `NOTION_INSERT_ROW_DATABASE` | one formatted row: link in properties, content in the page body |

This is a **single skill**, not a set of commands — the whole flow (first-time setup check,
scrape, analyze, save) is one `skills/ig-ingest/SKILL.md` that Claude follows automatically
whenever you paste an Instagram link.

The Actor's source lives at [riddhimaaan/instagram-ingest-scraper](https://github.com/riddhimaaan/instagram-ingest-scraper)
— you don't need to look at it to use this plugin, it's just the thing running on Apify.

## ⚙️ Configuration

- `config/notion.json` — **your** Composio `account` + Notion `database_id` (you create this;
  it's git-ignored). Template: `config/notion.example.json`.
- `~/.config/ig-ingest/cookies.txt` — your Instagram session, read fresh and sent with every
  `/ingest` call. Never stored on Apify's side under normal use.

## ✅ Requirements

- Claude Code + Apify MCP (your own account) + Composio MCP (with Notion connected)
- Instagram cookies (`~/.config/ig-ingest/cookies.txt`)
- That's it — no OS-specific requirements, no local packages.

## 🔐 Privacy

Media downloads to a short-lived Apify run, never to your machine. Only the post link +
extracted text are written to Notion. Your cookies and `config/notion.json` never leave your
machine except as a per-call input to your own Apify Actor run.

## 📝 License

MIT — see [`LICENSE`](LICENSE).
