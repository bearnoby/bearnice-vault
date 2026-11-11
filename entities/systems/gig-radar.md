---
pageType: entity
entityType: system
id: entity.systems.gig-radar
title: "System: gig radar"
aliases:
  - gig cards
  - concert radar
  - bearnice-gig-radar-scan
sourceIds:
  - repo:tools/concert-radar
  - repo:jobs/bearnice-gig-radar-scan.yaml
  - repo:agents/assistant/skills/gig-radar/SKILL.md
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: gig radar

Keeps Bear's concert backlog as one living card per band in #gig-radar. A daily scan with no model asks Ticketmaster and Last.fm for shows near where Bear will be on each date and updates the cards; every 10 minutes the gig-cards handler of the [[entities/systems/taps|taps dispatcher]] reads his ❌ and 🎟️ and keeps the cards in step. Watched bands get pings and on-sale reminders. The [[entities/systems/runway|runway]] and the [[entities/systems/morning-brief|morning brief]] read the shows he is going to. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `tools/concert-radar/`: `cards.mjs` (the cards, the daily scan, `--weekend`, `--going`, `--remind`), `cli.js`, `lib/` (`radar.js` scan, `store.js`, `travel.js`, `scoring.js`, `geo.js`, `lastfm.js`, `ticketmaster.js`), and `server.js`, the `concert_radar` MCP server the gateway keeps running (tools `concert_radar__*`: shows, backlog, pins, travel context). Tests in `tools/concert-radar/test/`.
- Repo `jobs/bearnice-gig-radar-scan.yaml`; her skill `gig-radar`; the original plan `docs/plans/gig-radar.md`.
- Her workspace `gigs/`: `concerts.json` (the backlog and every show's status), `cards.json` (one entry per card: `message_id`, shows; plus `passed`; read strictly, never hand-edit), `taste.json` (pinned bands and Last.fm taste), `cards.lock`. It also reads `travel.json` for where he will be.
- Site: `site/data/concerts.json` (mirror of `concerts.json`), page `/concerts/`.
- Env (gateway): `DISCORD_BOT_TOKEN_BEARNICE`, `DISCORD_CHANNEL_GIG_RADAR`, `TICKETMASTER_API_KEY`, `LASTFM_API_KEY`, `LASTFM_USER`; optional `GIG_CARDS_OWNER`, `GIG_CARDS_QUIET=1` (no publish), `GIG_WEEKEND_KM`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-gig-radar-scan` | `30 8 * * *`, Bear's time, timeout 15 min; runs `cards.mjs --scan` | none | #gig-radar |
| `bearnice-taps` (handler `gig-cards`) | every 10 min | none | #gig-radar |
| `gig-onsale-<band>-<date>` | one-shot jobs the cards create with `openclaw cron add` | none | #gig-radar (a message under the card) |

## Discord

- #gig-radar: a card per band lists its dates nearest to where he will be (up to six, then "+N more"), each linked to tickets with the distance that day. New bands get new cards at the bottom.
- ❌ means "not this time": the dates are marked passed, the card is deleted, and that band's run of dates stays hidden; a later tour gets a new card. Deleting a card counts as ❌.
- 🎟️ means "I'm going": on a one-date card the show becomes `ticketed`; on a several-date card the bot asks which, and his reply goes to Bearnice.
- Pinned bands: a new date within reach pings him under the card; each presale and public on-sale gets a "tomorrow" and an "on sale now" reminder. Shows he is going to or passed stay quiet.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `bearnice-gig-radar-scan` has `lastStatus` `ok`, `consecutiveErrors` 0.
- `docker exec openclaw-gateway node /home/node/tools/concert-radar/cards.mjs --dry-run` prints what would change and touches nothing. `--going` and `--weekend` only print JSON.
- `gigs/cards.lock` should not exist for long (a run takes it over after 15 minutes); `concerts.json` `last_run` is the last scan.

## Gotchas

- Any code change under `tools/concert-radar/` (not tests or docs) restarts the gateway on deploy, because the MCP server runs from it.
- Never post, edit or delete a card by hand, and never post a digest to #gig-radar. The weekly digest (`bearnice-gig-radar`) is retired.
- A broken `cards.json` stops the gig-cards handler, which then fails in the taps pass. Back it up next to itself before any hand change.
- Statuses he set are sticky: a scan never downgrades them, and a ❌ never passes a show he is going to.
- `docs/plans/gig-radar.md` still describes the Monday digest and says phase 3 is not started; the cards replaced that.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Reads taps through `tools/lib/taps.mjs` and runs as the `gig-cards` handler of `bearnice-taps` (`c2189f6`, `4fb6d46`); ported to `tools/lib` (`9a58cca`).
- 2026-11-05 Cards built in four stages: one card per band, ❌, "this weekend", watched bands and on-sale reminders, 🎟️ (`f91717a` to `35bc100`).
- 2026-10-24 Gig radar built: Last.fm taste, Ticketmaster, travel-aware backlog (`570dd5c`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/events-radar|System: events radar]]
- [[entities/systems/morning-brief|System: morning brief]]
- [[entities/systems/runway|System: runway]]
- [[entities/systems/taps|System: taps dispatcher]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
