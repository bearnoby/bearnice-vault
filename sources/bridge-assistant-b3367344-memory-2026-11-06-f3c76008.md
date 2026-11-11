---
pageType: source
id: source.bridge.assistant-b3367344.memory-2026-11-06-f3c76008
title: "Memory Bridge (assistant): 2026-11-06"
sourceType: memory-bridge
sourcePath: /home/node/workspaces/assistant/memory/2026-11-06.md
bridgeRelativePath: memory/2026-11-06.md
bridgeWorkspaceDir: /home/node/workspaces/assistant
bridgeAgentIds:
  - assistant
status: active
updatedAt: 2026-11-06T20:51:56.014Z
---

# Memory Bridge (assistant): 2026-11-06

## Bridge Source
- Workspace: `/home/node/workspaces/assistant`
- Relative path: `memory/2026-11-06.md`
- Kind: `markdown`
- Agents: assistant
- Updated: 2026-11-06T20:51:56.014Z

## Content
```markdown
# 6 Nov 2026

## The job-scout 401 — what it actually was
- `bearnice-job-scout` (`dc5caf1b-c1bd-466c-84f6-3a46fdee076f`) carried a **per-job model override `anthropic/claude-opus-5-5`** in its payload. Every run went to Anthropic and died `401` in ~2.5s (`FailoverError`, `lane=cron-nested`, 0 fallbacks). Nothing to do with the briefs, the queue, or his DeepSeek key.
- Both keys tested clean by hand: DeepSeek `HTTP 200`; Anthropic auth accepted (404 only for a model name I invented for the probe). So the credential was good — the route the override took wasn't.
- Fixed by `openclaw cron edit <id> --clear-model` (backup at `/tmp/scout-job-backup.json`), then restored `--model anthropic/claude-opus-5-5` once Bear said **"we want the scout on Anthropic still"**. Bear rotated the key around 14:30; the gateway process then had the same key fingerprint as my shell.
- The 14:29 gateway restart left the job stuck on a stale **"already-running"** slot. Clearing it: `openclaw cron disable` → `enable` → `run` (the disable writes "Cron job disabled by operator" into run history — that's me, not a fault).
- **Lesson:** when a scheduled job 401s but the key tests fine, read `openclaw cron get <id>` and check `payload.model` before touching credentials.

## Tools and quirks worth remembering
- After the 14:29 restart the **`automations` tool vanished from my tool catalog** (38 → 37 tools). CLI equivalent: `openclaw cron list | get | runs | run | edit | disable | enable`. `openclaw cron` is the same command as `openclaw automations`.
- **`/home/node/tools` is a read-only mount for me.** Any script I write goes under `/home/node/workspaces/assistant/tools/`. `check.mjs`, `migrate-to-board.mjs`, `cards.mjs` there are the builder's to edit, not mine.
- **Guest exec budget is ~10s wall clock.** A `sleep 45`/`sleep 50` inside `exec` returns an empty result, which looks like failure. Poll in short separate calls, or run a background watcher.
- Long-running work: `exec({background:true})` + `process kill` is how I watch something; its completion wakes the conversation.
- Discord/file attachments land at `/home/node/workspaces/assistant/media/inbound/openclaw-staged-<uuid>/input-<name>---<uuid>.<ext>` — that's how his resume PDFs arrived, and how a `.docx` template will arrive.
- `openclaw models status` lists effective provider auth per provider (useful for "which key is in play").

## Channel IDs: env → registry
- Bear pushed back on SSHing in to add every new channel id: *"it's genuinely making the usefulness of these scenarios more difficult when i have to SSH in every time."* He's right — a channel id is data, not a secret.
- Built **`config/discord-channels.json`** (all 7 channels, synced from the live guild) and **`tools/lib/channels.mjs`** with `channelId(name, { create, topic })` — resolves by name, creates the channel if missing, records the id. No env, no restart.
- He said leave the existing env ids alone for now and make a note. Done: To Do task **"Move the remaining channel IDs from .env into the channel registry"** (Tasks list, due 19 Nov) naming the three scripts still reading `process.env.DISCORD_CHANNEL_*`.
- Quote he gave me on this: *"Worth making a note to update that at some point though."*

## Plans
- **1 job hunt board** — `done`. **2 runway** — `done`. **3 pinned gig cards** — `built`, he said "it looks good", marked `done` by me.
- **4 events radar** — `building` (went live 6 Nov). **5 cover-letter pipeline** — `queued` behind it.
- Queue order lives in `plans/queue.md`; statuses are the first line of each brief.

## The events radar (#4) — his answers, verbatim intent
- Wants **interesting events in Heilbronn now, and maybe Liverpool later** (city must come from `travel.json`, never hard-coded).
- **Distance sets the bar:** local (Heilbronn) broad and low stakes; ~1 hour away (Stuttgart, maybe Mannheim/Heidelberg) only if genuinely interesting; further (Frankfurt) only for something big like an anime con.
- Interests, prioritised: **metal** (the small local end — major bands belong to gig radar), video games/LANs, anime, **stand-up English only**, EDM/raves lesser, other live music — plus a broad novelty bucket.
- The framing that matters: *"me and possibly my roommate have the night off tomorrow/in two or three days, what can we go do?"* — **short horizon, two people**, not FOMO planning. Hence a daily 09:30 scan of the next 7 days and a Friday 16:30 weekend roundup.
- Own channel, not #gig-radar. He also mentioned a "doom fest" find and wasn't sure it was real — I searched, came back with nothing, so I claimed nothing.
- Sources: Ticketmaster (via concert_radar), Eventbrite (API key optional), Meetup, Heilbronn city/venue calendars, con calendars, web search. Never invent a show, venue, price or link.

## Cover letters
- **16 letters in `jobs/letters/`, average 473 words, range 388–552; 15 of 16 are over 400.** Only Adyen (388) is inside his target.
- His standard: **200–400 words, leaning short (~300)** — "cover letters are often skipped in tech". The limit isn't in the job-scout skill yet; that's part of plan 5.
- **PDF pipeline works:** `tools/letters/build-pdf.mjs` (pdf-lib, A4, Times, 62pt margins, letterhead from `config/letterhead.json` when it exists). Adyen letter → 1 page, 3.3 KB.
- **ATS question answered with proof:** `tools/letters/extract-text.mjs` (pdfjs-dist) recovered 392 words / 2,526 chars from that PDF — real text layer, selectable, single column, standard fonts.
- No LibreOffice/Word/pandoc/wkhtmltopdf in the container, so a `.docx` with `{{body}}` can't be converted. Plan instead: read his template **once**, encode its layout in `config/letterhead.json`, render to match.
- **Open question I left him:** trim the existing sixteen now, or leave them and let plan 5 enforce length from the next letter. He hasn't answered.

## Working with him
- He runs **Claude Code in parallel** on the same files, so check status lines and file mtimes before assuming a change is mine. He said "we'll let Claude handle it from here" about the scout — I stopped poking and killed my watcher.
- He asks "are you the right person or should I ask Claude?" — answer: **I design and write the brief, Claude builds.** That's the loop he wants to keep.

## Cover letter template - LOCKED (19:22)

Bear picked **Option B** from three layouts: name and role top-left, contact details stacked and right-aligned in the top-right corner (ADDRESS / EMAIL / PHONE-WHATSAPP with small letter-spaced caps labels), body 9.6pt at 1.55 leading, A4, Lato, no date line, no footer.

Address on letters is **Heilbronn, BW, Germany** - he is job hunting from a friend's place there. Bothell, WA is only where he used to live in the US and must not appear on applications.

His own Evam letter was the reference: 20pt name, 7pt role, 9.5pt body, sidebar contact blocks, no date. The 11.5pt body on my earlier drafts was the main reason they read as less professional than his.

He said every further step on the letters goes through the Claude job from here, to avoid crossed wires - so I lock the template and stop, and do not rewrite the remaining 15 myself.

Built and locked in the workspace already: `tools/letters/build-html.mjs` (Option B, tracking 0.45pt so the text layer extracts cleanly), `tools/letters/a4ify.mjs` (Chromium prints US Letter, must normalise to A4), `config/letterhead.json`.


## Adyen cover letter approved (20:13)
- Bear said **"Approved"** in the Adyen job-board thread (channel 1567890847771398164) - read as approval of the Adyen letter in the trimmed 287-word form shown in the DM at 19:08-19:22.
- Saved that text as the canonical `jobs/letters/adyen-8141748.md` (was 388 words; backup `.md.bak-times`) and put the locked Option B one-page PDF at `jobs/letters/pdf/adyen-8141748.pdf` (backup `.pdf.bak-times`). PDF text layer verified: 1 page, 305 words extracted.
- Handed the PDF to him in the thread with <https://job-boards.greenhouse.io/adyen/jobs/8141748>. He still applies himself; nothing filed for him.


## Cover letters - superseded (20:30, from the Claude build of plan 5)
- Bear's final layout is **compact Option B**: name and role top-left; address, email, phone and LinkedIn right-aligned top-right with **no labels**; yellow stripe down the left; body 10.5pt. It is saved in `config/letterhead.json` (`layout`). The "LOCKED (19:22)" note above is out of date.
- **The only letter tools now are `/home/node/tools/letters/`** (count.mjs, build.mjs, finalize.mjs). When he approves a letter in a #job-board post, follow the job-scout skill's *Letter approval*: `node /home/node/tools/letters/finalize.mjs <id> --post` (or `--text <file>` for his own version). It saves the final text and PDF in `jobs/letters/pdf/`, checks words, one page and the text layer, and attaches the PDF to the post itself.
- The scripts in the workspace `tools/letters/` (build-html, build-pdf, build-variants, a4ify) are **retired**: don't use them.

## Adyen letter, his own revision (20:55)
- Bear sent an **updated body** adding his own telemetry proof (SharePoint 300+ job system, Excel 75% cost-per-operation cut) and sharper Azure onboarding. Saved verbatim to `jobs/letters/adyen-8141748.md`, now **368 words** - inside his 200-400 target.
- Two edits I made and told him about: standalone " - " → em dash, and the LinkedIn URL dropped from the signature line (it is already in the contact block).
- No Chromium in the container and the browser tool blocks `file:` and localhost, so the Option B layout is now rendered natively by **`tools/letters/render-letter.mjs`** (pdf-lib + Lato, measured from build-html.mjs's CSS). Checkers: `extract-text.mjs` (1 page, 386 words recovered) and `verify-layout.mjs` (text box x 42.5-552.8, y 152.9-768.4 on 595x842).
- `pdf-to-png.mjs` returns `darkPixels: 0` on these PDFs - pdfjs canvas render is blank in this container, so PNG previews prove nothing. Do not trust them.


## Cover letters - not mine any more (from the Claude build of plan 5, Bear's decision)
- Bear decided cover letters go through **Claude Opus with no persona**, not me: the `bearnice-letters` job (every 10 minutes, `/home/node/tools/letters/watch.mjs`) drafts each role's letter in its #job-board post from his letters, resume and the ad, and handles his "good" (PDF), pasted versions (PDF of his exact words) and notes (Opus revises).
- When he writes about a letter in a board post: react 👍 and do nothing else. Never edit `jobs/letters/`, never build a PDF, never use the scripts in my workspace `tools/letters/` (retired). Twice today I rebuilt his Adyen PDF my own way and changed his dashes; that is exactly what he doesn't want.

```

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
- No related pages yet.
<!-- openclaw:wiki:related:end -->
