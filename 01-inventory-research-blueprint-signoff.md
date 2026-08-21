# Chapter 1 — Inventory, Research, Blueprint, Sign-Off

Start by telling your agent to clone the book we'll be working on for the rest of this book:

```
git clone https://github.com/Humanitariansai/writing-guide.git
```

What arrives is a complete, finished textbook: **the Writing Guide**, a 24-chapter college composition and rhetoric text. Open `chapters/` and skim the titles — the literacy narrative, memoir, the profile ("telling a rich and compelling story"), proposal writing, evaluation and review, the analytical report, rhetorical analysis, position argument, the research process, the annotated bibliography, image analysis, multimodal writing, scripting for the public forum, portfolio reflection. It teaches a college student to write across the genres a first-year course demands. It builds with one command. It is fact-checked, illustrated, and done — and none of it is yours.

Here is what you and this book are going to do with it, over the next eleven chapters: **convert it into a copywriting book** — writing that exists to make a reader act. Chapters that serve a copywriter get transformed (the profile becomes a customer case study; the proposal becomes a pitch; rhetorical analysis becomes an ad teardown). Chapters that don't get cut, with logged reasons. Chapters copywriting needs that composition never taught — headlines, offers, landing pages, email, brand voice — get written. Along the way you'll add exercises and quizzes, generate flashcards, and export the result everywhere it needs to live: **Kindle** (a clean EPUB), **Canvas** (an importable course), and **Medhavy** (`.mdx`).

But none of that starts with editing. It starts with getting the **book map — the Blueprint — right.** A book is a few hundred decisions wearing a table of contents: who the reader is, what each chapter builds, what dies, what's missing, and which assessments the book owes its readers. Make those decisions mid-draft and they get made by momentum. Make them up front, against evidence, and every later edit has something to be checked against. That's this chapter, hands-on: inventory the book you just cloned, commission and run real research, have Blueprint draft the plan from that evidence, and then sign it — or refuse to, with reasons, which counts just as much.

The gate at the end of this chapter is called **GATE 0**, and it carries the rule that will follow you through this entire book: *the agent prepares; the human signs.* Your agent will do every step below with you — the listing, the research, the drafting of the plan. The one thing it cannot do, by the system's constitution, is approve the result. That signature is yours, it gets logged, and every chapter after this one will cite it.

## The exercise: six steps to a signed plan

Work with your agent. Each step produces an artifact you can open.

### Step 1 — Inventory the book you cloned

Ask your agent to read `chapters/` in your `writing-guide` clone and produce an inventory table: chapter number, title, what it actually teaches (one line, not the title restated), and approximate length. Read the table. Fix at least one row where the agent's one-liner misses the chapter's real point — you'll find one, and finding it is the lesson: *the inventory is a claim, and you just checked it.*

**Artifact:** the inventory table, saved in the repo.

### Step 2 — Write the research prompt

You're converting a book that teaches *writing* into a book that teaches *copywriting* — writing that exists to make a reader act. That conversion raises two genuine research questions, and you're going to commission the research rather than trust anyone's memory, including your agent's:

1. **Structure:** How do composition skills map to copywriting skills? Which of this book's 24 chapters have a copywriting counterpart (a profile becomes a case study; a proposal becomes a pitch; rhetorical analysis becomes an ad teardown), which have none, and what does a copywriting book need that a composition book never taught — headlines, offers, calls to action, landing pages, email, brand voice?
2. **Principles:** The classic principles of direct advertising — state the differentiator before writing a word, translate features into benefits, earn attention before asking for it — were formulated in a world of paid pages and scarce attention. **Do they still hold in an AI world**, where copy is generated in volume, feeds and search mediate attention, and your reader may be a recommendation algorithm before it's a person? Treat these as hypotheses to test, not doctrine to inherit — and cite them as principles, not as any famous name's endorsement.

Draft the prompt with your agent — then **read it**. Both tests must pass: would a stranger know what to answer and what evidence to bring back, *and do you understand every line of it yourself?* Agents love elaborate prompts; if you can't say what yours asks, you can't judge what comes back. Rewriting a prompt two or three times is normal and fine. Running a prompt you haven't read and thought about is not — that's the human gate, and it applies to prompts just as much as to plans.

