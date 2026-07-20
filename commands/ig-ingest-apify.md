---
description: Download an Instagram reel/post/carousel via Apify, analyze it, and save the extracted content to Notion.
argument-hint: <instagram-url>
---

Ingest this Instagram URL into Notion: $ARGUMENTS

Follow the `ig-ingest-apify` skill exactly, starting with its Step 0 first-time setup check (Apify,
Composio→Notion, cookies, Notion config) if it hasn't already been confirmed this session.
Then scrape via the Apify Actor (through the Apify MCP), analyze the media and transcript
yourself, and write one formatted row to the "Social Media Posts Link" Notion database via
Composio. Do not upload media to Notion. Report the resulting Notion page URL when done.
