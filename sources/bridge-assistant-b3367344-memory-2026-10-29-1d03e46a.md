---
pageType: source
id: source.bridge.assistant-b3367344.memory-2026-10-29-1d03e46a
title: "Memory Bridge (assistant): 2026-10-29"
sourceType: memory-bridge
sourcePath: /home/node/workspaces/assistant/memory/2026-10-29.md
bridgeRelativePath: memory/2026-10-29.md
bridgeWorkspaceDir: /home/node/workspaces/assistant
bridgeAgentIds:
  - assistant
status: active
updatedAt: 2026-10-29T12:45:52.879Z
---

# Memory Bridge (assistant): 2026-10-29

## Bridge Source
- Workspace: `/home/node/workspaces/assistant`
- Relative path: `memory/2026-10-29.md`
- Kind: `markdown`
- Agents: assistant
- Updated: 2026-10-29T12:45:52.879Z

## Content
```markdown
# 29 Oct 2026 — notes

## Lube / gesund.de
- gesund.de does **not** show live stock for pjur: every pharmacy in the list showed "Verfügbarkeit nicht bekannt" for PZN 00791148, so the pharmacy has to be called. Product page: https://www.gesund.de/produkt/400429-pjur-aqua-gel (also `#instruction-for-use`, leaflet PDF at https://productservice.gesund.de/api/product/beipackpdf/400429).
- Cheapest verified 100 ml that day: DocMorris €8.95; Shop Apotheke 100 ml €17.65 (30 ml €6.95).
- Silicone finding: pjur ORIGINAL and Sliquid Silver have **identical INCI** (Cyclopentasiloxane, Dimethicone, Dimethiconol); Eros Classic is the same family with cyclomethicone. Difference is viscosity only. **pjur LIGHT** is pjur's thinnest silicone ("unser dünnflüssigstes Gleitgel auf Silikonbasis"), from ~€22.95 — the one that matches his "liquid, not gel" taste.

## Fun Store Heilbronn (verified from their own site)
- Fun Store, Etzelstrasse 38, 74076 Heilbronn, +49 7131 645 35 79, Mo–Sa 10–20, central mail office@fun-store.de, site fun-store.de. Seven stores incl. Heilbronn.
- Wolt delivery only for Berlin and Saarbrücken — Heilbronn is walk-in only, no click & collect.
- Address sits in Industriegebiet Neckar (74076), commercial estate; building also has a dm and a bakery. Two Etzelstraße segments in Heilbronn — that's why maps confuse him. Coordinates ≈ 49.1538, 9.2168.
- **I cannot make phone calls.** No telephony at all; I over-offered once and corrected it. Email is the only outbound route.
- Two emails sent 29 Oct (~12:29 and a same-thread addendum ~12:31) asking about pjur AQUA 100 ml, Sliquid H2O, pjur LIGHT, pjur ORIGINAL, Sliquid Silver. Unsigned, no hold requested (his explicit instruction). **No reply as of 13:30.**

## Inbox monitoring (how it actually works)
- Timer-based, not event-based: `heartbeat-assistant` runs every 2 h (7200000 ms) and can be skipped with "requests-in-flight" if we're mid-conversation. No AgentMail webhook subscribed.
- State tracked in `heartbeat-state.json` (seen message IDs, last check, To-Do/travel notes).
- On-demand checks any time he asks — that's the fast path. He's happy with the 2 h poll.

## Website: "Germany cheat sheet" (new, standing project)
- Hub: /research/germany-cheat-sheet/ with subpages **lube** and **soap**. Concept: local German alternatives to the products he normally buys, more pages later (deodorant, shampoo, coffee…).
- Lube page moved from /research/lube-astroglide-alternatives/ → /research/germany-cheat-sheet/lube/; old URL is now a meta-refresh stub so his earlier links still land. pages.json updated (hub = kind index, soap = research).
- Added CSS to assets/site.css: `.pick-fig` (inline product photo floated right, stacks on ≤640px) and `.thumb` (54px table thumbnails). Product shots live in each page's `img/` folder.
- Container has **no PIL and no pip/ImageMagick**, so images are uncompressed (~2.8 MB on the lube gallery). Mentioned the weight to him.

## Body wash research
- He uses The Body Shop **Satsuma**; likes mandarin/satsuma and hotel eucalyptus. Hard filters: **no mint, no grapefruit, nothing with scrub bits** (bits catch in body hair).
- Verified: **Kneipp Aroma-Pflegedusche Glücksgefühl** — Mandarine & Bio-Sonnenblume, 200 ml, €3.29 (kneipp.com). Recommended first pick.
- Rossmann: **Isana Cremedusche Warm Mandarine** exists (4.3/5, 25 reviews) — creamier texture, cheap test. **Weleda Mandarino di Capri Shower Gel** listed at Rossmann.
- The Body Shop Heilbronn listed at Deutschhofstraße 19 — **unconfirmed** whether still trading (chain has closed German stores).
- **Correction I had to make:** tetesept has **no eucalyptus shower gel** at all (their eucalyptus is bath concentrate/salts/warming balm), so my earlier "Muskel-Entspannung Pflegedusche" recommendation was wrong and was removed from the page. tetesept Erkältungszeit shower gel does exist but contains *Mentha arvensis*. Net: mint-free eucalyptus body wash is not mainstream German drugstore stock — check natural-cosmetics shelves, pharmacies (Australian Bodycare), or sauna lines.
- Could not get an Isana photo: Rossmann blocks automated fetching; Kaufland/eBay 403; image search returns junk. Offered that he sends a shelf photo.

## Misc
- His current address (Bismarckstr 107, Heilbronn) is in keep.json as an address entry, source "bear"; the /links/ shelf shows it under the "you asked" filter.
- keep.json holds Layla's dashboard entry with `private: true`; the shelf deliberately shows it redacted. Tool output masks the token — read it by length/hash, not by echoing.

```

## Notes
<!-- openclaw:human:start -->
<!-- openclaw:human:end -->

## Related
<!-- openclaw:wiki:related:start -->
- No related pages yet.
<!-- openclaw:wiki:related:end -->
