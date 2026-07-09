# ceefax.cloud — Clacton Decides

A parody Ceefax teletext page backing Count Binface in the 2026 Clacton
by-election. Static single page (`index.html`) served by nginx, with
self-hosted [Umami](https://umami.is) analytics (open source, cookie-free —
no consent banner required). Everything deploys as one Docker Compose stack.

## Layout

- `index.html` — the whole site
- `Dockerfile` + `nginx.conf` — nginx serving the page on **port 8080**, with `GET /health`
- `docker-compose.yml` — the full stack: `web` (the site), `umami`, `db` (PostgreSQL)

## Local preview

```sh
DB_PASSWORD=dev APP_SECRET=dev docker compose up --build
# site: via `docker compose port web 8080` (or just open index.html directly)
```

The Umami script in `index.html` 404s harmlessly until analytics is live.

## Deploying on Dokploy

1. In Dokploy: create a **Docker Compose** service, Source = this repo,
   compose path `docker-compose.yml`.
2. **Environment** tab: set `DB_PASSWORD` and `APP_SECRET`
   (see `.env.example`; generate with `openssl rand -base64 32`).
3. **Domains** tab:
   - `ceefax.cloud` → service **web**, container port **8080**, HTTPS on
   - `analytics.ceefax.cloud` → service **umami**, container port **3000**, HTTPS on
4. DNS: **A records** for `ceefax.cloud` and `analytics.ceefax.cloud` → the server IP.
5. Deploy, then log in at `https://analytics.ceefax.cloud`
   (default credentials `admin` / `umami` — **change the password immediately**).

### Analytics wiring

The site is registered in Umami (domain `ceefax.cloud`) and its website ID
is set on the tracking script in `index.html`. If the Umami database is ever
reset, re-add the website and update `data-website-id` to match. Compose only
rebuilds the `web` service on redeploy — Umami and the database are untouched.
