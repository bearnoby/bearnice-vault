---
pageType: entity
entityType: system
id: entity.systems.website
title: "System: website"
aliases:
  - site
  - bearnice.lualderm.com
  - bearnice-site
  - /me/
sourceIds:
  - repo:docs/SITE.md
  - repo:tools/site
  - repo:tools/site-api
  - repo:caddy/Caddyfile
  - repo:docker-compose.yml
updatedAt: 2026-11-13T00:00:00.000Z
---

# System: website

Bearnice's private site at https://bearnice.lualderm.com: trackers that render from JSON the tools keep up to date, write-ups she writes from a template, and one page that saves, "My notes" at `/me/`. It has no job of its own; the tools publish. Caddy serves the `site/` clone directly, so a page is live the moment it is written, and a push to the `bearnice-site` repo is history and backup, not a deploy. Everything is behind one login. Pages come from the [[entities/systems/job-board|job board]] (`/research/job-hunt/`), the [[entities/systems/gig-radar|gig radar]] (`/concerts/`), the [[entities/systems/events-radar|events radar]] (`/events/`), the [[entities/systems/runway|runway]] (`/runway/`) and the [[entities/systems/ops-report|ops report]] (`/ops/`). `/me/` feeds the [[entities/systems/cover-letters|cover letters]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- `site/` is gitignored in the repo; on the server it is a clone of the private repo `bearnice-site`, and its own `README.md` is her guide to the layout, template and rules. Her skill `website`; repo `docs/SITE.md`.
- Repo `tools/site/publish.sh` (commit, rebase, push; run as `sh /home/node/tools/site/publish.sh "<message>"`) and `tools/lib/site.mjs` (`publishSite`: the one way tools publish; a failed push never throws, the page data is already live).
- Repo `caddy/Caddyfile`, `docker-compose.yml`: containers `bearnice-site` (Caddy 2, ports 80 and 443; login `SITE_USER` with a bcrypt hash `SITE_PASSWORD_HASH`; `SITE_DOMAIN`), `bearnice-site-api` (`tools/site-api/server.mjs`, no dependencies, no published port) and `bearnice-browser` (browserless Chromium, CDP on port 3000 inside the Compose network).
- `/me/` writes only `agents/assistant/jobs/sources/me/about-me.json` (the last 200 versions kept in `history/`). The boxes are `SECTIONS` in `server.mjs` and `ABOUT_TITLES` in `tools/letters/opus.mjs`; add a box in both.
- Secrets (names only): deploy key in `secrets/site-deploy/` (mounted read-only at `/home/node/.site-deploy`), `.env` values above. DNS: Cloudflare A record `bearnice`, DNS only.

## Jobs

None of its own. Publishers: `bearnice-taps` (job board and events pages), `bearnice-gig-radar-scan`, `bearnice-runway`, the hourly ops report (copies the `/ops/` page), and Bearnice herself for research pages.

## Discord

None. Bear opens the page from links in Discord posts and the brief.

## How to check it

- `curl -sI https://bearnice.lualderm.com | head -1` answers 401 without a login: Caddy is up.
- `docker compose logs --tail 30 site` for certificate renewals and errors; `docker ps --format '{{.Names}} {{.Status}}'` for the three containers.
- Her view: `docker exec openclaw-gateway curl -s -o /dev/null -w '%{http_code}\n' http://site:8080/` is 200; `docker exec openclaw-gateway openclaw browser --browser-profile browserless doctor` checks her browser.
- The ops report keeps the site certificate and container states (`site/data/ops.json`).

## Gotchas

- Never publish ports 8080 (the site without a login, Compose network only) or 3000 (browser CDP, no auth).
- Let's Encrypt needs the record unproxied and port 80 open (HTTP-01).
- Generated data (`data/concerts.json`, `jobs.json`, `runway.json`, `events.json`, `ops.json`, `keep.json`, `vault.json`) is written by tools; never hand-edit it or the pages that render it.
- A deploy restarts `site-api` on code changes in `tools/site-api/`, and `bearnice-site` when `caddy/Caddyfile` changes (a single-file mount).
- Her browser sees screenshots only as text from a vision model (Gemini); her own model is text-only.
- **Since Phase 4 (11 Nov):** the "scrapbook desk" design. Four fixed sections and one nav from `data/nav.json`; `data/pages.json` registers every page; one page shell and `assets/base.css` plus a stylesheet per page type; the job board is at `/jobs/` (redirects for moved pages are in the repo's `caddy/Caddyfile`, so a move is a deploy); `tools/site/check.mjs` runs inside `publish.sh` and errors block the commit; tool-written data (`concerts`, `events`, `jobs`, `runway`, `ops`, `keep` `.json`) is git-ignored by name. Her rules are the site's `README.md` and `DESIGN.md`; read those before changing the site.

## Change log

- 2026-11-11 "What I know" (`/vault/`, Bearnice section): a read-only view of this vault's people, places, systems, concepts and syntheses (not `sources/` or `reports/`). It renders `data/vault.json`, which `tools/vault-page/build.mjs` writes at the end of every vault sync (`dce6bde`; site `86a43ae`). To change what it shows, edit the note; never the JSON or the page.
- 2026-11-11 Phase 4 site roll-out (`34c3231`): new design, shell, nav and page register; `/jobs/`; site check in publish; generated data out of git; images to WebP.
- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 "My notes" page (`/me/`) feeds the letter writer (`3b24276`); one site publisher in `tools/lib` (`42126ab`).
- 2026-10-27 She can look at her own site: allowlisted browser sidecar and `:8080` (`a9bb107`, `c70b0d8`).
- 2026-10-26 Site built: Caddy at bearnice.lualderm.com serving her `bearnice-site` clone (`d8a20ce`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[syntheses/decision-site-and-shelf|Decision: Bearnice's site and the shelf]]
- [[entities/systems/memory-and-vault|System: memory and vault]]
<!-- openclaw:wiki:related:end -->
