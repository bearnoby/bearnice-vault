---
pageType: entity
entityType: system
id: entity.systems.job-board
title: "System: job board"
aliases:
  - job board follow-ups
  - board check
  - bearnice-job-board-followups
sourceIds:
  - repo:tools/job-board
  - repo:jobs/bearnice-job-board-followups.yaml
  - repo:tools/taps
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: job board

Turns Bear's taps on the [[entities/systems/job-feed|job feed]] cards into a record of the roles he acts on. A no-model check reads ⭐ 📨 ❌ in #job-hunt, writes the status to `seen.json`, moves acted-on roles to a post each in the #job-board forum (tagged by status and track) and publishes the job hunt page. A DeepSeek job then writes the follow-ups into each post: resume pick, visa route, apply-by date, follow-up date, visa case. Letters are the [[entities/systems/cover-letters|cover letters]] system. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `tools/job-board/check.mjs` (the check; exports `tick`), `tools/lib/taps.mjs` (reading taps), `tools/lib/site.mjs` (publishing). `migrate-to-board.mjs` was a one-off (old card threads to forum posts); no listing is left for it. Tests in `tools/job-board/test/`.
- Repo `jobs/bearnice-taps.yaml` (the `job-board` handler) and `jobs/bearnice-job-board-followups.yaml` + `.md` (prompt: "follow the Board follow-ups section of your job-scout skill"). Her skill `job-scout`: "Two layers", "Board follow-ups", "His replies".
- Her workspace `jobs/`: `seen.json` (every listing, its status and board fields), `board-state.json` (the follow-ups wake-up), `visa/routes.md`, `visa/eligibility.md`, `visa/cases/<company>.md`, `sources/resume-2026-swe.pdf` and `-tpm.pdf`.
- Site: `site/data/jobs.json` (written by the check from `seen.json`, never by hand), page `/research/job-hunt/` (moves to `/jobs/` in Phase 4).
- Env: `DISCORD_BOT_TOKEN_BEARNICE`, `DISCORD_CHANNEL_JOB_HUNT`, `DISCORD_CHANNEL_JOB_BOARD`; optional `JOB_BOARD_OWNER`, `JOB_BOARD_SEEN`, `JOB_BOARD_SITE`, `JOB_BOARD_GUILD`, `JOB_BOARD_QUIET=1` (no publish, no wake-up; tests).

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-taps`, handler `job-board` | every 10 min (8 min limit, lock taken over after 60 min) | none | #job-hunt, #job-board |
| `bearnice-job-board-followups` | 09:30 Bear's time, and woken by the check | `deepseek/deepseek-flash`, thinking high, isolated, no announce | #job-hunt, #job-board (message tool) |

The check wakes the follow-ups (`openclaw cron run`) when the set of pending work changes, again every 30 minutes while it is unchanged, at most 3 times; then it waits for 09:30.

## Discord

- #job-hunt (text): the feed cards. The check seeds ⭐ 📨 ❌; ❌ beats 📨 beats ⭐; ❌ deletes the card.
- #job-board (forum): one post per role that is interested, applied, interviewing or offer, with 📨 ❌ (📨 = applied; ❌ or deleting the post = passed). Tags: one status (⭐ 📨 🎤 🎉) plus one track (A sponsored, B remote, C contract, D US), resolved by name each run. Bear retagging 🎤 or 🎉 sets that status (`status_source: tag`). Bearnice answers there only when @mentioned (`requireMention` on the forum).

## How to check it

- `agents/assistant/taps/state.json`: `handlers.job-board.failures` 0 and `last_ok` within 10 minutes.
- `docker exec openclaw-gateway node /home/node/tools/job-board/check.mjs --dry-run`: reads Discord, writes nothing; `pending` lists follow-up work still owed (`<id>:star`, `:applied`, `:case`).
- `openclaw cron list --all --json`: `bearnice-job-board-followups` `state.lastStatus`.
- On 11 Nov: 87 listings (36 interested, 24 new, 19 passed, 7 applied, 1 closed), 44 posts on the board.

## Gotchas

- The check owns reactions, tags, board posts and reaction-set statuses. Bearnice sets a status only from a reply (`status_source: reply`); interviewing, offer and closed, and any reply-set status, are never changed by the check. Statuses other than `new` are sticky.
- Four writers share `seen.json`: the feed, the check, the letters and the follow-ups agent. Each re-reads and merges only its own fields before writing; keep it that way. The feed runs the check as its own process, outside the taps lock.
- Job tracking never goes to Microsoft To Do; old `apply_task_id` / `followup_task_id` fields are ignored.
- The 09:30 run on 11 Nov ended with status `error` ("Exec failed", `consecutiveErrors` 0). Cause not looked into yet.
- `stayClocks` overflowed at month ends until R3 (`e7756f3`); the board prefers the runway's clocks from `site/data/runway.json` when they are from today.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and read-only server checks.
- 2026-11-10 The check became the `job-board` handler of `bearnice-taps`, replacing `bearnice-job-board-check` (deployed 11 Nov, `45ce147`); follow-ups pinned to DeepSeek; moved to job files.
- 2026-11-09 No more To Do tasks; apply-by and follow-up live in the board post (`fd87c4c`).
- 2026-11-06 Bearnice answers in #job-board only when mentioned (`3853972`).
- 2026-11-05 Taps move roles from #job-hunt to the #job-board forum; ❌ deletes the card; hand retags update the role.

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/cover-letters|System: cover letters]]
- [[entities/systems/job-feed|System: job feed]]
- [[entities/systems/morning-brief|System: morning brief]]
- [[entities/systems/runway|System: runway]]
- [[entities/systems/taps|System: taps dispatcher]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