**Artifact:** the research prompt — the version *you* understand.

### Step 3 — Run the research

Have your agent run it — web sources, with citations, dead links rejected. Demand the same discipline the system demands everywhere: every claim in the findings traces to a source you could click. If the agent asserts that "benefit-led copy outperforms feature-led copy," there had better be a study, a documented test, or an honest "practitioner consensus, evidence thin" label attached.

Better still, don't run it once. The same plain-language prompt will run in any research system — your CLI agent, Gemini Deep Research, OpenAI's deep research — and *you* are the only one with access to all of them. Run two or three, hand the results back to your agent, and have it cross-verify: claims that independent systems agree on (with independent sources) get believed; claims only one system makes get checked; flat disagreements are the most valuable output of all, because they mark exactly where the evidence is genuinely unsettled. Triangulation is cheap, and it's the same habit as the fact dictionary: never let one fluent answer stand in for verification.

### Step 4 — Save it to the pantry

Research that lives in a chat scroll dies with the chat. Have the agent save the findings to `pantry/` — the repo's room for raw material — as a named file (e.g., `pantry/copywriting-conversion-research.md`). From now on, that file is citable: the Blueprint will reference it, Chapter 9 will draft from it, Chapter 11 will fact-check against it.

**Artifact:** the pantry file. Open it. Skim the sources. This is what "the trail" means.

### Step 5 — Blueprint

Now ask the agent to run **Blueprint** (the planning consultant in `conductor/00-blueprint/`) against two inputs: your inventory and your research. It will produce the new book's plan — vision (who the copywriting reader is, what the book argues), architecture (what each chapter builds, in what order, and the **assessment plan**: quizzes, exercises, flashcards, Canvas modules — each marked desired, needed, or out of scope), chapter spec, risks, and a proposed table of contents. With 24 inherited chapters, expect Blueprint's consolidation audit to fire — a course-adoptable book wants 12–14. Watch which chapters it proposes to cut, and notice whether you flinch. The flinch is information.

### Step 6 — Sign, or refuse

Read the plan. Not skim — read, with three questions: Is this reader someone I can picture? Does each chapter build something the research supports? Would I cut what it cuts? Then decide:

- **Approve:** tell your agent to record the sign-off — GATE 0 checked in `STATUS.md`, the plan's files marked verified with your name and the date. *The approval is logged.* That log line is the first entry in a trail that will, eleven chapters from now, let you defend every structural decision in your finished book.
- **Refuse:** just as valid. Name what's wrong, have Blueprint revise, read again. A refused gate with reasons is the system working; a reluctant signature is the system failing.

**Artifact:** the logged decision — either way.

## Worked example: one row, one finding, one plan line

From a real run of this chapter: the inventory row for writing-guide's Chapter 7 read *"Profile — telling a rich and compelling story: interview-based portrait of a person, emphasis on concrete detail."* The research's structure pass found that copywriting's nearest genre is the **customer case study** — same interview, same concrete detail, but organized around a transformation and ending in proof. The Blueprint line that resulted: *"Ch: Case Studies That Sell — converts writing-guide ch. 7; keeps interviewing and detail; adds outcome framing and a proof section; exercise: convert one profile into a case study with a verifiable result."* Inventory → evidence → plan, one line of each, all three citable. That's the loop, in miniature, that you just ran twenty-four times.

## What can go wrong

| Symptom | What it means | Fix |
|---|---|---|
| Agent starts rewriting chapters during Step 1 | it skipped the plan, like everyone wants to | stop it; point it at this chapter; inventory only |
| Research findings with no citations | fluent ≠ verified | send it back: every claim gets a source or a "consensus, evidence thin" label |
| Blueprint keeps all 24 chapters | consolidation audit didn't fire or was ignored | ask for it explicitly; a plan that cuts nothing decided nothing |
| You can't decide whether to sign | the plan is vague somewhere — find where | name the vague section; refusing with reasons beats signing with doubts |

