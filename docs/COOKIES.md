# Instagram cookies

Instagram blocks anonymous downloads, so the Apify Actor authenticates using **your browser
cookies** from a logged-in account. The plugin reads this file locally and sends its contents
to the Actor with each `/ingest` call — it's never stored on Apify's side.

> **Use a burner / secondary Instagram account.** You are handing these cookies to a scraper
> running on Apify's infrastructure. Never use an account you can't afford to have
> rate-limited or logged out.

## Export the cookies (2 minutes)

1. Log in to <https://www.instagram.com> in your browser with the account you want to use.
2. Install the **Cookie-Editor** extension
   ([Chrome](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm) /
   [Firefox](https://addons.mozilla.org/firefox/addon/cookie-editor/)).
3. Open instagram.com, click the Cookie-Editor icon.
4. Click **Export → Export as Netscape** (this copies the cookies to your clipboard in the
   `cookies.txt` format the downloader needs — *not* JSON).
5. Save them to the file below:

```
~/.config/ig-ingest/cookies.txt
```

Create the folder first if needed:

```bash
# macOS / Linux
mkdir -p ~/.config/ig-ingest
```
```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force "$HOME\.config\ig-ingest"
```
Then paste the exported cookies into `~/.config/ig-ingest/cookies.txt` (same path on all
three OSes — it's just a folder under your home directory).

## When to refresh

If ingests start failing with a login / checkpoint / rate-limit error, your cookies have
expired. Just repeat the export above to overwrite `cookies.txt`. The plugin will tell you
when this is the likely cause.

## Notes

- The file lives only on your machine and is sent per-call to the Actor — it's never saved on
  Apify's side unless you separately choose to pin it there yourself (advanced/self-hosting
  use only, not needed for normal use of this plugin).
- Format must be **Netscape**, not JSON. If you see `{ "name": ... }` you exported the wrong one.
