---
pageType: entity
entityType: system
id: entity.systems.heartbeat
title: "System: heartbeat"
aliases:
  - heartbeat-assistant
  - check-in
  - heartbeats
sourceIds:
  - repo:heartbeats/assistant.md
  - repo:config/openclaw.json
  - repo:scripts/crons.sh
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: heartbeat

Bearnice's check-in every 2 hours. OpenClaw runs it as its own job in an isolated session; she follows the instructions in `heartbeats/assistant.md`: new mail, tasks due, trips, runway decisions, build statuses. With nothing to say she answers `HEARTBEAT_OK` and nothing is sent. Her final reply goes nowhere, so anything for Bear is sent as a DM with the message tool first. It watches the [[entities/systems/runway|runway]] and the briefs of the [[entities/systems/upgrades-and-builds|upgrades and builds]]. Hub: [[entities/systems/systems|Bearnice's systems]].

## Files

- Repo `heartbeats/assistant.md`: the instructions. `scripts/crons.sh` pushes the file into the heartbeat job's "scratch" with `openclaw cron scratch <id> --file -` on every deploy; the file in the repo is the source of truth and replaces the scratch. OpenClaw no longer reads a workspace `HEARTBEAT.md`.
- Repo `config/openclaw.json`: the defaults name `assistant` as the heartbeat agent, and `agents.entries.assistant.heartbeat` sets `every: 2h`, `isolatedSession: true` and `model: deepseek/deepseek-v4-flash`. `AGENTS.md` ("Heartbeats") says to follow them.
- Her workspace: `heartbeat-state.json` (when she last checked, IDs of mail she already told him about, one-time notes such as a runway decision she drafted), `travel.json`, `decisions.json`, `plans/*.md` (first line `status: ...`), the AgentMail inbox, Microsoft To Do, and `site/data/runway.json`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `heartbeat-assistant` (OpenClaw's own, no file in `jobs/`) | every 2 h | `deepseek/deepseek-v4-flash` (set in the config; the ops list shows "default model", and the run records of 11 Nov show this model) | nowhere; Bear's DM through the message tool (account `bearnice`) |

## Discord

Bear's DM only, at most one message per heartbeat. Quiet hours are 23:00 to 09:00 his local time (`TZ=<current.timezone> date`): only something genuinely urgent then.

## What one heartbeat does

1. Reads `heartbeat-state.json`; lists AgentMail messages newer than the last check; DMs a summary of what needs him (not newsletters, receipts or spam). A booking confirmation for one of his trips is filled into `travel.json` and ticked off in that trip's To Do list, with the new count in the DM.
2. To Do tasks due today or overdue that she has not reminded him about today.
3. Travel: an `upcoming` trip that has started becomes `current` and her jobs are re-timed; if `current.until` has passed she asks once where he is, and never switches him home on her own.
4. Runway: a gap starting within 28 days with no covering decision means she drafts one (once); a decision due within 3 days gets one line a day at most.
5. Builds: reads the first line of each brief in `plans/`; for `built`, `blocked` or `stalled` she follows "Builds" in her upgrades skill; a blocked or stalled one is asked about only once.

## How to check it

- `docker exec openclaw-gateway openclaw cron list --all --json`: `heartbeat-assistant` `kind` heartbeat, `lastStatus` `ok`, a run within the last 2 hours.
- A healthy heartbeat is usually silent. `heartbeat-state.json` is rewritten each heartbeat.
- The scratch is current when the last deploy printed `heartbeat instructions for assistant`.

## Gotchas

- Instructions change only through a deploy; editing the job's scratch by hand is overwritten by the next one. If no heartbeat job exists yet, `crons.sh` warns and loads them on the next deploy.
- Her final reply is not delivered. Anything she writes there instead of using the message tool is lost.
- Routine checks, empty inboxes and errors must not go into her memory notes.

## Change log

- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-03 Runway and builds added to the heartbeat (`fc6c84f`, `fc3dfce`); DeepSeek direct (`0b63a6f`).
- 2026-10-24 Heartbeat every 2 hours with real instructions (`11e3921`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/ops-report|System: ops report]]
- [[entities/systems/runway|System: runway]]
- [[entities/systems/upgrades-and-builds|System: upgrades and builds]]

### Related Pages

- [[entities/systems/memory-and-vault|System: memory and vault]]
<!-- openclaw:wiki:related:end -->
