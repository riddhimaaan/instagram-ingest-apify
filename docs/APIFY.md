# Apify setup

This plugin's scraping and transcription (gallery-dl, yt-dlp, ffmpeg, faster-whisper) run on
[Apify](https://apify.com)'s cloud, not your machine — that's what makes this plugin need
**zero local installs**. You need an Apify account and its MCP server connected to Claude.

## 1. Create a free Apify account

[console.apify.com/sign-up](https://console.apify.com/sign-up) — the free tier's included
usage is enough for casual use; heavier use may need a paid plan since each `/ingest` call
runs on your own Apify account's compute.

## 2. Connect the Apify MCP server

- **claude.ai / Claude Desktop:** `claude.ai/settings/connectors` → **+** → find/add **Apify**
  → **Connect** → sign in with the account from step 1.
- **Claude Code CLI:** get an API token from
  [console.apify.com/settings/integrations](https://console.apify.com/settings/integrations),
  then:
  ```bash
  claude mcp add --transport http apify https://mcp.apify.com --headers "Authorization:Bearer <your-apify-token>"
  ```
  `exit` and restart `claude`, then confirm with `claude mcp list`.

If either of these doesn't match what you see (connector UIs change), check
[docs.apify.com/platform/integrations/mcp](https://docs.apify.com/platform/integrations/mcp)
for the current steps.

## 3. That's it

The plugin calls a public Actor — `industrial_nightfall/instagram-ingest-scraper` — for every
`/ingest`. You don't need to build, deploy, or configure anything on the Actor itself; you're
just running someone else's published Actor under your own account, the same way you'd run
any Actor from the Apify Store. Your Instagram cookies (`docs/COOKIES.md`) are sent with each
call from your local machine — they are never stored on Apify's side under this setup.

## Cost note

Every `/ingest` runs on **your** Apify account, billed at Apify's normal compute rates for
the account tier you're on. A single reel/post/carousel ingest is a small, short-lived run —
typically well within the free tier's monthly usage for casual use — but if you ingest a lot
of content, keep an eye on [console.apify.com/billing](https://console.apify.com/billing).
