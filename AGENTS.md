# AGENTS.md

Ongoing rules for working in this repo.

- All config lives in `.env` (never tracked); `env.example` is the tracked source of truth. Keep both in the same order/shape: globals on top, then one `# --- <service> ---` section per service, every var commented. Copy with `cp env.example .env`.
- Every required var uses `${VAR:?...}` fail-fast interpolation; optional vars use `${VAR:-default}` and are documented as optional. Never add silent defaults for required config.
- No `container_name:`; rely on default naming. `restart: unless-stopped`.
- No `ports:`; all HTTP goes through the shared external proxy network (`NGINX_PROXY_NETWORK`, default `web-proxy`, `external: true`). Proxy cluster must be up first.
- This stack is HTTP-only (no SSL): only `VIRTUAL_HOST` (+ `VIRTUAL_PORT`) is declared, no `ACME_HOST` / `GEN_SELF_SIGNED_CERT`. To add HTTPS, declare both TLS opt-ins (`GEN_SELF_SIGNED_CERT` from `.env`, default `false`).
- Hostnames come from `x-hosts` anchors derived from `BASE_DOMAIN`; never copy-paste literals.
- Named volume for state (`searxng-config`); no data bind mounts. No custom entrypoint: upstream image *is* the service, so no `PUID`/`PGID` handling.
- Homepage labels (`group`, `name`, `icon`, `href`, `description`) on proxied services.
- Changelog: keep `CHANGELOG.md` (Keep a Changelog) updated under `## [Unreleased]` as changes are made; never create release sections unprompted.
- Verify: `docker compose config`, `docker compose build`, `docker compose up -d`, `docker compose ps`. See `README.md` for detail.
