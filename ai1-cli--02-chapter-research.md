# Chapter 2 — Chapter Research

You signed a table of contents. Sixteen chapters now exist as titles and one-line promises — and nothing else. The amateur move, again, is to start writing. The system's move is Phase 1 of the spine: **research every chapter before drafting any chapter**, so that when prose finally happens it comes from gathered evidence, not from the model's memory of what copywriting books usually say.

## Overview: why research is a chapter-shaped job

A chapter that gets drafted without research inherits whatever the model believes — fluent, plausible, and unverifiable. The research pass replaces that with a **notes file per chapter**: the concepts the chapter must convey, real cases, prerequisites and forward connections, what's settled versus contested, where students get stuck, and — most valuable of all — **flags**: the claims that must be verified before drafting, and the gaps nobody could fill. The notes are the contract between the plan and the prose. Your job at the end of this chapter is to *check them* — you are the editor of the research, not its typist.

The work runs on a reusable recipe — the **Chapter Research Gatherer** (the full prompt sits in this book's appendices). Its shape: read the book's chapter list; scan a shared library of existing material and copy anything relevant into the pantry; then, for each chapter in order, gather conceptual foundations, cases, connections, state of the field, and teaching considerations into one structured notes file; index everything; report the flags. Two of its behavioral rules carry the whole philosophy: **never write chapter prose** (gathering and drafting are different jobs, gated separately), and **never fabricate sources** — a gap honestly flagged beats a citation invented.

## The exercise

1. **Point the gatherer at your chapter list.** The recipe reads `TIKTOC.md` by convention; your signed Blueprint is the real source of truth — for a converted book like ours, the Blueprint's TOC supersedes the old book's. (The old `TIKTOC.md` is still worth reading once: it tells you what the *source* book thought it was.)
2. **Scan the library first.** Research you already own beats research you fetch. Anything relevant gets copied with a `_lib_` prefix — provenance visible in the filename.
3. **Run the research, chapter by chapter, in order** — connections to adjacent chapters only become visible if earlier chapters are researched first.
4. **Read the notes files.** For each: do the concepts match what the Blueprint promised? Are the flags honest? Is anything load-bearing resting on `[K]` — model knowledge — that needs a source?
5. **Triage the flags.** Some block drafting (verify-first chapters), some are gaps to fill later, some are decisions only you can make. Write your dispositions down.

**Artifacts:** one `NN-slug_notes.md` per chapter in `pantry/`, the `_lib_` copies, an updated pantry index, and your flag triage.

## What can go wrong

| Symptom | Cause | Fix |
|---|---|---|
| Notes read like chapter drafts | the gatherer wrote prose | send it back; gathering ≠ drafting — the gate between them exists for a reason |
| Every claim has a confident citation | too good — probably fabricated | spot-check three; the rule is *flag, don't invent* |
| Notes for new chapters look as full as converted ones | suspicious — new chapters have no source material | new-chapter notes should carry more flags, not fewer |
| No flags at all | the research is hiding its gaps | a notes file with an empty section G has not been finished |

## Bridge

The pantry now holds everything a drafting pass needs — and a flag list that says exactly what must be verified first. Before any drafting, though, the book you're remaking should build under your name and live under governance. That's next.

---

## Bear's Copywriting Book

### The research run — 16 chapters, summarized

We ran the gatherer against the signed Blueprint (14 core + 2 modules). The full research sits in the pantry — `01-the-brief_notes.md` through `14-ai-operations_notes.md`, plus `m1-` and `m2-` for the modules, indexed in `pantry/README.md`. Every claim in every file carries a provenance mark: **[S]** traces to a source chapter of the writing guide, **[R]** to the triangulated Chapter-1 research (with its ✓✓/✓/⚠ grades carried forward), **[L]** to a library file, **[K]** to model knowledge — *verify before manuscript*. The agent's web access was limited this run, and the notes say so rather than faking it.

**The library scan paid for itself.** Twelve files copied from the shared MD library (301 scanned), including exactly the references several chapters were missing: Cialdini's *Influence* (chapters 6–8, 11), *Thinking, Fast and Slow* (8, 9, 13), *Co-Intelligence* and the prompt-architecture template (1, 14), *The Lean Startup* (5), *Groundswell* (3, 12), and *The $100 Startup* (the modules). One bonus discovery: the pantry already held per-chapter figure notes (`-cajal` files) for all 24 source chapters — every notes file now cross-references its figure material.

### What the research found

The process above is reusable for any book; what follows is what it actually turned up for *this* one. (The full notes — one file per chapter, every claim provenance-marked — sit in `pantry/`; this is the narrative digest.)

**The conversion thesis held, and more literally than expected.** The signed Blueprint bet that a composition textbook could be *remapped* rather than torn down, and the research confirmed it chapter by chapter. The strongest finding of the whole pass: some source chapters survive nearly verbatim *in craft*. The proposal chapter's spine — define the problem narrowly enough to be solvable, then argue the fix section by section, each section with a job — **is** the sales page; nothing needed inventing, only re-aiming. The old chapter 17's hard-won lesson (single-subject research: what one case can honestly claim about the many, and the ethics of studying a person) **is** commercial case-study discipline — the same scoped-claim restraint, now with a customer's metrics attached. The reasoning-strategies chapter converts with an inversion that makes it *more* valuable: in an AI workflow, analogy and causal reasoning aren't just persuasion tools, they're the audit checklist for fluent nonsense, because a model will hand you a persuasive-sounding analogy whether or not its load-bearing similarity holds.

**The human moat is real and specific.** Both research legs independently converged on the same center of gravity: the work AI structurally cannot do is *primary* — conducting interviews, catching the unguarded phrase, hearing how customers actually talk. The verified Beachway case makes it concrete enough to open the VoC chapter: an agency's polished line ("Your Addiction Ends Here") lost to a sentence mined verbatim from an Amazon book review — *"If you think you need rehab, you do."* — by more than 400% on clicks. The mined sentence won because nobody wrote it; somebody *said* it. That single case carries the chapter's whole argument, and its caveat (review mining is a VoC technique, not a survey technique) is itself teachable.

**The brief earns its place as chapter one.** The research supplied both halves of the argument: field studies showing AI lifts speed and average quality while the gains reverse outside the model's competence — which is exactly where judgment lives — and the Pepsi/Tropicana pair showing the two ways briefing fails: the 2017 ad produced from a "do something interesting" drive-by directive (no brief at all, confirmed by investigative reporting), and the 2009 redesign whose brief existed but was built on a missed insight — customer passion that, in the president's on-record words, "wasn't something that came out in the research." Different failures, different fixes, one lesson: copy inherits its brief's defects.

**Voice turned out to have public, teachable artifacts.** The research expected to need a permissioned example from Bear's network; instead the collection pass found the Mailchimp style guide — Creative-Commons licensed, so the book can *reproduce* it — which does the rare thing of converting personality adjectives into grammatical rules (scan for passive constructions; pair jargon with plain language; the mascot never speaks). Paired with Monzo's 2025 system, it shows an evolution worth a section in itself: brand voice as editorial *guide* becoming brand voice as product *writing system* — buttons, error messages, apologies.

**The new chapters found their footing — three of four.** Positioning (ch 5) now rests on verified Dunford: positioning as context-setting, not a tagline exercise — the frame of reference determines whether customers understand the product *at all* — with the five-plus-one components and the deliberately contrarian rules (never write a positioning statement; don't start with features). Awareness (ch 8) has its confirmed Schwartz spectrum, unaware through deal-ready, and an honest sentence where its second framework's history should be: AIDA gets attributed to everyone and no one. Governance (ch 14) found its stable core in role separation — machines verify conformance, humans sign — with the regulatory specifics deliberately *not* pinned to rule numbers, because rules update; the chapter teaches the checking habit instead, which is also the book's own fact-discipline applied to law. Only measurement (ch 13) still stands on thin external ground, and the research's conclusion there is itself a finding: the circulating benchmark tables are vendor folklore, so the chapter should teach readers to measure against their own baselines — a more honest lesson than any borrowed number.

**Where the evidence is thin, the book will say so.** The second research leg graded its own claims, and the grades transfer: strong for research-ops, proof, distribution, and governance; moderate for voice and message mining (much of it adjacent UX evidence rather than direct copywriting studies); thin for the autobiographical genres — which is exactly why they became modules. A book about verified claims should model calibrated confidence about its own foundations, and the notes files now make that possible chapter by chapter.

**The flag triage — and how the human resolved it.** The blocking flags went to Bear, and the resolutions show three different *kinds* of human answer, worth distinguishing because each is a move you'll reuse:

- **Verified (evidence supplied).** Ch 8's Schwartz five-stage spectrum — unaware → problem aware → solution aware → product/you aware → most aware/deal ready — checked and confirmed; the blocking flag lifted, the chapter may draft. Ch 5's Dunford material verified in detail: positioning as context-setting (not a tagline exercise), the five-plus-one components, the 10-step process, the three styles, and the polemical claims worth teaching as positions. The notes files now carry the verified content, marked with who verified and when.
- **Resolved by policy (the citation strategy changed).** The stripped regulator links (chs 11, 14) didn't get restored — they got *outgrown*: don't cite FTC rules by number, because rules update. Name the institution and the topic; teach readers to check the current version. The policy is better than the fix it replaced, and it's self-demonstrating — regulatory material is `stable: false` with an expiry, exactly like a fact-dictionary entry.
- **Resolved by the second leg.** `flawless-consulting` arrived in the OpenAI pass's library haul; Module B's flag closed itself.
- **Resolved by honesty (a fourth kind).** AIDA's origin didn't verify cleanly — so the manuscript will say exactly that: *"AIDA is one of those frameworks that gets attributed to everyone and no one; its exact origin is uncertain."* Stating uncertainty beats inventing a lineage, and the sentence itself models the book's citation discipline.
- **Resolved by collection.** The three real-world gaps closed with a verified case dossier (`pantry/verified-cases.md`): Pepsi 2017 + Tropicana 2009 for the brief chapter (the brief never written vs. the brief wrongly grounded); the Beachway VoC test for chapter 4 (a mined Amazon-review line — *"If you think you need rehab, you do."* — beating agency copy by +400% on CTA clicks, with the definitional caveat taught explicitly); Mailchimp (CC-licensed, reproducible) + Monzo for the voice chapter — the evolution from voice guide to product writing system. Every case carries its caveat in the record.
- **Still open:** Ries & Trout (verify or rely on Dunford alone); ch 7's objections taxonomy (source it or own it); the Starbucks voice URL (verify accessibility before citing).

**The pattern worth noticing:** the new chapters carry the most flags, and that's the system working — they have no source-chapter safety net, so their notes are honest about standing on thinner ground. A research pass where everything looks equally solid is a research pass that's lying about something.

### The second leg — and what integrating it changed

While the CLI agent researched the *copywriting book's* chapters, Bear ran the same gatherer through a second system (OpenAI deep research) — which, given the same recipe, made a different and equally valid choice: it researched **this book's** chapters. Thirteen notes files landed in the AI1 pantry, one per chapter of the amended outline (it read the outline's logged amendment and applied the renumbering on its own), each grounded in *local* sources — the conductor's research-phase prompt, the appendices — plus fourteen library files including the full texts of several books the first pass had only sampled.

Integration was mostly confirmation, which is what you want: the two passes overlap without contradicting. Three concrete changes came out of it: `flawless-consulting` — which the copywriting Module B had flagged as a Bear-decision — arrived in the second pass's library haul and is now copied across, the flag resolved; the second pass's pantry index doubles as an independent **coverage check** (it verified every outline chapter has notes — the kind of audit one system shouldn't run on itself); and its flags are characteristically *operational* where ours were *evidentiary* ("verify the exact clone command," "verify current Medhavy export expectations before writing instructions") — two systems, two failure-mode vocabularies, both worth having. The lesson from Chapter 1 repeats at scale: the value of the second leg isn't redundancy, it's that different systems are suspicious of different things.
