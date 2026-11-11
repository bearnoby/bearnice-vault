---
pageType: entity
entityType: system
id: entity.systems.runway
title: "System: runway"
aliases:
  - stay clocks
  - travel runway
  - bearnice-runway
sourceIds:
  - repo:tools/runway
  - repo:jobs/bearnice-runway.yaml
  - repo:agents/assistant/skills/runway/SKILL.md
updatedAt: 2026-11-11T17:12:48.000Z
---

# System: runway

Works out where Bear will be week by week and what that does to his stay clocks (Schengen 90 of 180 days, the US six-month and one-year marks), from his trips. A build with no model runs every 30 minutes, writes `data/runway.json` and publishes the `/runway/` page only when it changed. The [[entities/systems/job-board|job board]] uses its clocks so the two always agree, the [[entities/systems/morning-brief|morning brief]] reads its trips and decisions, and the [[entities/systems/heartbeat|heartbeat]] watches for gaps. It also marks the shows he is going to from the [[entities/systems/gig-radar|gig radar]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `tools/runway/build.mjs` (the whole build; `--dry-run` prints the result and writes nothing). Tests in `tools/runway/test/`.
- Repo `jobs/bearnice-runway.yaml`; her skill `runway` (decisions, drafting options) and the Travel and runway parts of `agents/assistant/AGENTS.md`.
- It reads, in her workspace: `travel.json` (the plan; every trip has `from`, `until`, a `place` ending in its country, and its own To Do list id), `travel-log.json` (where he was each day; the build keeps it), `decisions.json` (open questions with options; hers), `jobs/profile.json` (`stay_clocks` for days before the log), `gigs/concerts.json`, `jobs/seen.json` (interview dates). It also reads each trip's To Do list through `tools/microsoft-todo` to count bookings.
- Site: `site/data/runway.json` (output, never hand-edited), page `/runway/`.
- Env: `RUNWAY_WORKSPACE`, `RUNWAY_SITE`, `RUNWAY_TODO_TOOL` (defaults inside the gateway).

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-runway` | every 30 min, command `node /home/node/tools/runway/build.mjs` | none | nowhere (writes the page data) |

## Discord

None of its own. Decisions go to Bear as a DM from Bearnice (draft options, "go with B"), and the page is linked from the brief.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `bearnice-runway` `lastStatus` `ok`, `consecutiveErrors` 0, a run within the last 30 minutes.
- `docker exec openclaw-gateway node /home/node/tools/runway/build.mjs --dry-run` prints the build and writes nothing.
- `site/data/runway.json` has `updated` within the hour; `clocks.now`, `clocks.moves`, `clocks.reentry`, `trips`, `gaps` and `decisions` are what other systems read.

## Gotchas

- Nothing is counted from memory: the clocks come from trips with `from`, `until` and a country-ending `place`. A trip missing those makes wrong clocks, not an error.
- Decision options (`legs`) are run through the same clocks; the build adds the `effects`. Her write-up must not restate the numbers.
- A leg with no `until` is open-ended; after the last leg's end the build assumes he is home.
- `travel-log.json` is written by the build. Fix a wrong past day there by hand and the clocks follow.
- Month-end dates (the six-month and one-year marks) used to overflow; `addMonths` clamps to the last day of the month.
- The `/runway/` page and `runway.json` are never edited by hand. Since Phase 4 the page sits under Trackers in the shared nav (`data/nav.json`) and `runway.json` is git-ignored in the site repo; the home page's "Right now" reads it.

## Change log

- 2026-11-11 Phase 4 site roll-out (`34c3231`): shared nav and shell; the home page reads `runway.json`; the file is no longer committed.
- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Month-end overflow fixed in `addMonths` and the stay clocks (`e7756f3`); job moved to a file (`a03bf3c`); ported to `tools/lib` (`5779045`).
- 2026-11-05 Shows he is going to appear on the runway (`35bc100`).
- 2026-11-03 Decisions laid out as options side by side (`fc6c84f`); one To Do list per trip with a readiness line (`e815e98`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/gig-radar|System: gig radar]]
- [[entities/systems/heartbeat|System: heartbeat]]
- [[entities/systems/morning-brief|System: morning brief]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
