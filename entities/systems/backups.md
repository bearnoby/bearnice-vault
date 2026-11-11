---
pageType: entity
entityType: system
id: entity.systems.backups
title: "System: backups"
aliases:
  - memory backup
  - bearnoby-memory-backup
  - sessions branch
sourceIds:
  - repo:scripts/backup-memory.sh
  - repo:scripts/backup-excludes.txt
  - repo:scripts/verify-backup.sh
  - repo:docs/MIGRATION.md
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: backups

A host timer pushes Bearnice's live data to a private GitHub repo every night, in two halves. The `main` branch holds her whole workspace (`agents/assistant/`, minus what the repo already tracks and what is replaceable); the `sessions` branch holds OpenClaw's own database backup (chat history, cron state, channel state), with secrets redacted and big files cut into chunks. The persona, skills, config and tools are in the main repo on GitHub; the vault also has its own repo ([[entities/systems/memory-and-vault|Memory and vault]]); `.env` is Bear's own local copy. The [[entities/systems/ops-report|ops report]] alerts when the backup is old. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `scripts/backup-memory.sh` (the nightly script; `--sessions-only` stages and pushes whatever is in `backups/db-git`, for tests), `scripts/backup-excludes.txt` (what is left out; one list shared with the verify script), `scripts/sessions-chunks.sh` (split and join functions), `scripts/verify-backup.sh` (restore test), `scripts/restore-memory.sh` (for a new box), `scripts/systemd/bearnoby-memory-backup.{service,timer}`.
- Repo `docs/MIGRATION.md`: what lives where, the nightly backup, the sessions branch, a planned move, disaster recovery.
- Host: clones `/root/bearnoby-openclaw-memory` and `/root/bearnoby-openclaw-memory-sessions` (the staging clone, owned by root), OpenClaw's backup repo `backups/db-git`, deploy key `/root/.ssh/bearnoby_memory_deploy` (write access). Remote: `bearnoby/bearnoby-openclaw-memory`, private; it holds explicit content, so keep it private.
- Env overrides for tests: `MEMORY_REPO`, `MEMORY_DEPLOY_KEY`, `MEMORY_CLONE`, `MEMORY_SESSIONS_CLONE`, `MEMORY_DB_DIR`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnoby-memory-backup.timer` (host systemd, not an OpenClaw job) | daily 07:15 Europe/Berlin, up to 5 min random delay, runs missed ones | none | nowhere; the ops report alerts |

It runs after Bearnice's nightly memory upkeep (dreaming at 03:00). `deploy.sh` installs and enables the timer.

## Discord

None. A stale or failed backup is reported as one alert in #mod-updates by the hourly [[entities/systems/ops-report|ops report]].

## How to check it

- `systemctl list-timers bearnoby-memory-backup.timer` (next run) and `journalctl -u bearnoby-memory-backup -n 20 --no-pager` (last runs); neither writes anything.
- `site/data/ops.json` `backup`: the workspace and sessions halves each have an age in hours, healthy under 36; `lastRunResult` is `success`. The `/ops/` page shows the same.
- `git -C /root/bearnoby-openclaw-memory log -1 --format=%cd` is the last workspace commit.
- `bash scripts/verify-backup.sh` compares GitHub with the live workspace and is the real restore test. It clones to a temp folder and changes nothing, but needs the network and the deploy key; edits since 07:15 show up as differences, so run it after a backup.

## Gotchas

- The workspace is backed up **by exclusion**: a new file is covered the day it appears. Excluded: files the repo tracks (`SOUL.md`, `AGENTS.md`, `IDENTITY.md`, `USER.md`, `skills/`), `media/inbound/`, `jobs/pool.json`, `.backups/`, `*.bak*` and `*.prev*` copies, `node_modules/`, `.openclaw/`, `.trash/`, `memory/.dreams/`, and any `.git` (a nested repo would be recorded as an empty submodule link and fail the restore test). The vault's pages are in, as plain files.
- GitHub warns at 50 MB and rejects at 100 MB. Files on `sessions` over 40 MB are cut into line-aligned `.part-NNNN` chunks with a `.chunks` manifest (sha256, size, parts); earlier chunks stay identical night to night, so only the last chunk changes. The script fails if a staged file is over 95 MB. Two append-only transcript tables are the ones that grow.
- `backups/db-git` is left exactly as OpenClaw makes it, because its own restore needs it intact; only the pushed copy is chunked.
- The staging clone must stay owned by root or git refuses it as "dubious ownership".
- `restore-memory.sh` refuses to overwrite existing memory without `--force`, and rebuilds `backups/db-git-restored` for `openclaw backup git restore --repository`.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-10 Sessions-branch files over 40 MB split into chunks (`fefa7d1`); the whole workspace is copied by exclusion and a restore test added (`3fb94a5`); verify stops listing excluded files (`f9ba532`); staging clone kept owned by root (`3433010`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/memory-and-vault|System: memory and vault]]
- [[entities/systems/ops-report|System: ops report]]
<!-- openclaw:wiki:related:end -->
