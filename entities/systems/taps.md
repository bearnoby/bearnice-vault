---
pageType: entity
entityType: system
id: entity.systems.taps
title: "System: taps dispatcher"
aliases:
  - taps
  - bearnice-taps
  - tap handlers
sourceIds:
  - repo:tools/taps
  - repo:tools/lib/taps.mjs
  - repo:jobs/bearnice-taps.yaml
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: taps dispatcher

One 10-minute pass, no model, over Bear's taps (reactions) on every card the bot posts, with one handler per card type. It replaced four separate pollers (`bearnice-gig-cards`, `bearnice-events-taps`, `bearnice-job-board-check`, `bearnice-letters`). The pass itself holds no logic: each handler is the exported `tick({ cache })` of a tool, and the tool still runs on its own for debugging. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `tools/taps/taps.mjs` (the dispatcher; `HANDLERS` lists them in pass order) and `tools/taps/README.md` (handler table, limits, locks, exit status, the behaviour kept from the four pollers). Tests in `tools/taps/test/`, offline with fake handlers.
- Repo `tools/lib/taps.mjs`: the one way to read taps (`recentMessages`, `ownerTapped`, `botReacted`, `reactionCounts`, `cachedMessage`). A tap counts only when the count is above the bot's own reaction and Bear is among the reactors.
- Repo `jobs/bearnice-taps.yaml`: `every: 10m`, command `node /home/node/tools/taps/taps.mjs`, channels #gig-radar, #events-radar, #job-hunt, #job-board.
- Her workspace `taps/` (or `TAPS_DIR`): `state.json` (failure streaks per handler) and `<handler>.lock`.

## Jobs

| |Job | Schedule | Model | Posts to |
|---|---|---|---|
| |`bearnice-taps` | every 10 min | none | the four channels above (through its handlers) |

## The handlers, in order

| Handler | Card type | Tool | Limit | Lock taken over after |
|---|---|---|---|---|
| `gig-cards` | gig radar | `concert-radar/cards.mjs` | 8 min | 60 min |
| `events` | events radar | `events/scan.mjs --check` | 8 min | 60 min |
| `job-board` | job board | `job-board/check.mjs` | 8 min | 60 min |
| `letters` | cover letters | `letters/watch.mjs` | none | 180 min |

The systems behind them: [[entities/systems/gig-radar|Gig radar]], [[entities/systems/events-radar|Events radar]], [[entities/systems/job-board|Job board]], [[entities/systems/cover-letters|Cover letters]]. The cheap, independent ones come first. The job board runs before the letters so the letters read `seen.json` after the board check wrote it and reuse the board posts' starter messages it fetched; the letters run last because an Opus draft can take minutes.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `bearnice-taps` `lastStatus` `ok`, `consecutiveErrors` 0.
- `taps/state.json` in her workspace: the failure streak per handler (a pass that succeeds for a handler resets it).
- One JSON line per pass on stdout: `failed` (names, only when something failed), each handler's report, `ms` per handler. Do not start a pass by hand to see it: the handlers write to Discord. To look at one tool without writing, use its own dry run (`cards.mjs --dry-run`, `scan.mjs --dry-run`, `check.mjs --dry-run`, `watch.mjs --dry`).

## Gotchas

- **"Job bearnice-taps failed 3 times running"** means three passes in a row had at least one failing or timed-out handler. The alert text is the job's last error, which names the handler and its own streak (`taps: failed: letters (3 passes running): ...`). One alert covers all four: a second handler failing while it is up raises nothing new, but `/ops/` shows the current error.
- Handlers fail apart: one that throws is recorded and the next runs. Exit status is 1 when any failed; a handler skipped for a lock is not a failure.
- If a pass is still running at the next tick (a long letter draft), the second pass skips the handler still running and runs the others.
- `taps.mjs` takes no options on purpose: tools read `--dry-run` from `process.argv`, so a flag would reach some handlers and not others.
- Adding a handler: export `tick({ cache })` (return the report, throw on failure, no `process.exit`, no stdout), add a row to `HANDLERS` with a limit and a lock age, add its channel to the job file, and compare it side by side with the old poller against a fake Discord before deploying.
- Code under `tools/taps/` restarts nothing on deploy (a fresh process each run), but a change to `tools/concert-radar/` does restart the gateway.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Dispatcher built: one pass over the four tap handlers (`4fb6d46`); shared tap reading in `tools/lib/taps.mjs` (`c2189f6`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/events-radar|System: events radar]]
- [[entities/systems/gig-radar|System: gig radar]]
- [[entities/systems/job-feed|System: job feed]]
- [[entities/systems/ops-report|System: ops report]]

### Related Pages

- [[entities/systems/cover-letters|System: cover letters]]
- [[entities/systems/job-board|System: job board]]
<!-- openclaw:wiki:related:end -->
