---
pageType: entity
entityType: system
id: entity.systems.discord-access
title: "System: Discord access"
aliases:
  - discord_read
  - discord reader
  - message tool
  - Discord read access
sourceIds:
  - repo:tools/discord-read
  - repo:config/openclaw.json
  - repo:agents/assistant/AGENTS.md
updatedAt: 2026-11-23T21:00:00.000Z
---

# System: Discord access

What Bearnice can see and do on Discord herself, in any session (DM, server channels, scheduled jobs). She reads with `discord_read`, a read-only tool built for her: channels, threads and forum posts, messages with author, time, replies, reactions and attachments, attachment text (message.txt), and search. She organises the server (make, rename, move, archive channels; threads; pins) with OpenClaw's own `message` tool. Roles, kicks, bans, timeouts and presence are off. Built 21-23 Nov 2026 (brief 15) after she could only read the IKEA post on the [[entities/systems/job-board|job board]] by borrowing a job tool's Discord client. Hub: [[entities/systems/systems|Bearnice's systems]].

## How it runs

- `discord_read` is an MCP server the gateway keeps running (`mcp.servers.discord_read`). Its five tools reach her as `discord_read__list_channels`, `__list_threads`, `__read_messages`, `__read_attachment`, `__search_messages`. In Sonnet sessions they sit behind Code Mode's `exec` (she sees only `exec`, `wait`, `view_image`; everything else is called from inside `exec`).
- It makes only GET calls, plus the one POST Discord uses to look up Bear's DM channel. Scope: Bearnoby Personal (guild `1563148717903577148`) and Bear's DM; anything else is refused before its messages are read.
- `read_messages` takes an id, a Discord link (a message link reads around that message), a #name (partial names and open threads/posts too) or `dm`. Times are in Bear's timezone from `travel.json`. Up to 100 messages a call, oldest first, with before/after/around paging.
- `read_attachment`: text files come back inline (up to 60,000 characters); images, PDFs and other files up to 25 MB are saved to `media/inbound/discord/<message id>-<name>` in her workspace (not backed up) for `view_image`.
- `search_messages` is Discord's own guild search (words, author `bear`/`me`/id, channel, has=file/image/link…). It doesn't cover the DM.
- `message` tool: in `tools.alsoAllow` since 23 Nov. Before that her agent's `coding` profile dropped it before her allow list was read, so her chats never had it; scheduled jobs that post (the board follow-ups) got it through the gateway's forced-delivery path and kept working.

## Files

- Repo `tools/discord-read/`: `server.js` (MCP wiring, tool definitions, read-only annotations), `lib/read.mjs` (all reading and formatting), `test/read.test.mjs`, `package.json` (`@modelcontextprotocol/sdk`). Uses `tools/lib/discord.mjs`.
- Repo `config/openclaw.json`: `mcp.servers.discord_read` (env `DISCORD_BOT_TOKEN_BEARNICE`, `BEARNICE_WORKSPACE`, `USER_TIMEZONE`), `discord_read__*` in the agent's `tools.allow`, `message` in `tools.alsoAllow`, `channels.discord.accounts.bearnice.actions` (channels, messages, pins, threads, reactions on; roles, moderation, presence off).
- Repo `agents/assistant/AGENTS.md`, section 5 (The Discord server): read it yourself, never reconstruct from files; manage with `message`; never re-send the reply; never post in #job-board posts.
- Her workspace: `media/inbound/discord/` (saved attachments).

## Jobs

None of its own. Every session of hers has it, scheduled jobs included.

## Discord

Reads the whole of Bearnoby Personal and Bear's DM. Writes only through `message`, when Bear asks or a job's instructions say so.

## How to check it

- `docker exec openclaw-gateway openclaw mcp probe`: `discord_read: 5 tools`.
- Live read without a model: copy `tools/discord-read/lib/read.mjs` into the container and call `createReadTools` with `createDiscord` from `/home/node/tools/lib/discord.mjs` (as the build did), e.g. `read_messages({channel: "1572983273339228190"})` for the IKEA post.
- Offline: `bash scripts/test.sh --node` (suite `discord-read`, 9 tests).
- Did she use it: `openclaw sessions export-trajectory` for her DM session key, count `tool.call` names (`discord_read__read_messages`: 19 calls on 23 Nov).

## Gotchas

- `openclaw gateway call tools.effective` under-reports: it showed neither `message` nor `browser` for scheduled jobs that were calling both successfully. Don't use it as proof a tool is missing; look at real tool calls.
- Global `tools.alsoAllow` is what feeds her profile, because the agent has its own `allow` list (OpenClaw rejects `allow` and `alsoAllow` in the same scope).
- Discord's search can answer "still indexing" (code 110000) on a fresh index; the tool says so and suggests `read_messages`.
- As of 23 Nov, `message` in her DM hadn't been used yet after the deploy; the first channel she makes or archives confirms it.

## Change log

- 2026-11-23 build/15-discord-read-access: note written. Stage 1 (`eddb5fa`, deployed 23 Nov) the `discord_read` reader; stage 3 (`e5866d3`, deployed 23 Nov) `message` back in her chats, roles/moderation/presence off, read-only annotations.

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/heartbeat|System: heartbeat]]

### Related Pages

- [[entities/systems/memory-and-vault|System: memory and vault]]
<!-- openclaw:wiki:related:end -->
