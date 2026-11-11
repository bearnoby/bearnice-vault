---
pageType: entity
entityType: system
id: entity.systems.events-radar
title: "System: events radar"
aliases:
  - events
  - what's on
  - bearnice-events-radar
  - bearnice-events-weekend
sourceIds:
  - repo:tools/events
  - repo:jobs/bearnice-events-radar.yaml
  - repo:jobs/bearnice-events-weekend.yaml
  - repo:agents/assistant/skills/events-radar/SKILL.md
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: events radar

Answers "what can we go do in the next few days": one card per event in #events-radar, for the next 7 days near where Bear will be. Bearnice (DeepSeek) searches and writes her finds to `events/candidates.json`; a script with no model then trusts none of it: it opens every link, checks the date and name are on the page, finds the venue, measures the train or tram time from where he is that day, finds the last train back, applies a distance bar and posts the cards. Touring bands belong to the [[entities/systems/gig-radar|gig radar]] and are skipped here. Bear's taps are read by the events handler of the [[entities/systems/taps|taps dispatcher]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `tools/events/scan.mjs`: `--context` (what she needs before searching), `--dry-run`, plain run (verify and post), `--check` (taps), `--set` (status on his word), `--weekend [--post]`, `--brief`, `--site`. Tests in `tools/events/test/`.
- Repo `jobs/bearnice-events-radar.yaml` and `.md` (prompt), `jobs/bearnice-events-weekend.yaml`; her skills `events-radar` (the daily run, the card rules) and `events-now`.
- Her workspace `events/`: `interests.json` (his priorities; he edits it), `candidates.json` (her finds, the script's input), `seen.json` (every carded event and its status), `last-run.json`, `reach.json` (measured places within reach), `geocache.json`, `sources.json` (her calendars per place), `radar.lock`. Also `travel.json`.
- Site: `site/data/events.json` (rewritten from `seen.json` after every run and tap), page `/events/`.
- Env (gateway): `DISCORD_BOT_TOKEN_BEARNICE`, `TICKETMASTER_API_KEY` (optional), `TODO_TOOL`; `EVENTS_QUIET=1` (no publish), `EVENTS_CANDIDATES` (tests). The channel is found by name (`events-radar`) in `config/discord-channels.json` in her workspace, not in `.env`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-events-radar` | `30 9 * * *`, Bear's time, timeout 30 min | `deepseek/deepseek-flash`, thinking high | #events-radar (final reply is `NO_REPLY`) |
| `bearnice-events-weekend` | `30 16 * * 5`, Bear's time; `scan.mjs --weekend --post` | none | #events-radar |
| `bearnice-taps` (handler `events`) | every 10 min; `scan.mjs --check` | none | #events-radar |

`/events_now` (skill `events-now`) runs the daily job now.

## Discord

- #events-radar: a card per event (what, when, where, train time, last train back, price, link, why). Distance sets the bar: local up to 40 min, daytrip up to 90 min (strong matches only), far up to 3 hours (exceptional only). At most 10 cards a run and two of a kind.
- ⭐ interested (shortlist, nothing else), ✅ going (a task in his To Do **Tasks** list and the card says so), ❌ passed (card deleted). Deleting a card counts as ❌; cards of events that are over are removed. Going beats passed beats interested.
- Friday 16:30: one "This weekend" post (going first, then shortlisted, then the rest) that pings him; last week's is deleted.
- Today's and tomorrow's events (up to three lines) reach the [[entities/systems/morning-brief|morning brief]] through `scan.mjs --brief`.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: the two jobs `ok`, `consecutiveErrors` 0 (the daily one takes up to 30 minutes).
- `events/last-run.json` says what the last run posted and what it rejected, and why. A good day is three to eight cards; zero is allowed.
- `docker exec openclaw-gateway node /home/node/tools/events/scan.mjs --dry-run` prints the verdicts for `candidates.json` and writes nothing. `--context` also measures routes and may write `reach.json`, so leave it to her run.
- `events/radar.lock` is held for at most 20 minutes; a check that finds it held reports `skipped`.

## Gotchas

- Eventbrite and regioactive pages turn scripts away, so their links fail the check; she finds the venue's own page instead.
- Stand-up must be in English and the page must say so. Routing (Transitous) covers Germany and the UK only.
- She never posts a card or edits `seen.json`; Bear's `interests.json` changes only on his word.
- The taps handler reads `travel.json` and `seen.json` strictly: a broken file fails the handler, and the pass says so.
- Pinned to DeepSeek so the DM model can change on its own.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Job moved to a file (`562a84d`), pinned to DeepSeek (`046dba1`), taps through `tools/lib/taps.mjs` and the `events` handler of `bearnice-taps` (`c2189f6`).
- 2026-11-05 Stage 3 and 4: Friday roundup, today and tomorrow in the brief, the `/events/` page (`64ff032`, `22102b0`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/morning-brief|System: morning brief]]
- [[entities/systems/taps|System: taps dispatcher]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
