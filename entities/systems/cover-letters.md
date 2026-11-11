---
pageType: entity
entityType: system
id: entity.systems.cover-letters
title: "System: cover letters"
aliases:
  - letters
  - letters job
  - letters watch
sourceIds:
  - repo:tools/letters
  - repo:tools/taps
updatedAt: 2026-11-11T14:00:00.000Z
---

# System: cover letters

Writes a cover letter for a role on the [[entities/systems/job-board|job board]] when Bear asks for one, and turns the version he approves into a one-page PDF on his letterhead. No assistant persona: Claude Opus 5.5 writes from his own letters, resume, notes and the ad; DeepSeek gives it a light reword; code checks it. Bearnice stays out of it. Hub: [[entities/systems/systems|Bearnice's systems]].

## How it runs

1. Each board post without a letter gets a ✍️. Bear tapping it asks for a letter (untapped posts cost nothing).
2. `draft.mjs` / `opus.mjs`: the ad comes from the board's public API (Greenhouse, Ashby, Lever: `ad.mjs`), else Opus reads it with `web_fetch`; no readable ad, no letter. Opus (adaptive thinking, effort high) answers with working notes and the letter.
3. Checks (`letter.mjs`, `count.mjs`): 200-400 words for any PDF, the writer held to 200-250 (300 at most), no em dashes. A failing draft goes back to Opus up to twice. Then `reword.mjs` (DeepSeek over OpenRouter) rewords the paragraphs; the reword is kept only if it still passes.
4. The draft is posted in the role's post with its word count. His replies there, in order: "good"/"approved" makes the PDF (`finalize.mjs`, `build.mjs`: A4, one page, text layer read back word for word) and attaches it; a pasted letter ("Dear ...", 120+ words) becomes the PDF verbatim; any other note goes to Opus to revise, or is ignored if it isn't about the letter.

## Files

- Repo `tools/letters/`: `watch.mjs` (the handler; exports `tick`), `draft.mjs`, `opus.mjs` (the writer's prompt), `reword.mjs`, `ad.mjs`, `letter.mjs`, `count.mjs`, `build.mjs`, `finalize.mjs` (the only way a letter becomes a PDF), `seen.mjs`, `spend.mjs`, `fonts/`. Tests in `tools/letters/test/`.
- Repo `jobs/bearnice-taps.yaml` (the `letters` handler). Her skill `job-scout` tells her to react 👍 to letter talk and do nothing else.
- Her workspace: `jobs/letters/<id>.md` (draft; `.opus.md` before the reword, `.notes.md` the writer's notes), `jobs/letters/pdf/` (agreed `.final.md` and `.pdf`), `jobs/letters/style.md` (Bear's standing notes), `jobs/letters/spend.jsonl` (cost per operation), `jobs/sources/cover-letters.md` (his letters below the first `---`), `jobs/sources/resume-2026-swe.pdf` / `-tpm.pdf` (picked by the listing's `resume`), `jobs/sources/me/about-me.json` (his notes from `/me/`, written by `bearnice-site-api`), `jobs/interview-stories.md`, `config/letterhead.json` (layout he picked on 6 Nov).
- `seen.json` fields: `letter`, `letter_state` (draft or final), `letter_words`, `letter_drafted_on`, `letter_writer`, `letter_cursor`, `letter_final`, `letter_pdf`, `letter_pdf_on`, `letter_pdf_message_id`, `letter_failed_on`.
- Env: `ANTHROPIC_API_KEY`, `LETTERS_MODEL` (default `claude-opus-5-5`), `OPENROUTER_API_KEY`, `LETTERS_REWORD_MODEL`, `LETTERS_REWORD=off` (skip the reword), `LETTERHEAD`, `LETTERS_WORKSPACE`, `LETTERS_SPEND_LOG`, `JOB_BOARD_SEEN`, `JOB_BOARD_OWNER`.

## Jobs

| Job | Schedule | Model | Posts to |
|---|---|---|---|
| `bearnice-taps`, handler `letters` (last in the pass) | every 10 min; no time limit, lock taken over after 180 min | Opus 5.5 (writer), DeepSeek (reword); at most 4 Opus calls a pass | #job-board posts |

## How to check it

- `agents/assistant/taps/state.json`: `handlers.letters.failures` 0, `last_ok` recent. A long draft makes the next pass skip only this handler.
- `docker exec openclaw-gateway node /home/node/tools/letters/watch.mjs --dry`: what it would do, no model, no changes.
- Spend: `docker exec openclaw-gateway node /home/node/tools/letters/spend.mjs`, and the Spend section of `/ops/`. The log was still empty on 11 Nov (no letter since logging began on 10 Nov).
- On 11 Nov: 18 listings had a letter (11 `draft`, 6 `final`, 1 with no `letter_state`); 17 files in `jobs/letters/pdf/`.

## Gotchas

- A failed draft writes `letter_failed_on` and retries the next day, and posts "I'll try again tomorrow" even when that post itself failed (R4).
- On first sight of a post with a letter, the cursor starts at its newest message: history is never replayed.
- If Bearnice edits `jobs/letters/` or answers letter talk herself, she fights the job; the forum's mention-only rule exists for that reason.
- Changing the writer's prompt (`opus.mjs`) changes every future letter: show Bear a draft before deploying.

## Change log

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
- [[entities/systems/job-board|System: job board]]
- [[entities/systems/taps|System: taps dispatcher]]
- [[entities/systems/website|System: website]]
<!-- openclaw:wiki:related:end -->
