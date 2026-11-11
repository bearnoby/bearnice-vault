---
pageType: source
id: source.bridge.assistant-b3367344.memory-2026-11-11-1251-61ed0e87
title: "Memory Bridge (assistant): 2026-11-11-1251"
sourceType: memory-bridge
sourcePath: /home/node/workspaces/assistant/memory/2026-11-11-1251.md
bridgeRelativePath: memory/2026-11-11-1251.md
bridgeWorkspaceDir: /home/node/workspaces/assistant
bridgeAgentIds:
  - assistant
status: active
updatedAt: 2026-11-11T11:51:21.396Z
---

# Memory Bridge (assistant): 2026-11-11-1251

## Bridge Source
- Workspace: `/home/node/workspaces/assistant`
- Relative path: `memory/2026-11-11-1251.md`
- Kind: `markdown`
- Agents: assistant
- Updated: 2026-11-11T11:51:21.396Z

## Content
```markdown
# Session: 2026-11-11 12:51:21 Europe/Berlin

- **Session Key**: agent:assistant:discord:bearnice:direct:bear
- **Session ID**: 467d351f-d530-47b7-9612-27e79e490ec0
- **Reason**: idle

## Conversation Summary

user: "What vault tools do you have?"
assistant: "Fewer than my vault skill says. I checked just now.\n\n**What I can use**\n- 🌸 `memory_search` finds things across my memory, the vault and past sessions. With `corpus=all` or `corpus=wiki` it covers the vault pages too.\n- 🌸 `memory_get` pulls a specific page or line range once I know the path.\n- 🌸 File tools (read, write, edit) and `exec` for the vault folder at `/home/node/workspaces/assistant/vault`.\n- 🌸 `openclaw wiki ingest <file>` through `exec` adds new material.\n\n**What I can't use**\nMy vault skill mentions `wiki_search`, `wiki_get`, `wiki_apply` and `wiki_lint`. None of them are in my tool list, so I can't call them. I can still search and read the vault with the memory tools. For filing, I'd write a page or ingest a file instead. I can't run the lint check or the metadata edits.\n\nIf you want those four tools, they need enabling on the gateway side. 🐻"

```

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
- No related pages yet.
<!-- openclaw:wiki:related:end -->
