# Appendix — The AI+1 Recipe: Building a Course from Scratch

*Every step from an idea to a shipped, fact-checked, multi-format course — and the human signature that closes each one.*

This appendix is the whole book on one page. It names, in order, every step of the AI+1 pipeline: what you do, the prompt or script that runs it, what it needs, what it produces, and the gate a human signs before the next step begins. The other appendices (A–J and 90–97) are the full text of each prompt and script referenced here; the numbered chapters teach the judgment each step demands. This page is the map.

One principle governs the whole recipe: **the agent executes, the human signs.** Agents draft, research, render, and package tirelessly. The human owns the three decisions that carry authorial and legal weight — the *plan*, the *voice*, and the *facts* — and every artifact downstream cites those signatures. Nothing is published by a script; the final upload, import, or deploy is always a human act.

Two things before the steps. First, the recipe has two on-ramps that converge at scaffolding: building a course **from scratch** (start at Phase 1) or **remaking an existing book/course** (inventory it first, then run research against the restructure — the path the drafted chapters teach). Second, there are two gate systems that share the word "GATE": the planning consultant's internal **GATE 1–4** (which close each phase *inside* the Tic TOC session), and the operating spine's **GATE 0 / GATE 3 / GATE 4** (the human signatures on plan, voice, and facts). Both appear below where they fire.

---

## The pipeline at a glance

| Phase | Step | Runs with | Produces | Gate |
|---|---|---|---|---|
| 1 | Domain research | Appendix B prompt × Claude + GPT-4o + Gemini | A triangulated research brief | — (human reconciles) |
| 2 | Plan / Blueprint | Appendix A — Tic TOC prompt (`/i /l /c /m /p`, `/scaffold /silent`) | `TIKTOC.md` + `vision/architecture/chapters-spec/risks.md` | Internal **GATE 1–4** |
| 3 | Sign the plan | Human review | Signed plan in `STATUS.md` (`verified` + name + date) | **GATE 0** |
| 4 | Scaffold | `new_book.py` | The book directory (`chapters/`, `pantry/`, `images/`, `d3/`, …) | — |
| 5 | Per-chapter research | Appendix D — Research Gatherer / Research Pass | `pantry/*_notes.md` + flags | (cites GATE 0) |
| 6 | Draft | Appendix E — Chapter Writer | `chapters/NN-slug.md` + `logs/log.csv` | — (never publishes) |
| 7 | The human rewrite | Appendix F — the Combined Test (14 items) | Rewritten, author-owned chapters | **GATE 3** |
| 8 | Finishing + figure plan | Appendix G — Finishing Pass + CAJAL Image Suggest | Subtitles, visual comments, `pantry/*-cajal.md` | — |
| 9 | Figures | Appendix I — CAJAL (`/scope`…) + `node SCRIPTS/svg-to-png.mjs` | `images/*.svg` + 300-DPI PNG, `d3/*.html` | grayscale/colorblind test |
| 10 | AI enrichment | Appendix H — enrichment prompts | Chapter 00 (Claude Basics), Dig-Deeper prompts, LLM exercises | pause-to-select |
| 11 | Assessment assets | Appendices 91 / 92 / 93 | Teaching cases, Glimmers, `## Recall` cards | human vetting |
| 12 | Fact-check | Appendix J — Fact-Checking Assistant | `factchecks/MASTER_REPORT.md` + inline flags | **GATE 4** |
| 13 | Build & ship | `build.sh`, `build-imscc-standard.py`, `build-anki.py`, `build-react-site.py`, Appendices 96–97 | EPUB/PDF, `.imscc`, `.apkg`, React site, LTI tutor, Ask-AI layer | human uploads/deploys |
| 14 | (optional) Book → video | Ch 13 — Unreal Reels | Narrated lecture / explainer / reel | sign the lecture |
| 15 | (optional) Book → other media | Canvas / NotebookLM / Kindle / Medhavy | `.imscc`, NotebookLM audio, Kindle EPUB, `.mdx` | each platform's checks |

---

## The steps in full

### Phase 0 — Start with an idea
You need only two things to begin: a concept and a named reader. Everything else is built. If you are remaking an existing book instead of starting cold, inventory it first — what each chapter teaches and for whom — because that inventory becomes the evidence the plan is argued from.

### Phase 1 — Domain research (evidence before planning)
Run the **Domain Research Prompt** (Appendix B) with your field and role filled in, separately in **Claude, GPT-4o, and Gemini**, then reconcile the three outputs by hand. Running one model is the failure mode the book warns about; the finding worth keeping is where the three *disagree*. The output is a research brief — a task map, an irreducibly-human list, the top failure modes, the training gap — that you paste into the planner's project knowledge.