## Bridge

You now own a signed plan for a book that doesn't build under your name yet. Chapter 2 fixes that in about ten minutes — you'll change one sentence, run one command, and watch a real EPUB come out with your change in it. Then you'll put your name on the cover.

---

## Bear's Copywriting Book

*Every chapter of this book ends with this section: our actual run of the chapter's exercises, on the real clone, warts included. It's not the answer key — your run will differ, and should. It's proof the exercises work, and a worked example of what "done" looks like.*

### Step 1 — the inventory run

The ask, verbatim: *"Read `chapters/` in the writing-guide clone and produce an inventory table: chapter number, title, what it actually teaches — one line, not the title restated — and approximate length."* The agent read all 24 chapters (their TL;DRs and section structures) and came back with the table below. Word counts are exact (`wc -w`), rounded to the nearest hundred.

| # | Chapter | What it actually teaches | ~Words |
|---|---|---|---|
| 00 | Introduction | Route map: the gap between knowing the terms and using them with judgment | 1,700 |
| 01 | The Things We Carry | Every writer arrives with a loaded pack — experience, culture, language, now AI — and writing starts by inventorying it | 4,600 |
| 02 | The Digital World | Rhetorical reading of the media you already scroll: seven visible elements, with worked examples | 5,800 |
| 03 | Language, Identity, and Culture | What "Standard English" is and isn't; borrowing the academy's voice without surrendering your own | 4,100 |
| 04 | The Literacy Narrative | Scene-and-summary machinery for telling how you became a reader/writer | 5,600 |
| 05 | Bridging Identity and Academia | Which personal material transfers into academic genres, which doesn't, and the two ways the move fails | 5,100 |
| 06 | Memoir / Personal Narrative | Building a scene around the moment you changed — while handling memory's unreliability honestly | 4,800 |
| 07 | The Profile | Interview-driven portrait held together by one controlling sentence; three kinds of research; the puff-piece trap | 5,200 |
| 08 | The Proposal | Defining a problem narrowly enough to be solvable, then arguing a feasible fix section by section | 5,600 |
| 09 | Evaluation / Review | Judgment with earned criteria — what separates a review from a reaction | 4,900 |
| 10 | The Analytical Report | Writing from facts when fact-vs-opinion doesn't solve cleanly: three biases you compensate for | 5,100 |
| 11 | Rhetorical Analysis | Taking an argument apart at the seams: the appeals, the situation, named persuasion moves | 5,900 |
| 12 | The Position Argument | Defending a claim a reader might reject: debatable issue, claim, counterargument, two structures | 4,700 |
| 13 | Reasoning Strategies | A toolkit of reasoning patterns — analogy, cause-and-effect, classification — and when each persuades or misleads | 5,800 |
| 14 | Argumentative Research | Argument that enters a scholarly conversation: a sharper thesis and a source-quality hierarchy | 4,900 |
| 15 | The Research Process | Actually finding and recording evidence: library moves, field methods, source evaluation beyond the checklist | 5,100 |
| 16 | Annotated Bibliography | Summarize-and-evaluate discipline per source — and why the evaluation half is always the weakest | 4,600 |
| 17 | Case Study Profile | **Single-subject research: methods and ethics for studying one person, and what one case can claim about the many** | 5,100 |
| 18 | Navigating Rhetoric in Real Life | Transferring the toolkit beyond the classroom — including when LLMs belong in public-facing writing | 4,000 |
| 19 | Print / Textual Analysis | Close reading as evidence-based interpretation, written for an interpretive community | 4,900 |
| 20 | Image Analysis | Reading images as arguments: visual elements, plus the context the image can't supply | 4,900 |
| 21 | Multimodal and Online Writing | Combining text, image, and sound: what each mode rewards and how the rhetorical questions shift | 4,400 |
| 22 | Scripting for the Public Forum | Writing for the ear: repetition machinery, talk-scale structure, what delivery controls that the script can't | 5,000 |
| 23 | Portfolio Reflection | Evidence-based reflection on your own growth — three required moves, harder than it looks | 5,100 |

