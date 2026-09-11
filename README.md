# GitHub Pages wrapper — Shift & Handover

This folder is a **thin iframe wrapper**, meant to be published as its own GitHub Pages site so the
app has a short, memorable public URL — the same pattern as the team's other project, `bearsec`
(`https://tuh-apps.github.io/bearsec/`). It contains **no application logic, no data, and no
security controls** of its own; all of that lives in the actual Google Apps Script Web App. This
page's only job is to load that Web App inside a full-page `<iframe>` behind a short URL, show a
branded loading state while it does, and fall back to a direct link if loading takes unusually long
or the frame is blocked.

## Files

| File | Purpose |
|---|---|
| `index.html` | The wrapper page itself. |
| `favicon.png` | 512×512 PNG — **this exact filename is what `doGet()` in `Code.gs` requests** at `<public_web_app_url>/favicon.png` to set the deployed Web App's own browser-tab icon. Do not rename it. |
| `favicon.ico`, `favicon-16.png`, `favicon-32.png` | Small raster icons for this wrapper page's own browser tab. |
| `favicon.svg` | Source vector icon (also embedded inline, base64, as the favicon inside `Index.html` itself — kept here too as the master source if the icon ever needs to be re-exported). |
| `apple-touch-icon.png` | 180×180 icon used when a visitor adds this page to an iOS/iPadOS home screen. |

The icon is the same teal rounded-square calendar glyph already used for the in-app sidebar brand
mark, so the browser tab, the home-screen icon, and the app's own header all match.

## Setup steps (one-time)

1. **Deploy the Apps Script project as a Web App** (if not already) and copy its `.../exec` URL.
2. Open `index.html` in this folder and replace the placeholder:
   ```js
   var APP_URL = 'REPLACE_WITH_DEPLOYED_WEB_APP_EXEC_URL';
   ```
   with the real `/exec` URL.
3. Create (or reuse) a GitHub repository under the `tuh-apps` org for this project, push this
   folder's contents to it, and enable **GitHub Pages** for that repo (Settings → Pages → Deploy
   from a branch).
4. Once the Pages URL is live (e.g. `https://tuh-apps.github.io/<project-name>/`), open the Shift &
   Handover admin settings and set the `public_web_app_url` SystemSettings value to that URL
   (no trailing slash needed — the app trims it). This one setting is what:
   - makes `doGet()` set the deployed Web App's own browser-tab favicon, and
   - appends a "เข้าใช้งานระบบได้ที่ …" / "เข้าไปดู/ตอบคำขอได้ที่ …" link to OTP and
     Exchange/Transfer notification emails (see `Utils.emailLinkLine()` in `Code.gs`).

   Until this setting is filled in, both features stay fully inert — no broken links, no missing
   favicon errors — so it is safe to deploy the app before the GitHub Pages URL exists.

## Why an iframe wrapper is safe here

`doGet()` in `Code.gs` already sets `.setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)`,
so the deployed Web App is already configured to permit exactly this kind of framing — no change
was needed there for this wrapper to work.

The link this wrapper's URL gets used for in notification emails is always a plain link to the
**ordinary login page** — never a token-bearing "magic auto-login" link. Clicking it still requires
going through the normal OTP flow, so it cannot be used to bypass authentication even if an email
were forwarded, spoofed, or read by someone other than its intended recipient.
