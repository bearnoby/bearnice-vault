---
pageType: entity
entityType: system
id: entity.systems.cover-letters
title: "System: cover letters"
aliases:
  - letters
  - letters job
  - letters watch
  - letters writer
sourceIds:
  - repo:tools/letters
  - repo:tools/taps
  - repo:scripts/letters-writer.py
updatedAt: 2026-11-23T21:00:00.000Z
---

# System: cover letters

Writes a cover letter for a role on the [[entities/systems/job-board|job board]] when Bear asks for one, and turns the version he approves into a one-page PDF on his letterhead. Since 13 Nov 2026 the writer gets his whole work history (his resume and every Microsoft Connect) and the full job ad, under a short request written the way he would ask for it, and nothing else: no rulebook, no old letters, no style notes. The writer is Claude Fable through Claude Code on the server (his subscription) or Gemini 4 Carbon over OpenRouter, switched by one setting; DeepSeek gives it a light reword; code checks it. Bearnice stays out of the writing. Hub: [[entities/systems/systems|Bearnice's systems]].

## How it runs

1. Each board post without a letter gets a ✍️. Bear tapping it asks for a letter (untapped posts cost nothing).
2. The ad (`ad.mjs`): text he pasted for the role wins, else the board's public API (Greenhouse, Ashby, Lever, SmartRecruiters), a company careers page showing a Greenhouse job (`?gh_jid=`), or a page's JSON-LD job posting. No model reads a web page. **No ad, no letter**: the post asks him to paste the ad (a message or a text file), the listing gets `letter_needs_ad`, and his next long message there (text, or a message.txt) is saved as `jobs/letters/<id>.ad.txt` and the letter follows on that pass. The same goes for a later request that needs the ad (application answers, a recruiter email): the writer answers NO_AD, the post asks for the ad, the request waits in `letter_ad_request`, and it runs by itself once he pastes the ad, even on a role that already has a letter.
3. The writer (`writer.mjs`) gets `<resume>` and `<connects>` as text (`history.mjs`), the request, the role facts and the ad. Visa line: "I would need visa sponsorship to work in <country>"; the green card only for a US-only role (Bear: the goal is to leave the US). Roles in Germany get a no-contractions tone line.
   - `fable`: the gateway has no Claude Code, so `cli-bridge.mjs` writes `jobs/letters/queue/<id>.request.json`; the host unit `bearnoby-letters-writer.path` starts `scripts/letters-writer.py`, which runs `claude -p --model fable` with no tools, no MCP, no settings, in an empty folder, and writes `<id>.response.json`. Waits up to 12 minutes. A draft takes 15-35 s.
   - `carbon`: OpenRouter, `google/gemini-4-carbon`, about $0.15 a draft.
4. Checks (`draft.mjs check`, `letter.mjs`): letter format with the caps role line, 200-400 words, no em dashes, no machine-sounding phrase. Nothing about what the letter says. A failing draft goes back up to twice. Then `reword.mjs` (DeepSeek) rewords the paragraphs; kept only if it still passes.
5. The draft is posted in the role's post with its notes (where each fact came from), word count and "Written by Fable." (or Gemini Carbon). His replies, in order: "good"/"approved" makes the PDF (`finalize.mjs`, `build.mjs`); a pasted letter ("Dear ...", 120+ words) becomes the PDF verbatim; any other note goes to the writer to revise, or to write application answers, or is ignored, and an ignored message gets 👀 so it never looks lost. A text file on his message (Discord turns a paste over 2,000 characters into message.txt) is read as part of it.

## Files

- Repo `tools/letters/`: `watch.mjs` (the handler; exports `tick`), `draft.mjs`, `writer.mjs` (the request, both writers, `WRITERS`), `history.mjs` (resume and Connects as text), `cli-bridge.mjs` (the queue), `ad.mjs`, `reword.mjs`, `letter.mjs`, `count.mjs`, `build.mjs`, `finalize.mjs` (the only way a letter becomes a PDF), `seen.mjs`, `spend.mjs`, `fonts/`. Tests in `tools/letters/test/`.
- Repo host side: `scripts/letters-writer.py`, `scripts/systemd/bearnoby-letters-writer.path` and `.service` (installed and enabled by `deploy.sh`).
- Repo `jobs/bearnice-taps.yaml` (the `letters` handler). Her skill `job-scout` says letters aren't hers, except switching the writer when Bear asks.
- Her workspace: `config/letters.json` (`{"writer": "fable"}` or `"carbon"`; read on every call), `jobs/letters/<id>.md` (draft; `.orig.md` before the reword, `.opus.md` on letters from before 14 Nov; `.notes.md` the writer's notes; `.ad.txt` an ad he pasted), `jobs/letters/queue/` (requests in flight, normally empty), `jobs/letters/pdf/`, `jobs/letters/spend.jsonl`, `jobs/sources/connects/*.pdf` (15 Connects, 2017-10 to 2024-12, named by date), `jobs/sources/resume-2026-swe.pdf` / `-tpm.pdf` (picked by the listing's `resume`), `config/letterhead.json`.
- No longer read by the writer (kept): `jobs/sources/cover-letters.md`, `jobs/letters/style.md`, `jobs/interview-stories.md`, `jobs/sources/me/about-me.json` (the `/me/` page).
- `seen.json` fields: `letter`, `letter_state` (draft or final), `letter_words`, `letter_drafted_on`, `letter_writer` (fable, carbon, opus or bear), `letter_cursor`, `letter_final`, `letter_pdf`, `letter_pdf_on`, `letter_pdf_message_id`, `letter_failed_on`, `letter_fail_notice`, `letter_needs_ad`, `letter_ad_request` (`{id, text}`: his request waiting for the ad).
- Env: `OPENROUTER_API_KEY` (Carbon and the reword), `LETTERS_WRITER` (overrides the setting), `LETTERS_CARBON_MODEL`, `LETTERS_REWORD_MODEL`, `LETTERS_REWORD=off`, `LETTERS_QUEUE`, `LETTERS_CLI_WAIT_MS`, `LETTERHEAD`, `LETTERS_WORKSPACE`, `LETTERS_SPEND_LOG`, `JOB_BOARD_SEEN`, `JOB_BOARD_OWNER`. Host: `CLAUDE_BIN` (default `/root/.local/bin/claude`, signed in to Bear's subscription).

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-taps`, handler `letters` (last in the pass) | every 10 min; no time limit, lock taken over after 180 min | Fable via Claude Code (default) or Gemini 4 Carbon; DeepSeek reword; at most 4 writer calls a pass | #job-board posts |
| `bearnoby-letters-writer.path` (host systemd) | when a request file appears | Fable via `claude -p` | nothing (writes the response file) |

## How to check it

- `agents/assistant/taps/state.json`: `handlers.letters.failures` 0, `last_ok` recent.
- `journalctl -u bearnoby-letters-writer.service -n 20`: one line per request, `fable ok in 20s` or the CLI's error. `systemctl is-active bearnoby-letters-writer.path` is `active`. `jobs/letters/queue/` is empty between passes.
- Try a draft without posting: `docker exec -w /home/node/workspaces/assistant openclaw-gateway node /home/node/tools/letters/draft.mjs <listing-id> --out /tmp/x.md` (add `-e LETTERS_WRITER=carbon` to try the other writer).
- `docker exec openclaw-gateway node /home/node/tools/letters/ad.mjs <url> [<listing-id>]`: the ad text, or "he would be asked to paste it".
- `docker exec -w /home/node/workspaces/assistant openclaw-gateway node /home/node/tools/letters/history.mjs`: resume and Connect word counts (15 Connects, about 33,000 words on 13 Nov).
- Spend: `docker exec openclaw-gateway node /home/node/tools/letters/spend.mjs`; rows carry `writer` and `via` (`claude-cli` costs 0). On `/ops/` the "Cover letters (OpenRouter)" row holds Carbon and the reword; Fable letters cost nothing there.
- 17 Nov: four Fable drafts and two revisions on the board since the deploy (ClickHouse, SumUp, Elastic), all `ok`; Motherduck waiting for its ad.

## Gotchas

- Fable runs on Bear's Claude subscription: about 76,000 tokens a draft or revision. If Claude Code is signed out or out of quota, the post says the writer didn't answer and the draft is retried the next day (`letter_failed_on`); there is no silent fall-back to Carbon.
- A Connect or resume dropped into `jobs/sources/` is read by the next pass; there is no cache to clear. Mid-2022 has no Connect.
- The prompt is the whole design: changing the request in `writer.mjs` changes every letter. Bear compared Fable, Opus and Gemini (Hydrogen, Carbon, Argon) on 13 Nov; Opus and Argon were weaker, Carbon about level with Fable.
- The reword may change the visa line's wording ("need" to "require"); Bear chose to keep it in the reword.
- A failed draft that isn't a missing ad writes `letter_failed_on` and retries the next day; a notice that couldn't be posted is kept in `letter_fail_notice` and posted by a later pass.
- On first sight of a post, the cursor starts at its newest message: history is never replayed.
- If Bearnice edits `jobs/letters/` or answers letter talk herself, she fights the job; the forum's mention-only rule exists for that reason. Her only part is `config/letters.json`.

## Change log

- 2026-11-23 build/15-discord-read-access (`544180e`, deployed 23 Nov): a request that needs the ad waits for it and runs when he pastes it (`letter_ad_request`); text attachments are part of his message; 👀 on a message the writer ignores. Fixes the IKEA post of 20 Nov, where his pasted ad was read as a note and ignored.
- 2026-11-13 build/10-cover-letter-writer-overhaul (`bf172c5`, deployed 13 Nov): whole history in (resume and 15 Connects), rulebook and old letters, style notes and `/me/` out; Fable via Claude Code on the host (queue + `bearnoby-letters-writer`) or Gemini Carbon, switched in `config/letters.json`; Anthropic API no longer used; checks cut to format, 200-400 words, dashes and tells; always the full ad, asks him to paste it; visa line "would need visa sponsorship", green card only for US-only roles.
- 2026-11-11 Note written (V3) from the repo at `c076417` and read-only server checks.
- 2026-11-10 Became the `letters` handler of `bearnice-taps`, replacing `bearnice-letters` (deployed 11 Nov, `45ce147`); cost logged to `spend.jsonl`; ad read from the board's API, no ad no letter; DeepSeek reword; Bear's `/me/` notes feed the writer; the writer sells rather than restating the CV.
- 2026-11-06 Built in three stages (word count, PDF on his letterhead, approve in the post); Opus writes them; letters on request with ✍️.

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
### Referenced By

- [[entities/systems/systems|Bearnice's systems]]
- [[concepts/letter-taste|Letter taste]]
- [[entities/systems/job-board|System: job board]]
- [[entities/systems/taps|System: taps dispatcher]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
