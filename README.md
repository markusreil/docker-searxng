# SearXNG

Privacy-respecting metasearch engine, deployed as a downstream service behind the shared nginx reverse proxy, with TLS via the proxy's cert generator.

## Architecture

- `searxng`: upstream `searxng/searxng` image, exposes 8080, routed via `VIRTUAL_HOST=searxng.<BASE_DOMAIN>`.
- Named volume `searxng-config` holds `/etc/searxng` (settings persist across restarts/rebuilds).
- `searxng/` build: `FROM searxng/searxng`, baking the tracked `docker/settings.yml` in as the seed template (see `searxng/README.md`). No custom entrypoint: the upstream one already seeds first-run-only (per spec rules 8-9).
- No `ports:` mappings; all traffic goes through the external `web-proxy` network.

## Quickstart

```sh
cp env.example .env
# Edit .env: set SEARXNG_SECRET (openssl rand -hex 32); keep BASE_DOMAIN=localhost for local use
docker compose config
docker compose up -d
docker compose ps
```

Open `https://searxng.localhost/` (ensure the proxy cluster is up first and `*.localhost` resolves to it). The proxy issues a real certificate via `ACME_HOST`.

## Usage

Category tabs under the search bar (General, Images, Videos, News, …) filter
by category; the same works from the query box with `!` bangs:

- `!images red panda` — any category works as a bang and they stack
  (`!images !videos …`). Multi-word categories use `_`: `!social_media`.
  Full category list (taken from the live Preferences page, which renders
  every category with at least one loaded engine): tabs `!general`
  `!images` `!it` `!music` `!videos` `!files` `!social_media` `!news`
  `!map` `!science`, plus `!web` `!blogs` `!books` `!currency`
  `!translate` `!wikimedia` `!icons` `!lyrics` `!radio` `!apps`
  `!packages` `!q&a` (stackoverflow, askubuntu, superuser, … — note: `&`
  must be URL-encoded in GET links, typing it in the search box is fine)
  `!repos` `!software_wikis` (arch wiki, gentoo, nixos wiki, …)
  `!scientific_publications` `!dictionaries` `!movies` `!shopping`
  `!stock_images` `!weather` `!other`. (`!onions` needs Tor engines,
  disabled here, so it has no engines to query.)
- Engine bangs narrow to one engine: `!duckduckgo`, `!mojeek`, `!bing`.
- Typing `!` triggers autocomplete with every available bang.
- Quirk worth knowing: for non-tab categories (`!icons`, `!lyrics`,
  `!packages`, …) the bang queries every loaded engine in that category
  even if it shows as disabled in Preferences — the `disabled` flag is
  only enforced for tab categories. Harmless in practice: those engines
  are fast and keyless. To truly remove one, set `inactive: true` for it
  in `searxng/docker/settings.yml` (unloaded engines can't be banged).
- Preferences → Search can switch the output format to JSON (enabled in
  this config via `search.formats`) for scripting use.

## Operational notes

- Start order: the proxy cluster must be up first (it owns the `web-proxy` network).
- `SEARXNG_SECRET` must be a long random value; changing it invalidates sessions.
- `BASE_URL` derives from `BASE_DOMAIN`; changing `BASE_DOMAIN` moves the whole stack.
- TLS: both proxy opt-ins (`ACME_HOST`, `GEN_SELF_SIGNED_CERT`) are declared; `BASE_URL` and `homepage.href` use `https://`. The self-signed path is driven by `SEARXNG_GEN_SELF_SIGNED_CERT` (default `true`).
- Tracked defaults seed the `searxng-config` volume on first start only; afterwards the volume is authoritative. To change defaults, edit `searxng/docker/settings.yml`, rebuild, then reseed (remove service + volume, `up -d`). To tweak live settings only, edit `settings.yml` inside the volume and restart.