Twenty-four chapters, ≈117,000 words, median ≈5,000 per chapter.

**The row we fixed.** The agent's first one-liner for chapter 17 was *"a longer, more in-depth version of the Chapter 7 profile"* — a title restatement wearing a disguise. Reading the chapter's actual sections (research methods, ethics, what one case can claim about the general) shows it's a qualitative-research chapter, not a bigger profile. That's the bolded row above, corrected. One row out of twenty-four — which is roughly the hit rate you should expect, and exactly why the exercise says *read the table*: the inventory is a claim, and we just checked it.

**Artifact:** saved in the clone at `pantry/ai1-ch1-inventory.md` — where Step 2's research prompt and Step 5's Blueprint will cite it. (You can already see conversion candidates flickering in the table — profile→case study, proposal→pitch, review→product review — but those are Step 5 decisions, not Step 1 ones. The inventory's job is to be true, not to be the plan.)

> **Bear's note (2026-06-12):** It's long. I like it — this is a fine first pass, and I'll almost certainly cut it down later. I don't really know what I want to cut until it's written, so for a rough draft, this is exactly fine.

*(Notes like that one are part of the method, not decoration: the margin is part of the record. A first pass you intend to cut is honest work; a first pass you pretend is final is the fluency trap with your own name on it.)*

### Step 2 — the research prompt, rejected once

The agent's first draft was impressive. It classified every chapter into CONVERTS / CUTS / UNCLEAR taxonomies, decomposed the principles question into five numbered hypotheses with a HOLDS / HOLDS-MODIFIED / SUPERSEDED grading rubric, and specified a three-tier source-grading scheme with retrieval dates. Thorough, structured, machine-shaped.

The author read it and said: **"I don't understand that prompt at all."**

That sentence is the human gate doing its job. A prompt the human can't read is a prompt the human can't judge — and everything that comes back from it inherits the problem: how would you check findings against questions you never understood? So the elaborate version was rejected and kept (`pantry/ai1-ch1-research-prompt-rejected.md` — go read it as a specimen), and the replacement asks the same things in plain language:

> For **every chapter**: (1) Is this still relevant in the age of AI — people draft with AI now; does this skill still matter, and to whom? (2) If it's relevant, how would it change for a copywriting book — what's the counterpart, what's different?
>
> For **the book as a whole**: (3) Is classic Ogilvy-style copywriting still relevant in the age of AI — which principles hold, which need updating, and why? (4) Is there anything missing — what does a copywriting book need that this writing book never taught?
>
> Ground every answer in sources I can click. When the evidence is thin or just practitioner opinion, say so plainly.

Four questions. A stranger could answer them; more importantly, the person who has to sign GATE 0 understands exactly what was asked. The lesson of this step, in one line: **rewriting prompts is normal and fine — running a prompt you haven't read and thought about is not.** The rewrite cost two minutes. Running the unread version would have cost a research file shaped like a rubric nobody wanted.

**Artifacts:** `pantry/ai1-ch1-research-prompt.md` (the version that runs) and `pantry/ai1-ch1-research-prompt-rejected.md` (the gate's receipt).

### Step 3 — the research run (triangulated)

Here's how we're running it. Bear asked the CLI agent to run the research — and is *also* running the identical prompt through **Gemini Deep Research** and **OpenAI deep research**, two systems the agent has no access to. All three result sets land in the pantry, and the agent's job shifts from sole researcher to **cross-verifier**: build one findings file where every claim is marked by how many independent runs support it, every source is deduplicated and checked, and every disagreement between the three systems is surfaced rather than smoothed over.

Notice the labor split, because it's the book's whole argument in miniature: the human is the only party who can reach all three systems, so the human orchestrates; the agent is best at the tedious-and-exacting part, so the agent reconciles; and nothing gets believed because one model said it fluently. Where the three runs agree from independent sources, the Blueprint gets a solid plank. Where they disagree — and they will — that's not noise, that's the research telling us where the AI-era copywriting question is genuinely open, which is exactly what Question 3 wanted to know.

**What came back.** The two deep-research systems returned strikingly different *shapes*. Run A produced a confident translation matrix — all 24 chapters mapped to copywriting counterparts, Ogilvy principles sorted into holds-vs-modernize, Schwartz's awareness stages, even conversion formulas — and rated essentially *every* chapter "highly relevant." Run B discriminated: it labeled the evidence quality of every judgment ("strong," "moderate," "thin and mostly adjacent"), cited institutions (NIST, FTC, Google's spam policies, platform surveys), and demoted three chapters — the literacy narrative, the memoir, the portfolio reflection — to "selective."

The reconciliation marked every claim: **✓✓** where both runs agreed from their own reasoning (most of the conversion map — profile→case study, proposal→sales page, rhetorical analysis→funnel teardown — and the whole Ogilvy verdict: *principles survive as disciplines; the environment they assumed is gone*), **⚡** where they disagreed, and **⚠** on specifics neither we nor the second system could verify (a named book's "lost chapter," a platform integration, every percentage whose link got stripped in transit — flagged: *do not enter the manuscript uncited*).

Three things the triangulation bought that one run wouldn't have:

1. **A grading curve with no failures is itself a finding.** Run A flattered the premise — twenty-four for twenty-four "highly relevant" should make you suspicious, and only having a second run beside it makes the flattery visible.
2. **The disagreements are the decision.** The contested chapters (04, 06, 23 — the autobiographical school genres) aren't research failures; they're the research telling us the answer depends on a vision question the Blueprint must now ask: *is our reader a founder or creator whose person is part of the product?* If yes they convert; if no they cut. Either way it's decided explicitly, not averaged.
3. **The agent's honest leg.** The CLI agent ran no independent web pass (sandboxed) — its contribution was reconciliation, checking both runs against the actual manuscript inventory, and flagging the unverifiables. Knowing what each leg can and can't do is part of the method.

**Artifacts:** `pantry/ai1-ch1-research-findings.md` — the reconciled, cross-marked file the Blueprint will cite, ending with six implications for the table of contents (including the math that forces consolidation: ~17 convertible chapters + ~7 new ones > the 12–14 target). The raw exports go in beside it — with their live citation links, which chat transfer strips.

### Step 5 — the Blueprint, built from the evidence

With the findings in hand, Blueprint ran against both pantry files plus one structural decision the research disagreements forced: **name the readers.** Four were named — the employed copywriter (the core's *stated default*, so the book's bias is a decision rather than an accident), the founder, the creator, and the freelancer — with founder and creator merging into one module (both monetize a person; they differ by channel, not craft).

The proposal that came back: a **14-chapter core** in which every chapter traces to its research support (the strongest-evidence material — voice-of-customer interviewing, the human moat both runs named — sits at the heart), two supplemental **modules** (*When You Are the Product* for founders/creators, carrying the scene-and-summary craft salvaged from the demoted memoir chapters; *When Copy Is Your Business* for freelancers), and a full disposition table for all 24 source chapters: 21 convert or merge, 2 demote to Module A, 1 splits — **none wholly cut**, though the school apparatus (assignment framing, rubric-shaped evaluation) dies globally, with the reason logged. The consolidation audit fired exactly as predicted, and every merger pairs chapters the research mapped to the same counterpart.

Two discipline notes worth copying. The demotions are justified by **audience-conditional relevance** — the thing the triangulated disagreement actually established — not by "thin evidence," which would punish uncertainty as if it were a verdict. And the one chapter built on single-run, unverified-as-delivered sources (buyer-awareness frameworks) carries a hard rule in the proposal itself: *verify before drafting, or the chapter reframes around what verifies.*

**Artifact:** `pantry/ai1-ch1-blueprint-proposal.md` — vision, the four readers, the assessment plan (every chapter produces a portfolio piece), the 14+2 TOC with support marks, the disposition of all 24 chapters, top risks, and four open questions for the human. Which is the cue for Step 6: read it, and sign or refuse.

### Step 6 — the sign-off, and why

Bear read the proposal and approved it — three words in the chat: *"That looks great."* The agent's job at that moment is not to celebrate; it's to **make the approval a record**. The proposal file's status block now reads GATE 0 SIGNED, with the signer, the date, and — this is the part that matters — *the why*, stated so that anyone reading the plan later knows what the signature actually warranted:

- **Every TOC line traces to marked evidence.** Approving the plan means approving its trail, not its vibes — each of the 14 core chapters carries its ✓✓/✓/⚠ support mark back to the findings file.
- **The readers are named.** The core's bias toward the employed copywriter is a decision on the record, not an accident waiting for a reviewer to find.
- **The demotions have the right reason.** Modules A and B exist because relevance is audience-conditional — the thing the research disagreement actually established — not because evidence was thin, which would have punished uncertainty as if it were a verdict.
- **Nothing was cut silently.** All 24 source chapters have a disposition and a reason; even the one global cut (the school apparatus) is logged with its why.
- **The unverified material is fenced.** The one chapter resting on single-run sources carries its own rule — verify before drafting, or reframe around what verifies — so the signature doesn't accidentally endorse claims nobody checked.

**And this is what was signed** — the copywriting book's table of contents, as approved at GATE 0:

| # | Core chapter | Built from | Support |
|---|---|---|---|
| 1 | The Brief: Judgment Over Fluency — directing, evaluating, and rejecting AI copy | 00 | ✓✓ |
| 2 | Voice: Yours, the Brand's, the Customer's — guardrails, tone, dialect with intent | 01+03 | ✓✓ |
| 3 | Reading the Feed: Teardowns — swipe files, funnel reverse-engineering, AI-search surfaces | 02+11 | ✓✓ |
| 4 | Voice of the Customer: Interviews and Message Mining — the human moat | 07+19 | ✓✓ |
| 5 | Offers and Positioning — the product as hero; what copy can't rescue | new | ✓✓ |
| 6 | The Sales Page: Problem to Pitch — section-by-section argument | 08 | ✓✓ |
| 7 | Objections and Risk Reversal — skepticism as the market condition | 12 | ✓✓ |
| 8 | Buyer Awareness and the Frameworks — Schwartz's stages; PAS, BAB, AIDA | new | ✓ ⚠ *verify first* |
| 9 | Reasoning That Survives Scrutiny — analogy, cause, classification; QA on fluent nonsense | 13 | ✓✓ |
| 10 | Research Ops and the Claims File — primary research, provenance, the compliance database | 10+15+16 | ✓✓ |
| 11 | Proof: Case Studies, Reviews, and Evidence Assets — FTC-aware testimonials, white papers | 17+09+05+14 | ✓✓ |
| 12 | Multimodal: Image, Page, and Script — text-image fit, layout, VSLs, short-form video | 20+21+22 | ✓✓ |
| 13 | Measurement and the Postmortem — CR/CTR/ROAS, testing, the campaign debrief | new + 23 | ✓✓ |
| 14 | AI Operations and Governance — model choice, sign-off, disclosure, provenance | new + 18 | ✓ |

| Module | For | Carries |
|---|---|---|
| A — When You Are the Product | founders + creators | origin story, transformation narrative (scene craft from 04/06), building in public |
| B — When Copy Is Your Business | freelancers | proposals and pricing as self-applied copy, the portfolio case study, client acquisition as your own funnel |

Fourteen core chapters, two modules, all 24 source chapters accounted for — and every row checkable against the findings file it came from.

The four open questions were **deferred with a note** rather than rushed — the gate rule allows that, provided each is resolved before its chapter drafts. And one practical wrinkle became a forward link: this clone predates AI+1 governance, so the approval is logged in the proposal file itself, to be transferred to a proper `STATUS.md` when the book is brought under governance — which is Chapter 2's job, and now you know one reason it matters.

That's Chapter 1 complete, and notice what you never did: edit a chapter. You inventoried a real book, commissioned research, triangulated it, watched a plan get built from the evidence, and signed your name to it. Everything that happens in the next eleven chapters happens *because* this signature exists — and can be checked against it.
