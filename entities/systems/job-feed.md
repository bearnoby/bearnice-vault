---
pageType: entity
entityType: system
id: entity.systems.job-feed
title: "System: job feed"
aliases:
  - job hunt feed
  - job sources
  - bearnice-job-sources
sourceIds:
  - repo:tools/job-sources
  - repo:jobs/bearnice-job-sources.yaml
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: job feed

Finds jobs for Bear's move to Europe and keeps the best ones as cards in #job-hunt. A fetch with no model pulls company boards, job boards and forwarded job-alert mail into a pool; Claude Sonnet 5.5 grades each new job once (0-100, Batch API); a feed step with no model keeps the top 24 A and B cards in the channel. What happens after a tap is the [[entities/systems/job-board|job board]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `tools/job-sources/`: `run.sh` (the job: fetch, grade, feed; each step runs even if the one before failed), `fetch.mjs` (sources, dedupe, screening, pool), `grade.mjs` (Sonnet batch), `score.mjs` (the arithmetic and its defaults), `feed.mjs` (cards and the status message). Tests in `tools/job-sources/test/`. One npm dependency, `@anthropic-ai/sdk`.
- Repo `jobs/bearnice-job-sources.yaml`; her skills `job-scout` ("The feed runs itself", "Your part in the feed") and `scout-now`.
- Her workspace `jobs/`: `profile.json` (what he wants; `phase` is `europe-first`), `companies.json` (employers; a `board` there makes the fetch poll it: 45 of 63 have one), `sources.json`, `grading.json` (weights, bands A 75, B 60), `pool.json` (found jobs, grades, `last_run`, `grading.log`, `grading.spent_usd`, `feed.status_message_id`), `seen.json` (each card is a listing with `status: new`, `message_id`, `pool_key`, `score`).
- Env (gateway): `ANTHROPIC_API_KEY`, `JOB_GRADER_MODEL` (default `claude-sonnet-5-5`), `AGENTMAIL_API_KEY`, `AGENTMAIL_INBOX`, `DISCORD_BOT_TOKEN_BEARNICE`, `DISCORD_CHANNEL_JOB_HUNT`, `JOB_SOURCES_DIR`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-job-sources` | `0 10-22/2 * * *`, Bear's time (Europe/Berlin now), timeout 30 min | none; the grader calls Sonnet 5.5 itself | #job-hunt |

`/scout_now` (skill `scout-now`) runs the job now, also outside its hours. When the feed posts new cards it runs `tools/job-board/check.mjs` once itself, so they get their taps without waiting for the [[entities/systems/taps|taps]] pass.

## Discord

- #job-hunt (text): up to 24 cards, best last, plus one status message edited each run (feed size, pool, last fetch's funnel, failed sources). `feed.size`, `bump_margin` (3) and `max_per_company` (3) are code defaults.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `bearnice-job-sources` has `state.lastStatus` `ok` and `consecutiveErrors` 0. A run takes about 3 minutes.
- `pool.json` `last_run`: `at` within the last 2 hours between 10:00 and 22:00, `failed` empty. From the repo root: `python3 -c 'import json;r=json.load(open("agents/assistant/jobs/pool.json"))["last_run"];print(r["at"],r["funnel_line"],r["failed"])'`
- Without writing anything: `docker exec openclaw-gateway node /home/node/tools/job-sources/fetch.mjs --dry-run --only=<source>` (add `--why=<source>` for each posting's fate) and `node /home/node/tools/job-sources/feed.mjs --dry-run`.
- Grading spend is `pool.json` `grading.spent_usd` ($1.99 in total on 11 Nov) and the Spend section of `/ops/`.

## Gotchas

- `sources.json` on the server only sets `countries` and `sectors_liked`. Queries, boards, ages, refresh hours and the feed's size are the defaults in `fetch.mjs` (`DEFAULTS`) and `feed.mjs` (`FEED`) until a key is added to the file. The skill describes them as living in `sources.json`.
- Changing weights in `grading.json` re-scores for free on the next run; changing `fit_max` or what fit means needs `grade.mjs --regrade` (about $1-2).
- A card leaves only by a tap or by a better job (3 points or more) bumping it; the bumped job goes back to the pool. Age never removes a card.
- Arbeitsagentur answers this host with 403 (shown as "blocked"); Microsoft careers rate-limits, so it stops at its first 429 and resumes with the countries it reached least recently.
- Mail: only messages whose sender or subject matches LinkedIn, ReLocue, "job" or "Fwd" are read; re-reading one is harmless.
- The job's time zone follows Bear: `crons.sh` sets it from `travel.json` on each deploy, and she re-times it with `automations` when he moves (AGENTS.md).

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and read-only server checks.
- 2026-11-10 `normCo` keeps "B.V." out of the dedupe key (R3, `e7756f3`); moved to `jobs/bearnice-job-sources.yaml` (`26166ca`); ported to `tools/lib` (`9498337`).
- 2026-11-09 Built: no-model fetch with per-source refresh, LinkedIn and ReLocue mail, Sonnet grading on Bear's 50/50 score, top-24 feed, at most 3 cards per company. Replaced the Mon/Wed Opus scout (`bearnice-job-scout`, retired).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/job-board|System: job board]]
<!-- openclaw:wiki:related:end -->
