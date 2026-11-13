---
pageType: entity
entityType: system
id: entity.systems.memory-and-vault
title: "System: memory and vault"
aliases:
  - memory
  - dreaming
  - vault sync
  - bearnoby-vault-sync
  - Memory Dreaming Promotion
sourceIds:
  - repo:docs/VAULT.md
  - repo:scripts/vault-sync.sh
  - repo:config/openclaw.json
  - repo:agents/assistant/skills/vault/SKILL.md
updatedAt: 2026-11-13T00:00:00.000Z
---

# System: memory and vault

How Bearnice remembers. `MEMORY.md` is the small file of durable facts that loads into her private sessions; `memory/YYYY-MM-DD.md` are the day's notes. Every night OpenClaw's dreaming (the `memory-core` plugin) promotes durable things into `MEMORY.md` and prunes stale ones. The vault is the larger organised store, kept by the `memory-wiki` plugin in bridge mode: it imports her memory, daily notes and dream reports, she adds linked pages, and a host timer syncs it with a private GitHub repo that Bear reads in Obsidian. These notes live in it. Hub: [[entities/systems/systems|Bearnice's systems]]. The vault's pages are also in the nightly [[entities/systems/backups|backup]].

## Files

- Her workspace: `MEMORY.md`, `memory/` (daily notes), `vault/` (the work tree: `WIKI.md`, `index.md`, `entities/` with `entities/systems/`, `concepts/`, `syntheses/`, `sources/`, `reports/`, `.openclaw-wiki/`).
- Repo `config/openclaw.json`: `plugins.entries["memory-core"].config.dreaming` (on, `0 3 * * *` in `USER_TIMEZONE`, pinned `model: deepseek/deepseek-flash`), `plugins.entries["memory-wiki"]` (bridge mode, global scope, path `/home/node/workspaces/assistant/vault`, Obsidian render mode, search corpus `all`, no compiled digest in her prompt), `memory.search.provider` `none`, the `session-memory` hook (50 messages), and the five `wiki_*` tools in her allowlist.
- Repo `docs/VAULT.md` (layout, first deploy, sync, conflicts, Obsidian on Windows), `scripts/vault-sync.sh`, `scripts/systemd/bearnoby-vault-sync.{service,timer}`, her skill `vault`, `AGENTS.md` ("Memory").
- Host: git dir `/root/bearnice-vault.git` (root-owned, outside her workspace, so nothing she can write is run by root), its `sync-status`, deploy key in `secrets/vault-deploy/`. Remote: `bearnoby/bearnice-vault` (private).

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `Memory Dreaming Promotion` (OpenClaw's own) | daily 03:00 Europe/Berlin | `dreaming.model` `deepseek/deepseek-flash` | nowhere |
| `bearnoby-vault-sync.timer` (host) | 7, 22, 37 and 52 past each hour | none | nowhere; alerts via the ops report |

The sync commits what she changed (author Bearnice), rebases onto Bear's pushes, pushes `main`, and hands the work tree back to UID 1000. It never force-pushes or resets. A rebase that conflicts is aborted at once, her commits go to the branch `bearnice-conflict`, and an alert is raised until a later sync succeeds.

## Discord

None. Bear reads and edits the vault in Obsidian after a `git pull`; the ops alert for a conflicted or dead sync goes to #mod-updates.

## How to check it

- `cat /root/bearnice-vault.git/sync-status` (read-only): `result=ok`, a recent last-ok time. `systemctl list-timers bearnoby-vault-sync.timer` shows the next run.
- `docker exec openclaw-gateway openclaw wiki status` and `wiki doctor` read the setup. `wiki lint` also compiles (it rewrites indexes and `## Related` blocks), so run it deliberately.
- Is `MEMORY.md` in her prompt? `site/data/ops.json` `memory.loading` is true, with `provenance.status` not `untrusted` and `latestDmPrompt.includesMemoryMd` true. The `/ops/` page shows the same, and an alert fires when it is false.

## Gotchas

- **The provenance taint is sticky.** Content from the web, an email, the browser or session history marks everything she writes afterwards as untrusted, and an untrusted `MEMORY.md` is silently dropped from her prompt. So "remember X" is written to `MEMORY.md` first in the turn, before any lookup; otherwise it goes in today's note and dreaming promotes it. The old custom nightly rewrite (`bearnice-reflection`) caused it by reading session histories and was retired for stock dreaming. To clear it, delete the row `core:memory-artifact-provenance` for `MEMORY.md` in `plugin_state_entries` (done once on 10 Nov; the copy is in `/root/archive/`). Which run tainted it is not known.
- Vault files belong to UID 1000. A root session writing a note runs `chown 1000:1000` on it, or the next compile cannot rewrite its `## Related` block.
- Pages outside `entities/`, `concepts/`, `syntheses/`, `sources/` and `reports/` are ignored by the compiler and by search, without a warning.
- The compiler never deletes a page it did not create; compile and lint run only when someone runs them.
- Never put keys, tokens or booking logins in the vault: it goes to GitHub and to Bear's PC.

## Change log

- 2026-11-11 Each vault sync now ends by rebuilding `site/data/vault.json` for the `/vault/` page (`tools/vault-page/build.mjs`, run in the gateway container; best effort, never changes the sync result) (`dce6bde`). See [[entities/systems/website|System: website]].
- 2026-11-11 Note written (V3) from the repo at `c076417` and the live job list of 11 Nov.
- 2026-11-11 `wiki_*` tools added to `tools.alsoAllow` (`85dcb60`); build sessions keep these notes (`ae97ee5`).
- 2026-11-10 `memory-wiki` enabled in bridge mode (`0520b8f`); sync script and 15-minute timer (`9c1b014`); vault page count and sync alert in the ops report (`80a8c5b`).
- 2026-11-02 Stock OpenClaw memory and dreaming replace the custom reflection (`28c5b40`).

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[entities/systems/backups|System: backups]]
- [[entities/systems/ops-report|System: ops report]]

### Related Pages

- [[entities/systems/heartbeat|System: heartbeat]]
<!-- openclaw:wiki:related:end -->
