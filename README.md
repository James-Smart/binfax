# ceefax.info — Clacton Decides

A parody Ceefax teletext page backing Count Binface in the 2026 Clacton
by-election. Static single page (`index.html`), served by nginx, with
self-hosted [Umami](https://umami.is) analytics (open source, cookie-free —
no consent banner required).

## Layout

- `index.html` — the whole site
- `Dockerfile` + `nginx.conf` — nginx serving the page on **port 8080**, with `GET /health`
- `analytics/docker-compose.yml` — Umami + PostgreSQL stack for `analytics.ceefax.info`

## Local preview

```sh
docker build -t ceefax-info . && docker run --rm -p 8080:8080 ceefax-info
# then open http://localhost:8080
```

(Or just open `index.html` in a browser — the Umami script 404s harmlessly
until analytics is deployed.)

## Deploying on Dokploy

### 1. The site (Application)

1. Push this folder to a Git repo.
2. In Dokploy: create an **Application**, Source = the repo, Build Type = **Dockerfile**.
3. Domains tab: add `ceefax.info` (and `www.ceefax.info` if wanted), **Container Port = 8080**, HTTPS on.
4. DNS: **A record** for `ceefax.info` → your server's IP.
5. Deploy.

### 2. Analytics (Docker Compose)

1. In Dokploy: create a **Docker Compose** service pointing at `analytics/docker-compose.yml`.
2. Environment tab: set `DB_PASSWORD` and `APP_SECRET` (see `analytics/.env.example`;
   generate with `openssl rand -base64 32`).
3. Domains tab: add `analytics.ceefax.info` → service `umami`, **Container Port = 3000**.
4. DNS: **A record** for `analytics.ceefax.info` → your server's IP.
5. Deploy, then log in at `https://analytics.ceefax.info`
   (default credentials `admin` / `umami` — **change the password immediately**).

### 3. Wire them together

1. In Umami: **Settings → Websites → Add website**, domain `ceefax.info`.
2. Copy the generated **Website ID**.
3. In `index.html`, replace `REPLACE-WITH-UMAMI-WEBSITE-ID` with it.
4. Redeploy the site. Views, visitors, referrers, countries and devices
   appear on the Umami dashboard.
