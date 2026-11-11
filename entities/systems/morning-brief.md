---
pageType: entity
entityType: system
id: entity.systems.morning-brief
title: "System: morning brief"
aliases:
  - daily brief
  - daily morning briefing
  - bearnice-morning-brief
sourceIds:
  - repo:jobs/bearnice-morning-brief.yaml
  - repo:jobs/bearnice-morning-brief.md
  - repo:jobs/README.md
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: morning brief

A model run at 10:00 where Bear is: Bearnice reads his day from the systems around her and writes a short brief (about ten lines at most, readable in under a minute). OpenClaw posts her final reply to #daily-brief. It is a pure reader: it feeds nothing. It draws on the [[entities/systems/runway|runway]], the [[entities/systems/job-board|job board]], the [[entities/systems/gig-radar|gig radar]] and the [[entities/systems/events-radar|events radar]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `jobs/bearnice-morning-brief.yaml` (schedule, model, `announce: daily-brief`) and `jobs/bearnice-morning-brief.md` (the prompt, one paragraph; the job's whole behaviour is in it).
- What the prompt reads: `travel.json` (where he is, his local time, the weather from the Open-Meteo forecast API), Microsoft To Do (pending tasks), yesterday's `memory/` note, `site/data/runway.json` (trips starting in 21 days, decisions due in 14), `site/data/jobs.json` (the job board), `node /home/node/tools/concert-radar/cards.mjs --going` (shows he is going to) and, on Fridays, `--weekend`, and `node /home/node/tools/events/scan.mjs --brief` (today and tomorrow).
- Env: `DISCORD_CHANNEL_DAILY_BRIEF` (resolved by `scripts/jobs.py` into the delivery target).

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-morning-brief` (display name "Daily Morning Briefing") | `0 10 * * *`, `tz: bear`, isolated session | `deepseek/deepseek-flash` | #daily-brief (`announce`) |

## Discord

- #daily-brief: one post a day, for Bear to read.
- Her final reply is the post, so the job must end with the brief itself and nothing else.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `bearnice-morning-brief` `lastStatus` `ok`, `consecutiveErrors` 0, delivery to #daily-brief.
- Healthy: a post in #daily-brief each day around 10:00 his time, an opening line on where he is, no section for anything that has nothing to say.
- Each source can be checked alone without writing: `cards.mjs --going`, `scan.mjs --brief` and `cat site/data/runway.json` print JSON only.

## Gotchas

- **The time follows `travel.json`.** `tz: bear` in the job file means the zone in `travel.json` `current.timezone`, else `USER_TIMEZONE`, else Europe/Berlin. `scripts/crons.sh` sets it on each deploy; when he moves, Bearnice re-times the job with her `automations` tool (AGENTS.md lists the five jobs she moves). Until one of those happens the brief comes at 10:00 in the old zone.
- What the prompt forbids: listing a trip's tasks one by one, re-opening a decision already made, repeating job tasks that live in To Do, and inventing anything from the gig and events commands (use only their facts, leave the line out when they list none).
- The prompt orders what matters: a show or event tonight goes right after the opening line; trips get one line each; the job hunt gets exactly one line.
- Every section is optional in the prompt: if one is missing from the brief, run its source command (above) to see whether it had nothing to say.
- Pinned to DeepSeek so the DM model can change independently (`046dba1`).

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Moved to a job file with a prompt file (`3b36f0a`); pinned to DeepSeek (`046dba1`).
- 2026-11-05 Gigs he is going to, the Friday weekend block and events nearby added (`bc1c047`, `35bc100`, `64ff032`).
- 2026-11-03 Trips (one line each) and decisions due, from the runway (`e815e98`, `fc6c84f`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/events-radar|System: events radar]]
- [[entities/systems/gig-radar|System: gig radar]]
- [[entities/systems/runway|System: runway]]
<!-- openclaw:wiki:related:end -->