### Phase 2 — Plan the course (the Blueprint)
Paste the **Tic TOC** prompt (Appendix A) into a Claude or ChatGPT project and work its command sequence: intake `/i1–/i4`, learning architecture `/l1–/l4`, chapters `/c1–/c4`, market `/m1–/m4`, production `/p1–/p3`, then `/g1 /fulltoc` and `/scaffold` (run `/silent`). It produces `TIKTOC.md` and the four planning files — `vision.md`, `architecture.md`, `chapters-spec.md`, `risks.md`. The consultant will not cross its own **GATE 1–4** until you confirm each phase, and it audits for chapter bloat at 18 (hard ceiling 20; target 12–14). This is where a from-scratch course and a remake converge: both leave Phase 2 with a Blueprint.

### Phase 3 — Sign the plan → GATE 0
Read the Blueprint and either sign it or refuse it, logging the decision (`verified` with your name and the date) in `STATUS.md`. **This is the first load-bearing signature.** Every downstream step cites it — the research and drafting prompts are built to *refuse to run from an unsigned outline*. Signing the plan is the moment the course becomes a commitment rather than a suggestion.

### Phase 4 — Scaffold the directory
Run `python new_book.py "Title" "Author"` (with optional `--subtitle`, `--chapters N`, `--tiktoc TIKTOC.md`, …). It writes the full working directory — `book.md`, the four planning files, `chapters/` stubs, `pantry/`, `images/`, `d3/`, `SCRIPTS/`. Thin files here mean a thin spec; the scaffold makes vagueness visible before you've written a word.

### Phase 5 — Research every chapter
Paste the **Research Gatherer** (lighter) or **Research Pass** (deeper, nine sections) from Appendix D. Reading `TIKTOC.md` — or the signed GATE 0 plan — it writes one notes file per chapter to `pantry/NN-slug_notes.md`, each carrying explicit **flags and gaps**. Its rules are hard: *never write chapter prose, never fabricate a source.* You triage the flags. Research every chapter before drafting any: that ordering is Phase 1 of the spine.

### Phase 6 — Draft the chapters
Paste the **Chapter Writer** (Appendix E). From `TIKTOC.md`, `book.md`, and each chapter's pantry notes, it writes every undrafted `chapters/NN-slug.md` (default eight-section anatomy, including "What would change my mind" and "Still puzzling") and logs to `logs/log.csv`. It leaves existing chapters untouched and it **never publishes** — every draft stops in `chapters/` for your review.

### Phase 7 — The human rewrite → GATE 3
This is the one step with **no generator**. Apply the **Combined Test** (Appendix F) — a fourteen-item pass/fail read-through — to each chapter; the three lowest items are your rewrite targets. Then rewrite. The two failure modes it catches, voice drift and fabricated specificity, read fine on a skim and fail the book. **This is the second load-bearing signature (GATE 3):** the rewrite is where the author's expertise actually enters the book, and it is the book's whole argument in miniature.

### Phase 8 — Finishing pass and figure planning
Paste the **Chapter Finishing Pass** then **CAJAL Image Suggest** (Appendix G). The finishing pass adds an italic subtitle and inline visual placeholders (`<!-- → [IMAGE/TABLE/INFOGRAPHIC/CHART: …] -->`) without rewriting prose; Image Suggest writes one figure plan per chapter to `pantry/<slug>-cajal.md`. It flags rather than skips.

### Phase 9 — Make the figures
Use **CAJAL** (Appendix I) — the figure architect with the SCOPE framework and commands `/scope`, `/hero`, `/scan`, `/video`, `/split` — to design each figure, then render with `node SCRIPTS/svg-to-png.mjs` into `images/*.svg`, 300-DPI PNGs, and optional `d3/*.html`. The house rules are enforced: Okabe-Ito colorblind-safe palette, at most 6–8 components per figure, no baked-in text labels, axes from zero, and it will not draw until you've stated the concept in one sentence and named what to leave *out*.

### Phase 10 — Enrich for AI
Paste the enrichment prompts (Appendix H) to add the LLM-native layer: a **Chapter 00 (Claude Basics)**, inline **Dig Deeper** prompts, and end-of-chapter **LLM exercises**, plus deep-research and when-to-use-AI passes and comment-driven tables and figures. It pauses for you to select the running project and confirm Chapter 00 before proceeding.

