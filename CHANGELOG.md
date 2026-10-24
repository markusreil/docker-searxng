# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- TLS via nginx-proxy: `ACME_HOST` + `GEN_SELF_SIGNED_CERT` opt-ins (self-signed driven by `SEARXNG_GEN_SELF_SIGNED_CERT`, default `true`); `BASE_URL` and `homepage.href` now `https://`.

- Initial SearXNG compose stack (HTTP-only, no SSL): compose file, env.example, README, AGENTS.md.
- SearXNG build stage baking tracked `docker/settings.yml` in as seed template; ahmia/torch disabled.
- Tuned seed defaults: reliable keyless engine core (brave, bing, duckduckgo, mojeek, marginalia upweighted), google/startpage/cloudflareai disabled, limiter off.
- Disabled `brave` (scraper 429s on self-hosted IPs).
- Added custom `redtube` video engine (`json_engine`, no API key).
- Added custom `bunkr albums` engine (`xpath` over balbums.st).

### Changed

### Deprecated

### Removed

### Fixed

### Security
