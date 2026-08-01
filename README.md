# nextelement.flobiz.in

Landing page for the **Flobiz × Next Element** partnership — Anaya AI, the voice + WhatsApp
assistant that replies to every ad lead in 30 seconds. Single self-contained static page.

Live target: **https://nextelement.flobiz.in**

---

## What this is (and isn't)

- **One static HTML file** (`index.html`). No backend, no build step, no database.
- Everything dynamic happens **in the visitor's browser**:
  - **Voice demo** → the browser connects directly to `wss://voice.cashflohero.ai` via the
    Flo/cashflohero embed SDK (loaded from `https://voice.cashflohero.ai/sdk/voice-agent-sdk.js`).
    Nothing routes through our own server.
  - **WhatsApp CTAs** → plain `https://wa.me/971553129985` links.
  - **Fonts** → Fontshare CDN (`api.fontshare.com` / `cdn.fontshare.com`).

Because it's static, you deploy it like any other static page. **The only hard requirement is
HTTPS** (see below).

---

## ⚠️ Must be served over HTTPS (or `http://localhost`)

The voice demo uses the microphone and a Web Audio **AudioWorklet**, which browsers only allow in a
**secure context**. Consequences:

- ✅ `https://nextelement.flobiz.in` — works.
- ✅ `http://localhost:PORT` / `http://127.0.0.1:PORT` — works (localhost is treated as secure).
- ❌ **Opening `index.html` directly as a `file://`** — the worklet fails with
  *"Unable to load a worklet's module."* This is the #1 gotcha. Always serve it, never double-click.

---

## Local preview

```bash
# from this folder
python3 -m http.server 8791 --bind 127.0.0.1
# then open http://127.0.0.1:8791/  (NOT the file:// path)
```

or

```bash
npx serve .
```

---

## Deploy

### DNS (one-time)

Point the subdomain at your static host with a **CNAME**:

```
nextelement   CNAME   <your-host-target>     # e.g. <site>.netlify.app / <proj>.pages.dev
```

Your host then issues the TLS cert for `nextelement.flobiz.in` automatically.

### Host options (pick one — all give free HTTPS)

**Cloudflare Pages** or **Netlify** (recommended — both read the included `_headers` file):
1. Connect this repo (or drag-drop the folder).
2. Build command: *none*. Publish directory: `/` (root).
3. Add custom domain `nextelement.flobiz.in`.

**Vercel:**
- Import repo, framework preset **Other**, no build command, output dir `.`.
- Add the domain in Project → Settings → Domains.

**GitHub Pages:**
- Settings → Pages → deploy from branch (root). The included `CNAME` file sets the custom domain.
- Note: GitHub Pages can't set custom headers, so the optional CSP below won't apply there.

---

## Optional: Content-Security-Policy

The page works with **no CSP** (default). If you want to lock it down, the CSP **must** allow the
voice SDK, its blob worklet, the websocket, and the fonts — otherwise the voice demo breaks. Tested
directives (already provided, commented, in `_headers`):

```
Content-Security-Policy:
  default-src 'self';
  script-src  'self' 'unsafe-inline' https://voice.cashflohero.ai blob:;
  worker-src  'self' blob:;
  style-src   'self' 'unsafe-inline' https://api.fontshare.com;
  font-src    'self' https://cdn.fontshare.com;
  connect-src 'self' https://voice.cashflohero.ai wss://voice.cashflohero.ai https://api.fontshare.com;
  img-src     'self' data:;
  base-uri    'self'
```

`'unsafe-inline'` is required because the page's CSS and JS are inline. If you remove it, you must
externalise `<style>`/`<script>` first. **Do not ship a CSP that omits `blob:` or
`voice.cashflohero.ai`** — that reproduces the worklet failure.

---

## Editing the page

Everything lives in `index.html` (inline CSS + JS). Common edits:

| Change | Where |
|---|---|
| WhatsApp number | search `971553129985` (used in `wa.me` links + display text) |
| Voice agent | search `VoiceAgent.init` — `agentId` / `serverUrl` |
| Copy / sections | the HTML is commented per section (`<!-- ===== HERO ===== -->`, etc.) |

### Voice agent config

```js
VoiceAgent.init({
  serverUrl: 'wss://voice.cashflohero.ai',
  agentId:   'meraki-aesthetics-clinic-clinica-agent-anaya',  // Meraki Aesthetics Clinic
  widget:    false,   // in-card UI; our "Tap to talk" button drives start()/stop()
  ...
});
```

- The agent is on **prod** cashflohero (`voice.cashflohero.ai`). Staging (`voice.niobooks.in`)
  requires credentials and won't resolve this agent.
- Sanity-check an agent id before shipping:
  ```bash
  curl -s https://voice.cashflohero.ai/api/embed/config/meraki-aesthetics-clinic-clinica-agent-anaya
  # 200 + JSON = valid;  404 "Agent not found" = wrong id / wrong host
  ```
- To swap agents, change `agentId` to the id shown on the cashflohero Agents page.

---

## Optional hardening: self-host fonts

Today the page pulls **General Sans / Satoshi / JetBrains Mono** from Fontshare at runtime. To drop
the third-party dependency (and the `font-src`/`style-src` CSP entries), download the woff2 files,
add `@font-face` rules, and remove the `<link href="https://api.fontshare.com...">`. Not required —
purely removes a CDN from the critical path.

---

_Powered by Flobiz. Delivered by Next Element. Anaya AI._