### Phase 11 — Build the assessments
Generate the graded and self-study layer the plan promised: **teaching cases** (Appendix 91 — Case Study Generator, five-part case + author vetting notes), **Glimmers** (Appendix 92 — a Socratic interrogator that makes the student defend an answer, run in its own conversation), and **spaced-repetition cards** (Appendix 93 — atomic `Q:`/`A:` pairs in a `## Recall` section). Each requires a human vetting pass for domain authenticity and coverage.

### Phase 12 — Fact-check → GATE 4
Paste the **Fact-Checking Assistant** (Appendix J, web access on) at the book directory. It classifies every assertion, assigns a verdict (CONFIRMED / OUTDATED / UNVERIFIED / CONTRADICTED), and writes `factchecks/MASTER_REPORT.md` plus inline flags and reference sections. Items marked OUTDATED, CONTRADICTED, or COMBINATION are flagged for expert review. **This is the third load-bearing signature (GATE 4):** claims are verified here, before anything is built or filmed.

### Phase 13 — Build and ship everywhere
The chapters are the single source of truth; deterministic, standard-library scripts package them into every format, and **none of them publishes on its own** — the human's upload, import, or deploy is the final confirmation each time:

- **EPUB / PDF** — the shared `chapters/*.md` + `metadata.yaml`, via `./build.sh`.
- **Canvas** — `python3 build-imscc-standard.py` → one `.imscc` (IMS Common Cartridge 1.3); you import it and diff module order against the signed plan (Appendix 90).
- **Anki** — `python3 build-anki.py` compiles the `## Recall` cards → one `.apkg` the student imports (Appendix 94).
- **React site** — `python3 build-react-site.py` scaffolds a Next.js project; a developer runs `npm install` / build / deploy (Appendix 95).
- **Medhavy AI tutor** — the LTI 1.3 setup (Appendix 96) connects the course to the LMS, gated by a **Step-1 Institutional Review** (FERPA, security, admin approval) before any configuration.
- **Ask-AI layer** — per-surface configuration plus the Parallel-LLM Companion Prompt (Appendix 97) turns on reader-facing AI across Canvas, Medhavy, the React site, and Kindle/PDF.

### Phase 14 — (optional) From book to video
A finished, GATE-4-verified course can feed **Unreal Reels** (Ch 13) to become narrated lectures, explainers, or reels. Two laws hold: *audio is the master clock*, and *every gate is signed by a human*. A fidelity pass checks the narration against the source chapter, you resolve every flag, and you sign the lecture. The signature warrants only that the video is faithful to a verified source — which is exactly why the book had to be fact-checked first.

### Phase 15 — (optional) Book → other media
Because the finished manuscript is Markdown, it converts cleanly onto every reading and study surface. These are the same source files pointed at new destinations; where a build script already appeared in Phase 13, it is the same tool run for a different endpoint. The Markdown stays the single source of truth, every conversion is deterministic, and nothing goes live until a human uploads or submits it.

- **Canvas (LMS)** — `python3 build-imscc-standard.py` converts `chapters/*.md` → one `.imscc` (IMS Common Cartridge 1.3). Import it into Canvas and verify module order against the signed plan (Appendix 90). Runs on the same source as the EPUB.
- **NotebookLM (audio)** — upload the chapter `.md` files (or the combined manuscript) into a NotebookLM notebook; NotebookLM generates the audio overview and deep-dive discussion straight from the source text. Those recordings are the input to the NotebookLM → YouTube pipeline (chapter-ordered episodes on @MedhavyAI).
- **Kindle (e-book)** — `./build.sh` renders `chapters/*.md` + `metadata.yaml` to EPUB with `styles/kindle-book.css`; run the EPUB through **Kindle Previewer's validation** before submitting to KDP so it passes Amazon's checks on every device.
- **Medhavy (AI-tutor course)** — convert each chapter `.md` to `.mdx` (the Medhavy content format) so the text can carry interactive components, then deliver it into the Medhavy course that the LTI 1.3 integration (Appendix 96) wires to the LMS.

---

## The three signatures that carry weight

Everything else in the recipe can be re-run, regenerated, or rebuilt by an agent. These three cannot be delegated, because they are what make the course *yours* and *true*:

1. **GATE 0 — the plan.** You signed what this course should be, argued from evidence.
2. **GATE 3 — the voice.** You rewrote the drafts until you could defend every line as your own.
3. **GATE 4 — the facts.** You verified every checkable claim before a single format shipped.

A course that clears all three is one you can put your name on — which is the entire point of AI+1: not to ask an AI better, but to *ship* a finished, checked thing you signed.
