---
pageType: source
id: source.bridge.assistant-b3367344.memory-2026-11-05-a5cdf15c
title: "Memory Bridge (assistant): 2026-11-05"
sourceType: memory-bridge
sourcePath: /home/node/workspaces/assistant/memory/2026-11-05.md
bridgeRelativePath: memory/2026-11-05.md
bridgeWorkspaceDir: /home/node/workspaces/assistant
bridgeAgentIds:
  - assistant
status: active
updatedAt: 2026-11-05T13:37:18.295Z
---

# Memory Bridge (assistant): 2026-11-05

## Bridge Source
- Workspace: `/home/node/workspaces/assistant`
- Relative path: `memory/2026-11-05.md`
- Kind: `markdown`
- Agents: assistant
- Updated: 2026-11-05T13:37:18.295Z

## Content
```markdown


## Job board → forum (upgrades idea 1, rescoped twice)
- Bear's ruling: the feed stays exactly as built (cards in #job-hunt, ⭐📨❌, quick to scan). Only the tap outcome changes: ⭐/📨 promote the role into a **Community forum** as one post per role with tags; ❌ never goes on the board.
- Forum created by Bear: `job-board`, id `1567796469602459758` (type 15, tags/topic still empty). Community enabled; `DISCORD_CHANNEL_JOB_BOARD` added to gateway env.
- Migration is a copy, not a move (Discord can't reparent threads): 12 live threads, 50 messages, all authored by me — copy into board posts, then archive+lock the original threads. Nothing deleted, no taps re-run.
- Plan `plans/1-job-hunt-board.md` rewritten as a delta on the 3 Nov build; `plans/queue.md` already lists it. Waiting on Claude Code to build, then I verify: dry-run → one real ⭐ → migration → site links.
- Gap found: this session has no Discord `message` tool exposed, so I can't create/rename/tag channels even though AGENTS.md says I manage the server. Bear will fix when he can; bot itself is Administrator, and `channels.discord.actions.channels` is enabled by default, so it's only tool exposure.

## Durable lessons from the 24 Oct restart/recovery session (logged at flush, 5 Nov)

- **AgentMail key scope:** `AGENTMAIL_API_KEY` can list messages (`GET /v0/inboxes/<inbox>/messages` → 200) but **cannot** read a single inbox or message detail — those return 403 `missing_permission: inbox_read`. Use the list endpoint for checks; a broader key from the AgentMail console is needed for full body reads.
- **`microsoft_todo` MCP tools can silently vanish from the tool catalog** after a gateway restart ("bundle-mcp server not connected") even though the server process is healthy. `openclaw mcp probe` still reports 9 tools. The reliable fallback is a local node script that speaks JSON-RPC over stdio to `/home/node/tools/microsoft-todo/server.js` (`initialize` → `tools/call` with `list_task_lists` / `get_tasks{list_id,status}`). Always test before claiming the tools are unusable.
- **`get_tasks` needs `list_id`, not `listName`.** Passing a name silently falls back to the default *Tasks* list and made me report Shopping as empty when it had items. Resolve ids via `list_task_lists` first.
- **`heartbeat-state.json`** in the workspace is the heartbeat's own bookkeeping file (last check, seen mail message_ids, To-Do reminder date, travel snapshot). It is created by the heartbeat, not part of the bootstrap files. Keep it updated at the end of each heartbeat rather than relying on chat history.
- **Duplicate automations appear after restarts** — two jobs both named `bearnice-morning-brief` at 10:00 Europe/Berlin → #daily-brief existed at once, which would have double-posted. When a brief looks duplicated, compare `cron_jobs` by name + schedule and disable the stale one; never delete the one the current schedule points at.
- **System monitor jobs (`heartbeat:*`) return "system-owned monitor jobs cannot be edited by cron clients"** — they cannot be disabled/enabled/fixed via the automations CLI or tool. Report it instead of retrying.
- **Long `openclaw status` / `openclaw mcp probe` calls routinely exceed the default exec timeout** and get SIGTERMed; they are read-only, so a later clean re-run is the right recovery, not a replay of dependent work.
- **Gateway memory pressure warnings** (`heap_threshold` ~105–106% at ~590 MiB RSS) precede the container restarts that keep interrupting turns. It is a host/environment issue to flag to Bear, not something I can fix from inside.
- **Behaviour note for myself:** on a restart-interruption notice I should verify state first, then finish the original heartbeat check — and never invent a preference (e.g. claiming an 08:00 brief time) that isn't in USER.md. USER.md says 10:00 and the cron matches.

```

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
- No related pages yet.
<!-- openclaw:wiki:related:end -->
