# Deploying guywithgames.com

This repo is the **source of truth** for `www.guywithgames.com`. The live site is a
static page served by **Apache** on the **Guy Linode VPS** (Rocky Linux), from:

```
/var/www/sub-domains/guywithgames/html/
```

Edit files here → commit/push → deploy to the Linode → fix ownership, perms, and
(on Rocky Linux) the **SELinux context**. The last step is the one people forget:
Rocky's Apache (`httpd`) runs under SELinux, so a file with the right owner and
mode can *still* 403 if its SELinux type is wrong after an upload.

| Fact | Value |
|---|---|
| Host | Guy Linode VPS — `50.116.53.154` (Netbird `100.87.206.106`), Rocky Linux 9 |
| Web root | `/var/www/sub-domains/guywithgames/html/` |
| Served by | Apache `httpd` (ports 80/443, Let's Encrypt TLS via certbot) |
| SSH key | `~/.ssh/linode-priv` (**on Betelgeuse**; not on Cygnus) — `root@50.116.53.154` |
| Owner / mode | `apache:apache`, files `644`, dirs `755` |
| SELinux type | `httpd_sys_content_t` (default for `/var/www`; `restorecon` restores it) |

---

## Path A — homelab MCP (preferred, from Betelgeuse)

The homelab MCP server keeps a local mirror and handles upload + perms in one call:

```
site_deploy("guywithgames")                 # all changed files
site_deploy("guywithgames", files="index.html")
```

It SFTPs as `root`, then runs `chown apache:apache` + `chmod 644`. (If the MCP
predates the SELinux note, still run `restorecon` — see Path B step 3.)

## Path B — manual (from a host that has `linode-priv`, i.e. Betelgeuse)

```bash
# 1. get this repo's files onto the box (from the repo dir)
scp -i ~/.ssh/linode-priv index.html 404.html 50x.html avatar.webp halfax-ai-banner.webp \
    root@50.116.53.154:/var/www/sub-domains/guywithgames/html/

# 2. ownership + perms
ssh -i ~/.ssh/linode-priv root@50.116.53.154 '
  cd /var/www/sub-domains/guywithgames/html &&
  chown apache:apache *.html *.webp &&
  chmod 644 *.html *.webp'

# 3. SELinux context (Rocky Linux — REQUIRED or Apache may 403/permission-denied)
ssh -i ~/.ssh/linode-priv root@50.116.53.154 '
  restorecon -Rv /var/www/sub-domains/guywithgames/html'
```

No Apache reload is needed for static-file changes. If you ever add a *new*
directory under the web root, `restorecon -R` it too.

## Verify

```bash
curl -sI https://www.guywithgames.com/ | head -1            # HTTP/2 200
curl -s  https://www.guywithgames.com/ | grep -o '<title>[^<]*'   # new title
```

## Deploying from Cygnus

Cygnus does **not** have `linode-priv`. Either deploy from Betelgeuse (Path A/B) or
copy the key into the Tier-A pattern first. The git repo (`gitlab:halfax/guywithgames`)
is reachable from anywhere; only the final push to the Linode needs the key.
