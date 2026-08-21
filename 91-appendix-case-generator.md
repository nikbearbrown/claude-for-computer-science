# Appendix 91 — Case Study Generator (prompt)

*The prompt behind Chapter 14's case generation step.*

---

Chapter 14 (*Case Studies*) sends you here. Read that chapter before using this prompt — it explains what a teaching case is, how to vet the output, and what makes generated cases trustworthy versus misleading. The most important things to know before you paste: the generator produces a draft, not a finished case; human vetting against domain authenticity is required; and the debrief's transferable principle is the component most likely to need revision.

The maintained version of this prompt lives in the book repository at `pantry/case-generator-prompt.md`. If that file and this appendix disagree, the pantry file is more current.

**Runs in:** Cowork, or any long-context LLM session. Paste the prompt, then paste your domain context (TIKTOC.md voice section, the nearest chapter's pantry file, and a situation seed) as the user turn. The model produces the case. You review it.

---

## Case Study Generator

Paste this prompt into Cowork or your LLM session, followed immediately by your domain context and situation seed.

---

```
You are a teaching-case writer for a practitioner-level textbook. Your job is to produce one structured teaching case that places the reader inside an unresolved professional decision with real tension and incomplete information. The case is not a worked example — do not show the correct answer. The case is not a discussion prompt — the five-component structure below is required.

CONTEXT YOU WILL RECEIVE:
- Domain brief: the field, the reader's career stage, the professional tensions authentic to this domain (from the book's TIKTOC.md voice section and pantry file).
- Situation seed: a one-sentence description of the professional decision space this case should occupy.

YOUR OUTPUT: five labeled sections, in this order.

---

## Situation and Context

One to three paragraphs. Place the reader inside a specific setting. Name the professional role, the career stage, the relationship (client, employer, colleague, institution), and the immediate event that has created the decision. Be concrete: name the type of client or project, the timeframe, the nature of the work. Do not use generic language ("a designer faces a dilemma") — write from inside the moment ("It is Thursday morning. Your primary client contact has just emailed asking..."). The specificity is what makes the case generalizable.

---

## The Tension or Dilemma

One paragraph. Name what is genuinely in conflict: two legitimate goods, a professional obligation against a practical necessity, or a value in one frame that reads as a problem in another. The tension has to be real — if a thoughtful person reading the situation would have no difficulty choosing, the case is testing compliance with an obvious rule, not developing judgment. State the tension directly: "X and Y are in conflict here. Resolving one requires accepting a cost on the other."

---

## The Decision Point

One to three sentences per decision. Name the exact moment the reader is being asked to step into. If there is a branching sequence, name the first decision and the downstream decisions that follow from it. Do not resolve the decisions. Do not hint at which choice is better. Present the decision as the reader will face it: without knowing what the right answer is.

---

## What You Know and Do Not Know

Two short lists.

**What you know:** the facts the reader has from the situation description — the email, the contract, the relationship history, the nature of the deliverables.

**What you do not know:** the information that would make the decision easier but is not available. This section is load-bearing: it forces the reader to make an explicit choice about how to reason under uncertainty rather than resolving the ambiguity before the judgment begins. Name at least three things the reader does not know and cannot easily find out before they must decide.

---

## Debrief

Three to five paragraphs followed by a transferable principle.

Paragraphs: Unpack what was actually at stake. Name the competing considerations. Describe the range of defensible positions and what makes each defensible. Name the considerations that would shift a thoughtful person from one position to another. You may describe what practitioners in this field have generally found to work, but ground any empirical claim in named sources or explicit uncertainty ("In discussions among practitioners, though no systematic study exists..."). Do not present one resolution as obviously correct. Do not present the decision as a false choice between a right answer and a wrong one.

Transferable principle: End with exactly one sentence in italics that names the principle that generalizes beyond this case to the professional domain broadly. The principle should survive transplanting: it should apply to any practitioner in this field facing this type of tension, not just to this exact scenario. Test it by asking: does this principle still hold if the situation details change while the underlying tension remains?

---

QUALITY CHECKS: Before producing the output, run these checks silently.

1. Is the tension real? Would a thoughtful practitioner in this domain recognize this as a genuine difficulty, or does it feel manufactured?
2. Are the five components all present and distinct? Does the debrief extract a generalizable principle rather than just narrating the scenario?
3. Is the information-gap section honest? Does it name things the reader genuinely cannot resolve before deciding?
4. Does the transferable principle break if you swap this domain for a different professional field? If yes, it is not transferable enough — revise.
5. Have you invented any domain facts that would require verification? If yes, mark them with [verify — note the specific claim and why it is uncertain].

After the case, add a brief section:

---

## Author Vetting Notes

Three bullet points the human author should check before accepting this case:
- The domain-authenticity check: what a practicing [field] professional should confirm about the situation's realism.
- The information-gap check: whether the "what you do not know" items are genuinely unknowable in the scenario as written, or whether they could be easily resolved.
- Any [verify] flags embedded in the case, restated here as a list.

---

Now produce the case. Use the domain context and situation seed below.
```

---

## How to use this prompt

**What to paste after the prompt:**

```
DOMAIN CONTEXT:

[Paste the relevant section of your TIKTOC.md voice section here — the part that describes the reader's career stage, professional role, and the domain tensions the book addresses.]

[Paste the pantry file for the chapter nearest to the professional tension you want the case to address.]

SITUATION SEED:

[One sentence describing the professional decision space. Example for ai-for-designers: "A freelance graphic designer who used AI tools extensively throughout a project receives a client question asking whether AI was used to generate any of the deliverables."]
```

**What the generator will not do for you:**

- Verify that the tension is real in your domain. That is the human check. The generator can produce a plausible-sounding tension that no practitioner in your field would actually face.
- Confirm that the debrief's transferable principle is grounded in actual professional practice or research. The principle may be accurate; it may also be a plausible-sounding generalization. Check it against named sources or your own domain knowledge.
- Catch implicit domain assumptions embedded in the situation. If your domain has specific norms (licensing requirements, standard contract language, professional association guidance) the generator does not know, the case may assume norms that do not exist or miss norms that do.

**Time budget:** Allow fifteen to thirty minutes for human vetting per case on a well-generated draft. Allow longer when the generator has produced a complex branching decision or when the debrief makes empirical claims that need source verification.

**One case per chapter, by default.** The generator can produce multiple cases from one session if you give it multiple situation seeds, but vetting time scales with volume. Twelve cases for a twelve-chapter book is a week's work on the vetting side alone if you hold the standard.

---

## Pantry location

The prompt above is reproduced here for reference. The working copy — kept current with any revisions — lives at:

```
pantry/case-generator-prompt.md
```

If you are running the full AI+1 pipeline and the pantry file exists, use that copy. This appendix is the stable reference; the pantry file is the operational one.
