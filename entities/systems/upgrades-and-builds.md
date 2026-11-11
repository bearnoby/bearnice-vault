---
pageType: entity
entityType: system
id: entity.systems.upgrades-and-builds
title: "System: upgrades and builds"
aliases:
  - upgrades
  - build runner
  - builder
  - bearnice-upgrades
sourceIds:
  - repo:jobs/bearnice-upgrades.yaml
  - repo:agents/assistant/skills/upgrades/SKILL.md
  - repo:builder/runner.py
  - repo:builder/BUILDER.md
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: upgrades and builds

Every Sunday Bearnice (Opus 5.5, high effort) looks at a week of Bear's life and, only if an idea clears her bar, DMs him a pitch for a whole system. Approved ideas become briefs in her workspace `plans/`, and a runner on the host starts one Claude Code session per brief, in queue order; the session plans with Bear in the Claude app, builds in stages and sets the brief's status. The finished build updates the matching note in [[entities/systems/systems|Bearnice's systems]]. The [[entities/systems/heartbeat|heartbeat]] watches the brief statuses. Never run the runner by hand.

## Files

- Repo `jobs/bearnice-upgrades.yaml`; her skill `upgrades` (how she looks and judges, the pitch format, what to do when he answers, "Builds"); `/upgrades` runs the job now through her `automations` tool.
- Her workspace: `ideas.json` (runs and ideas with `status`: proposed, discussing, approved, planned, declined, done), `plans/<id>-<name>.md` (the brief; first line `status: <value>`, optional `model:` line), `plans/queue.md` (the build order, one brief per line).
- Repo `builder/runner.py` (the runner), `builder/install.sh`, `builder/BUILDER.md` (the instructions every build session reads first, including "System notes"), tests in `builder/test/`.
- On the server, outside the repo: `/root/builder/` (`state.json`, `env` for the Claude Code token, `runner.log`, `enabled`, `lock`) and `/root/.claude/jobs/pins.json`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-upgrades` | `0 10 * * 0` (Sunday), Bear's time, isolated session | `anthropic/claude-opus-5-5`, thinking high | Bear's DM (`announce: dm`); `NO_REPLY` when no idea clears the bar |
| root cron `builder/runner.py` | every 10 min, installed by `builder/install.sh` | Claude Code per brief | Bear's DM (permission-prompt notice only) |

## Discord

Bear's DM only. He answers a pitch there ("yes", "no", "yes with changes", a question); a yes becomes a brief. The build itself is followed in the Claude app, not in Discord.

## Brief statuses

`queued`, then `building` (set by the runner), then `built` (set by the session once Bear signs off), or `blocked: <reason>`, or `stalled` (the session ended unfinished; nothing else starts until Bear's answer is recorded as `resume`, `queued` or `skipped`). `done` and `skipped` are ignored. Setting the open brief back to `queued` makes the runner close its session and start over.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `bearnice-upgrades` `ok` after each Sunday.
- The first line of each file in `plans/` is the status; `plans/queue.md` is the order.
- On the host, read-only: `ls /root/builder`, the keys of `/root/builder/state.json` (`current`, `sessions`, `notified`), and `crontab -l | grep runner`. The runner runs only while `/root/builder/enabled` exists. The `builder` section of `/ops/` shows the session; a session with no pid raises an ops alert.

## Gotchas

- **8-hour idle retire.** Claude Code retires a session waiting at a prompt for 8 hours and takes it offline in the app, though `claude agents --json` still lists it with no pid. The runner pins each session it starts (`pins.json`) and counts a listed session as running only if it has a pid. A dead session is marked `stalled`.
- **Permission prompts do not reach his phone.** A session stuck on one waits for a terminal; the runner DMs Bear once per prompt with how to clear it. Sessions are told to avoid prompts (Claude Code's own worktree tool, no worktrees elsewhere).
- **Model per brief.** A `model: sonnet` (or `opus`, `haiku`, or a plain model id) line under the status beats the `BUILDER_MODEL` environment variable, which beats the default `claude-opus-5-5`; effort is `BUILDER_EFFORT`, default high. Mechanical briefs take sonnet; design work leaves the line out. Anything else is ignored with a warning.
- A build changes repo files in its own worktree and live data in place; it merges and deploys only after Bear says so in the session.
- Once a brief is not `queued`, it belongs to the builder; Bearnice only changes its status as the skill says and never starts or messages a session herself.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-11 Build sessions read and update the system notes first and last (`ae97ee5`).
- 2026-11-10 A brief can choose its model with a `model:` line (`042bd87`); job moved to a file (`ea786bb`).
- 2026-11-05 Sessions are pinned and dead ones detected by pid (`71757a1`).
- 2026-11-03 Approved ideas become briefs; one Claude Code session per brief (`fc3dfce`, `f5e51bc`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/heartbeat|System: heartbeat]]
- [[entities/systems/ops-report|System: ops report]]
<!-- openclaw:wiki:related:end -->
