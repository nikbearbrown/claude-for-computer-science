# Chapter 13 — From Book to Video

You shipped the book. It is an EPUB, a Canvas course, and a Medhavy export, and every claim in it traces to a source you signed. Here is the thing almost nobody does next: a finished book is not the end of the work — it is the richest raw material you will ever have for video. The same chapter that reads well on the page can become a ten-minute narrated lecture, a one-minute explainer, a mini-biography, a promo reel, or a kids' language-learning music video. This chapter is about the system that makes those, and why it is a natural extension of everything you have already done.

That system is **Unreal Reels** — a sibling to AI+1. Where AI+1 turns an idea into a book through gated phases, Unreal Reels turns a finished book into video through gated phases. It is a separate repository with its own scripts and skills, referenced here the way Medhavy was in Chapter 12: you will meet enough to run it and know what it is, not its full internals. What matters is that it is built on the *same two ideas* you have been living for twelve chapters, so it will feel familiar the moment you open it.

## The two laws you already know, applied to time

AI+1 has a constitution; Unreal Reels has two laws, and you already believe both. The first: **audio is the master clock.** In a video, timing is not decoration you add at the end — it is a fact set at the beginning. The narration is generated and *measured* first, and every visual decision downstream (how long a clip runs, when a caption word lights up) is derived from that measured audio, never guessed. The second: **the pipeline is phase-gated, and the human signs every gate** — beats, then audio, then visuals, then render, and nothing proceeds until you approve. If that sounds like the seven-phase spine with the serial numbers filed off, that is exactly the point. It is the same discipline — the agent executes, the human signs, everything leaves a trail — pointed at video instead of prose.

There is one file underneath it all, the way `facts.json` sits under the book: `beat_sheet.json`, where **one beat = one narrated line = one thing on screen.** Learn to read it and you can read any Unreal Reels project, just as you learned to read a chapter's sidecar.

## Why your book is the reason the video is any good

You have seen what "make me a video about X" produces: fluent, generic, and confidently wrong — the video equivalent of the unchecked book-shaped text this whole book was a reaction against. It is slop for the same structural reason: nothing anchors it.

Your video will be different in one decisive way. It is made *downstream of a book you already fact-checked and signed.* When you turn a chapter into a lecture, the narration expands that chapter's own prose; the figures are that chapter's own figures; and the claims were verified at GATE 4 before a single frame existed. The video's job is not to be right about the world from scratch — it is to stay *faithful* to a source that is already right. That is a smaller, checkable task, and Unreal Reels checks it: a fidelity pass flags any narration sentence that drifts from the chapter, and — because your book fed the shared fact commons — can cross-check against it too. The book is what turns the video from a guess into a spine.

## Three forms, one engine

Unreal Reels wears several faces over one engine. Three are worth knowing now:

A **lecture** is the highest-value form for most books: a chapter becomes a narrated, captioned slide video. It runs three stages — build a pool of candidate visuals from the chapter, select them into a slide deck (you sign the plan, and the speaker-notes you approve become the narration seed), then narrate and render. The governing rule of the narration echoes your human-rewrite gate: *discuss the slide, do not read it.* The voice explains; the slide states.

An **explainer** is a short, sketch-style piece that draws one idea on as it is narrated — good for a single concept a chapter turns on. And the **reels** aspects (Songbird, Bios) make music videos, promos, and mini-biographies — which, for the copywriting book you just built, is not a detour at all: a landing-page hero video or an email teaser reel is exactly the kind of act-making copy your book now teaches.

For a first run, the free, book-anchored path is the lecture, and you can go a long way on open tools — local text-to-speech for the voice, forced alignment for captions, code-drawn figures — before any paid service enters.

## The gate that is still yours

Everything you learned about signatures carries over. In Unreal Reels the agent drafts the plan, writes the narration, generates the audio, and assembles the render — but it never signs, and it never renders the final cut on its own. You approve the plan before audio is spent; you approve the narration before it is voiced; you preview the assembled lecture and only *then* render it. A lecture built this way even carries the same kind of sign-off sidecar your chapters do, so a video becomes a first-class AI+1 artifact: gated, signed, and traceable back to the chapter and the facts beneath it.

And the payoff is the one this whole book has been about. The pipeline hands you not a finished video but a *spine* — a high-quality, accurate starting point you open and make your own. Two teachers handed the same lecture spine for the same chapter will quickly diverge into two different lectures, each unmistakably theirs. You have done this once already, with a book. Now you can do it with everything the book can become.

## The exercise: turn one chapter into video

Work with your agent. Each step produces an artifact you can open and a line in the trail.

### Step 1 — Orient the agent and choose the form

Point your agent at the Unreal Reels repository (its `AGENTS.md` is the contract, the way `AI1.md` is here) and ask it to summarize, in a few sentences, the two laws and the phase-gated pipeline — then to check which service keys you have without printing them. Now pick *one* finished, signed chapter of your copywriting book and decide, with the agent, which form fits it: a lecture (teaches a concept), an explainer (one sharp idea), or a promo reel (makes the reader act). Write one sentence on why.

**Artifact:** a short note naming the chapter, the chosen form, and the reason — saved in the repo.

### Step 2 — Build the lecture spine and preview it

Run the lecture pipeline on your chosen chapter to a *rough draft*: let it build the visual pool, generate the slide plan, and stop at the plan gate. Read the plan — especially the speaker-notes, because those become the narration. Fix what is wrong, then let it narrate (local TTS is fine) and assemble a preview. Do **not** let it render the final on its own.

**Artifact:** a previewed lecture spine (deck + narration) for one chapter, and a line in the trail recording that you reviewed the plan.

### Step 3 — Check fidelity, then sign

Run the fidelity pass on the narration against your chapter. It returns a short triage list: any spoken claim that the chapter does not support. Read every flag. For each, decide — the narration overreached (fix it) or the chapter itself is thin (a note back to your manuscript). When the list is clean, sign the lecture the way you sign a chapter, and state in one sentence what your signature warrants: that the video is faithful to a source you already verified, not that it is true from scratch.

**Artifact:** the fidelity report with every flag resolved, and a signed sidecar on the lecture — your book, now watchable, still yours.

---

*Coda still applies.* Everything in this chapter assumes a finished book, because that is what makes the video honest. If you skipped ahead and have only a draft, the video will inherit the draft's flaws — garbage in, fluent garbage out. Ship the book first. Then let it speak.
