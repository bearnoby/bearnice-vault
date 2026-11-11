---
pageType: entity
entityType: system
id: entity.systems
title: "Bearnice's systems"
aliases:
  - system notes
  - systems index
sourceIds:
  - repo:docs/VAULT.md
updatedAt: 2026-11-11T14:00:00.000Z
---

# Bearnice's systems

One note per system Bearnice runs: what it does, its files, jobs and channels, how to check it, its gotchas and a dated change log. Claude Code build sessions read the note before touching a system and update it when they finish (`builder/BUILDER.md`); Bearnice reads them when Bear asks how something works; Bear reads and edits them in Obsidian. A note is a map, not the truth: when the code or the server disagrees, they win and the note gets fixed.

## Where things are

- Repo: `/root/Bearnoby-OpenClaw` on the server (GitHub `bearnoby/Bearnoby-OpenClaw`). Jobs are `jobs/<name>.yaml` (`jobs/README.md`); the live list with last runs is `/ops/` on her site.
- Her workspace: `agents/assistant/` in the repo = `/home/node/workspaces/assistant/` in the gateway. "Workspace `jobs/`" in a note means `agents/assistant/jobs/`.
- Tools: `tools/` in the repo = `/home/node/tools/` (read-only) in the gateway; run them with `docker exec openclaw-gateway node /home/node/tools/...`.
- Her site: `site/` in the repo (its own repo, `bearnice-site`) = `/home/node/site/`; served at bearnice.lualderm.com.
- Times in "Bear's time" follow `travel.json` (`tz: bear` in a job file); Europe/Berlin at the time of writing.

## The notes

- [[entities/systems/job-feed|Job feed]]: jobs found, graded and carded in #job-hunt. Job `bearnice-job-sources`.
- [[entities/systems/job-board|Job board]]: taps to statuses, the #job-board forum, follow-ups, the job hunt page. Jobs `bearnice-taps`, `bearnice-job-board-followups`.
- [[entities/systems/cover-letters|Cover letters]]: letters on request in board posts, PDFs on his letterhead. Job `bearnice-taps`.
- [[entities/systems/gig-radar|Gig radar]]: one living card per band in #gig-radar, on-sale reminders. Jobs `bearnice-gig-radar-scan`, `bearnice-taps`.
- [[entities/systems/events-radar|Events radar]]: what's on in the next 7 days, cards in #events-radar. Jobs `bearnice-events-radar`, `bearnice-events-weekend`, `bearnice-taps`.
- [[entities/systems/runway|Runway]]: where Bear will be, his stay clocks, travel decisions. Job `bearnice-runway`.
- [[entities/systems/morning-brief|Morning brief]]: the 10:00 post in #daily-brief. Job `bearnice-morning-brief`.
- [[entities/systems/heartbeat|Heartbeat]]: her check-in every 2 hours (mail, To Do, trips, builds). Job `heartbeat-assistant`.
- [[entities/systems/upgrades-and-builds|Upgrades and builds]]: Sunday ideas, briefs, the build runner. Job `bearnice-upgrades`, root cron `builder/runner.py`.
- [[entities/systems/taps|Taps dispatcher]]: one 10-minute pass over Bear's taps on all cards. Job `bearnice-taps`.
- [[entities/systems/memory-and-vault|Memory and vault]]: `MEMORY.md`, daily notes, dreaming, this vault and its sync. Job `Memory Dreaming Promotion`, `bearnoby-vault-sync.timer`.
- [[entities/systems/backups|Backups]]: nightly push of her workspace to a private repo. `bearnoby-memory-backup.timer`.
- [[entities/systems/ops-report|Ops report]]: the hourly health report, `/ops/`, alerts in #mod-updates. `bearnoby-ops-report.timer`.
- [[entities/systems/website|Website]]: her site, publishing, the `/me/` notes endpoint. No job of its own.

## Keeping the notes

- The shape and the rules are in `docs/VAULT.md` ("System notes") in the repo.
- They are live files in the vault, not repo files: edit them in place, never in a git worktree, never with git. The vault sync commits them within 15 minutes.
- They belong to UID 1000 (the gateway's user). A session running as root runs `chown 1000:1000` on every note it writes.
- The `## Related` block at the end is written by the wiki compiler; the Notes block is for Bear and Bearnice. Build sessions leave both alone.

## Change log

- 2026-11-11 Notes started (V3): this page and the job feed, job board and cover letters notes.

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/backups|System: backups]]
- [[entities/systems/cover-letters|System: cover letters]]
- [[entities/systems/events-radar|System: events radar]]
- [[entities/systems/gig-radar|System: gig radar]]
- [[entities/systems/heartbeat|System: heartbeat]]
- [[entities/systems/job-board|System: job board]]
- [[entities/systems/job-feed|System: job feed]]
- [[entities/systems/memory-and-vault|System: memory and vault]]
- [[entities/systems/morning-brief|System: morning brief]]
- [[entities/systems/ops-report|System: ops report]]
- [[entities/systems/runway|System: runway]]
- [[entities/systems/taps|System: taps dispatcher]]
<!-- openclaw:wiki:related:end -->
