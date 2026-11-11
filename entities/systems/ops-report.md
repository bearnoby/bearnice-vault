---
pageType: entity
entityType: system
id: entity.systems.ops-report
title: "System: ops report"
aliases:
  - ops
  - /ops/
  - bearnoby-ops-report
  - health report
sourceIds:
  - repo:scripts/ops-report.sh
  - repo:tools/ops
  - repo:tools/ops/README.md
updatedAt: 2026-11-11T17:12:48.000Z
---

# System: ops report

One file that says how the whole system is doing, and an alert when something changes. An hourly host timer gathers what only the host can see (disk, memory, containers, the backup unit, the vault sync status, the builder, the site certificate, the job list) and pipes it into `tools/ops/report.mjs` inside the gateway container, which adds what OpenClaw's databases know (runs, spend, tokens, tool calls, whether `MEMORY.md` loads) and writes `site/data/ops.json`. The `/ops/` page renders it. It is a host timer, not an OpenClaw job, so it still runs when OpenClaw's scheduler is down. No model. It watches every other system, in particular [[entities/systems/backups|Backups]], [[entities/systems/memory-and-vault|Memory and vault]], [[entities/systems/upgrades-and-builds|Upgrades and builds]] and [[entities/systems/taps|Taps]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `scripts/ops-report.sh` (host side; read-only; never prints `.env` values), `scripts/systemd/bearnoby-ops-report.{service,timer}`, `scripts/set-budget.sh`.
- Repo `tools/ops/`: `report.mjs` (report and alerts), `lib.mjs` (pure logic, tested), `page/` (`index.html`, `ops.js`, `ops.css`: the source of truth for the `/ops/` page), `install.mjs` (copies `page/*` into `site/ops/` when different and keeps `data/ops.json` out of the site's git), `docs.mjs` (writes `docs/JOBS.md` and `docs/CHANNELS.md`), `sample/ops.json`, and `README.md` (every field of `ops.json`, the alert rules). Tests in `tools/ops/test/`.
- Output: `site/data/ops.json` (atomic write, UTC times, a section that cannot be computed becomes `{ "error": ... }`), page `/ops/` (a footer link on every page; it uses the site's shared shell and `assets/base.css`).
- Her workspace: `ops/alerts-state.json` (which alerts are active), `ops/budget.json` (optional, `{ "anthropicMonthlyUsd": N }`). The databases are opened read-only.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnoby-ops-report.timer` (host systemd) | hourly at :20 (clear of the 07:15 backup and the :07/:22 vault sync), up to 60 s random delay | none | #mod-updates (alerts, plain text, no pings, `Ops: <text>`) |

## Discord

#mod-updates only, found by name in the workspace channel map. Alerts post once when a condition starts and once (`Resolved: ...`) when it clears. Conditions: an enabled recurring job failing 3 times running (`bearnice-taps` included), a backup older than 36 hours or a failed backup run, disk over 80%, `MEMORY.md` not loading, the live config lacking a repo key, a build session with no pid, a database-backup file at 90 MB, a vault sync that is conflicted, failing for an hour or not running, a new `openclaw.json.clobbered.*` file, and the Anthropic monthly budget crossing 50, 75, 90 and 100% (only when a budget is set). On the first run it posts one "Ops monitor started. Already active: ..." message.

## How to check it

- `site/data/ops.json` `generatedAt` is within the last hour or so, `errors` is empty and `hostFacts` is `present`. The `/ops/` page shows a "report is stale" banner after 2 hours, which means the timer is dead.
- `systemctl list-timers bearnoby-ops-report.timer` and `journalctl -u bearnoby-ops-report -n 20 --no-pager`.
- `scripts/ops-report.sh --dry-run` prints the JSON and writes and posts nothing; `--facts-only` prints just the host facts. `--no-alerts` does write `ops.json`, so leave it alone.
- Spend: `ops.json` `spend`; the budget is set with `bash scripts/set-budget.sh <usd|off>`.

## Gotchas

- The `/ops/` page is not edited on the site: change `tools/ops/page/`, deploy, and the next hourly run copies it over.
- `docs/JOBS.md` and `docs/CHANNELS.md` are generated from the live `ops.json` (`tools/ops/docs.mjs`; the command is in the header of `JOBS.md`) and are not edited by hand. Regenerate after adding, removing or rescheduling a job or channel.
- If the gateway container is not running, `ops-report.sh` exits 1 and nothing is alerted from the host.
- Unknown is not an alert: a section that fails produces no condition.
- The ops list and `docs/JOBS.md` show the heartbeat's model as "default model", while `config/openclaw.json` sets one for it (see [[entities/systems/heartbeat|Heartbeat]]). Check the live job if it matters.

## Change log

- 2026-11-11 Phase 4 site roll-out (`34c3231`): the `/ops/` page uses the site's shared shell and is linked from the footer.
- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Vault page count, last sync and a vault-sync alert (`80a8c5b`); budget alerts and `set-budget.sh` (`ff5a480`); channels come from the job files (`ce3bf08`); JOBS.md and CHANNELS.md regenerated (`bc0435c`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/backups|System: backups]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
