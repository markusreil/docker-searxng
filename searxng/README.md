# SearXNG image

Built `FROM searxng/searxng` (upstream image *is* the service). The only build
step replaces the upstream `settings.template.yml` with our tracked
`docker/settings.yml`, so first-run seeding produces our defaults.

## Why no custom entrypoint

The upstream entrypoint already seeds `/etc/searxng/settings.yml` from the
template on first start only (`if [ ! -f ... ]`) and manages `searxng:searxng`
ownership itself. A custom entrypoint would add nothing, so per spec rules
8-9 there is none — and therefore no `PUID`/`PGID` handling either.

Reseed procedure: `docker compose rm -sf searxng`, remove the
`searxng-config` volume, then `up -d` (destroys customized settings).

## Environment variables

| Var | Purpose |
| --- | --- |
| `SEARXNG_VERSION` | Upstream image tag (build arg + image tag). |
| `SEARXNG_SECRET` | Runtime session-signing key override (compose env). |
| `BASE_URL` | Public base URL, derived from `BASE_DOMAIN` (compose env). |
| `BUILD_DATE` | UTC build timestamp (build arg, optional, fails open to `unknown`). |

Build with `BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) docker compose build`.

## Tracked settings (`docker/settings.yml`)

- `use_default_settings: true` — inherits all upstream defaults.
- `secret_key: "ultrasecretkey"` — placeholder the upstream entrypoint
  replaces with a random value on first seed (same as stock behavior).
- Disabled: `ahmia`, `torch` (Tor-only, fail without a Tor proxy — log
  noise observed 2026-10-19), `google` + `startpage` (aggressive bot
  blocking of datacenter/self-hosted IPs), `cloudflareai` (needs API key),
  `yahoo news` / `github code` / `openclipart` (inactive by default
  upstream).
- Upweighted (`weight: 2`) + longer timeouts: `bing`,
  `duckduckgo`, `mojeek`, `marginalia` — the reliable keyless core.
- `brave` disabled — scraped web UI rate-limits self-hosted IPs hard
  (persistent 429s, 3-minute suspensions observed 2026-10-19). Re-enable
  with a `braveapi` API key instead if you want Brave coverage back.
- The `limiter.toml` warning at startup is unconditional and harmless:
  upstream defaults already set `server.limiter: false`; single-user
  localhost instances need no botdetection rate limiter.
- Note: `ahmia`/`torch` still log an import error at startup even when
  disabled — SearXNG imports every engine module before filtering. The
  errors are noise; the engines are never queried.
- Nearly all other default engines work without keys; key-gated engines
  (google_cse, braveapi, kagi, deepl, youtube_api, wolframalpha, …) ship
  commented out / with empty keys upstream, so nothing to do for them.
- Custom `redtube` engine (`json_engine` over the RedTube search API,
  `!redt`, videos category) — no API key needed, verified live
  2026-10-19 (20 results). SafeSearch is `0` in this config so it isn't
  filtered.
- Custom `bunkr albums` engine (`xpath` over balbums.st album search,
  `!ba`, images+files categories, title/link/file-count/thumbnail all
  parsing, verified live 2026-10-19). Pagination via `page` param.
