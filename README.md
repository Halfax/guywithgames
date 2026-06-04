# guywithgames

Source for **[www.guywithgames.com](https://www.guywithgames.com)** — a single hand-written
static page that presents a self-hosted AI homelab and the projects running on it.

No framework and no build step: the whole site is one `index.html` (~680 lines: HTML with
inline CSS and a little JS), a couple of images, and the standard error pages. It's served
as static files by Apache on a Linode VPS.

## What's here

| File | Purpose |
|---|---|
| `index.html` | the entire page — copy, layout, inline styles + a little JS |
| `404.html`, `50x.html` | Apache error pages |
| `avatar.webp`, `halfax-ai-banner.webp` | page images |
| `DEPLOY.md` | how the site is deployed (Apache on Rocky Linux, perms, SELinux context) |

## What it covers

The page links out to the live, interactive pieces — the public AI chat and the **Augur**
news-corroboration dashboard — and describes the homelab behind them: a small fleet of
hosts on a private mesh, a self-hosted multi-model AI server, a shared cross-agent memory
layer, the **Story Universe** narrative engine, and the MCP tooling used to operate the
fleet in plain English.

## Deploy

Edit here → commit → push → deploy to the VPS. The Rocky-Linux gotcha most people forget —
fixing the **SELinux context** after upload (Apache can 403 a file even with the right
owner and mode) — is documented in [`DEPLOY.md`](DEPLOY.md).
