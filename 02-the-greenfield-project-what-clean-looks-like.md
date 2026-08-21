# Chapter 2 — The Greenfield Project: What Clean Looks Like
*Why the shape of a folder is a message your agent reads before it reads anything else.*

Here is something I have noticed over years of working with tools that read your files: they are not confused by bad writing. They are confused by bad filing. You can write terrible prose and an agent will still process it. But if your project folder looks like the aftermath of someone who kept postponing decisions, the agent will make its own decisions about which file is canonical — and it will be wrong in ways you only discover later.

The question I want to dig into is why. Not "clean projects are better" — everyone knows that. The question is: what is the agent actually doing when it navigates a folder, and why does folder structure interfere with or support that process in ways that feel disproportionate to the effort of creating it?

## The Problem Is Retrieval, Not Intelligence

When you point an agent at a project, it does not read every file. That would be slow and would exhaust context. Instead, it reasons about which files are probably relevant before opening anything.

That reasoning uses names. Path structure. Dates. File types. The shape of the hierarchy.

Think about what that means. The agent is making predictions about what a file contains based entirely on where the file lives and what it is called. A file named `final2.md` in a flat folder is ambiguous — it could be source, draft, or output. A file named `report.md` inside a folder named `current/` inside a folder named `client-project/` is not ambiguous. Before the agent opens it, it already knows: this is an active working draft, not a completed artifact, not archived material.

![Flat folder vs. structured project tree](images/02-the-greenfield-project-what-clean-looks-like-fig-01.png)
*Figure 2.1 - Flat folder vs. structured project tree*

<!-- → [DIAGRAM: Single flat folder on the left containing 8 unlabeled files vs. structured folder tree on the right with current/, outputs/, archive/ — arrow from each right-side file to its role label. Caption: The same eight files, two folder strategies. The left gives the agent no signal before it reads; the right answers the question before it's asked.] -->

This is the core insight. Folder structure is metadata. It is not decoration. It is not organizational virtue. It is a signal layer the agent reads before it reads content. When you name a folder `archive/`, you are not just telling yourself that those files are old. You are telling every tool that ever touches this project: skip this unless you are explicitly asked to look here.

## What the Structure Actually Says

Here is the folder structure I use as a default for any project I expect an agent to touch:

```text
project/
  _PROJECT_INDEX.md
  AGENTS.md
  _MANIFEST.md
  PROJECT_RULES.md
  status.md
  session-handoff.md
  .ai/
    manifest.yaml
    workflows/
    schemas/
    sessions/
    audit/
  src/
  current/
  research/
  templates/
  staging/
  build/
  outputs/
  archive/
```

| Folder name | Role |
|---|---|
| src/ | Canonical source for software or structured artifacts. |
| current/ | Active drafts being edited now. |
| research/ | Evidence, notes, and background material. |
| templates/ | Reusable skeletons and scaffolding. |
| staging/ | Drafts awaiting validation or review. |
| build/ | Regenerable intermediates; do not edit directly. |
| outputs/ | Finished deliverables. |
| archive/ | Superseded material; ignore unless explicitly requested. |
| .ai/ | Machine-readable workflow metadata. |

*Each folder is a role declaration the agent can use before it reads content.*

The names matter. Not because there is a universal standard that every agent recognizes — there is not — but because the names encode roles that any reasoning system can infer. `archive/` means "old." `outputs/` means "finished." `current/` means "live." `build/` means "generated." These are not arbitrary; they map onto how humans already talk about file states, which means they map onto how language models predict what files contain.

Anthropic's own context-engineering guidance treats file organization explicitly as a way to help agents retrieve relevant material without flooding context. The structure does not just help you find things. It helps the agent decide what not to retrieve — which is equally important, because context space is finite.

## Tracing the Transformation

The cleanest way to see what this buys is to watch a messy project become a clean one.

Before:

```text
client-report/
  report.md
  report-old.md
  report-final.pdf
  sources.md
  prompt.txt
  chart.svg
  scratch.json
```

Seven files in a flat folder. An agent asked to "update the report with the new data" has to resolve several questions before it can act: Is `report.md` or `report-final.pdf` the live source? Is `report-old.md` an earlier draft of the same document, or something different? Should `scratch.json` be treated as a data source? Is `prompt.txt` a template to follow or a record of a past session?

The agent will make guesses. Some guesses will be right. Some will not.

After:

```text
client-report/
  current/
    report.md
  research/
    sources.md
  templates/
    prompt.txt
  outputs/
    report-final.pdf
  images/
    chart.svg
  build/
    scratch.json
  archive/
    report-old.md
```

Now the agent asks "update the report with the new data" and the path to the answer is unambiguous. `current/report.md` is the live source. `outputs/report-final.pdf` is the last completed deliverable — context, not target. `archive/report-old.md` is not worth reading unless specifically requested. `build/scratch.json` is generated intermediate data, not authoritative input.

The lesson I keep returning to: **put meaning in folder boundaries, not just filenames.** A filename tells you what a thing is called. A folder path tells you what role it plays. The role is what the agent needs.

One limit worth naming: exact folder names are convention, not law. If your repo already uses `output/` instead of `outputs/`, or `dist/` because it is a software build system, keep it. What matters is that the roles are consistent and documented somewhere the agent can find — typically `_MANIFEST.md`. The convention I use here is a default, not a requirement.

## The Files That Live at the Root

Four files belong at the project root, not in subfolders. Each does a specific job.

`_PROJECT_INDEX.md` is the entry point. If you manage several projects, this file at the workspace level tells an agent which project is active without opening any project folder. A simple table: project name, status, last updated, one-line state, path. That is enough for an agent to route itself without reading everything.

`AGENTS.md` is the instruction set for automated tools. Any agent that reads this project should read `AGENTS.md` first. It answers: what should I do here, what should I not touch, where do I write output, what formats do I use.

`_MANIFEST.md` is the map. It explains the folder structure for this specific project — what lives where, why, what naming conventions are in use. If you diverged from the default structure for good reasons, this is where you say so.

`PROJECT_RULES.md` is the constraint layer. Prose conventions, output standards, decisions that have already been made and should not be reopened. An agent that reads this before acting will not ask questions you have already answered.

![Root instruction files](images/02-the-greenfield-project-what-clean-looks-like-fig-02.png)
*Figure 2.2 - Root instruction files*

<!-- → [DIAGRAM: Four files at project root, each with a one-sentence role label and an arrow indicating which entity reads it — AGENTS.md and PROJECT_RULES.md read primarily by agents; _PROJECT_INDEX.md and _MANIFEST.md read by both human and agent. Caption: Root files are the project's instruction layer. They exist to answer questions before they get asked.] -->

The distinction between `AGENTS.md` and `PROJECT_RULES.md` is worth making explicit. `AGENTS.md` is operational — it says what to do and how. `PROJECT_RULES.md` is editorial — it says what the project has decided about its own standards. A book project might have both: `AGENTS.md` that says "write chapters to `current/`, run `/done` after writing," and `PROJECT_RULES.md` that says "Feynman register, ~3,000 words per chapter, no learning objectives blocks."

## Git and the Generated Layer

Here is a confusion I see constantly: people treat version control as a security boundary.

It is not. `.gitignore` keeps files out of git history. It does not keep a local agent from reading those files. That distinction matters because the right reason to gitignore something is to keep generated files from cluttering your history — not to hide them from your tools.

The generated layer belongs in `.gitignore` for cleanliness, not secrecy:

```gitignore
build/
outputs/
output/
.ai/audit/
.ai/sessions/
__pycache__/
*.pyc
node_modules/
```

`build/` and `outputs/` are regenerable. Tracking them in git adds noise — every time you rebuild, you get a diff that says "outputs changed" without telling you anything meaningful about what changed in your source. Keep source clean; let generated artifacts be what they are.

The `.ai/` folder is worth discussing separately. `audit/` and `sessions/` are logs of what agents have done — useful locally for debugging, meaningless in a shared repo. The schemas and workflows in `.ai/schemas/` and `.ai/workflows/` are different: those are part of your project's configuration and probably should be tracked.

## The Workspace Index

If you run multiple projects — and most people using AI-assisted workflows eventually do — you need one more thing above the project level: a workspace index.

```markdown
# _PROJECT_INDEX.md

| Project | Status | Updated | One-line state | Path |
|---|---|---|---|---|
| cli-tooling-book | active | 2026-06-14 | drafting chapters | projects/cli-tooling-book/ |
| data-pipeline | paused | 2026-06-01 | awaiting review | projects/data-pipeline/ |
```

That table answers the question "which project is live right now?" without opening any project folder. For an agent navigating a workspace with six projects, it is the difference between reading one file and reading sixty.

| Project | Status | Updated | One-line state | Path |
|---|---|---:|---|---|
| CLI tooling book | Active | 2026-06-14 | Drafting chapters and figures. | books/cli-agnostic-ai-tooling-for-local-project-workflows/ |
| Branding toolkit | Paused | 2026-05-28 | Waiting for prompt audit. | books/branding-and-ai/ |
| Legacy notes | Archived | 2025-12-18 | Historical source only. | archive/legacy-notes/ |
| Evaluation harness | On hold | 2026-04-03 | Needs benchmark data. | projects/eval-harness/ |

*The workspace index is a routing table for agents.*

The "one-line state" column is the one I get wrong most often. The temptation is to write something polished: "Developing AI-assisted CLI workflow textbook." That is a description, not a state. What I actually need is something that orients an agent returning to the project after a gap: "drafting Chapter 4, waiting for source from Nik" or "blocked on figure revisions." The state column is a handoff note, not a label.

## What Breaks Without This

Let me be direct about what actually goes wrong when projects are not structured this way, because the failure mode is subtle.

The agent does not crash. It does not tell you something went wrong. It uses the wrong source, and produces output that looks correct — that has the right format, the right length, the right tone — but is built on a file you thought you had retired.

![Silent staleness timeline](images/02-the-greenfield-project-what-clean-looks-like-fig-03.png)
*Figure 2.3 - Silent staleness timeline*

<!-- → [DIAGRAM: Timeline showing three phases — Week 1: agent uses correct source file (report.md); Week 3: report-v2.md added to flat folder; Week 4: agent uses report.md (stale) instead of report-v2.md — output looks fine but is wrong. Caption: The failure mode is not noise, it is silent staleness. The agent produces plausible output from the wrong source because it cannot tell which file is live.] -->

I have had this happen on a chapter draft. The agent retrieved an archived version instead of the current one, made the revisions I asked for, and returned something polished. It took me a paragraph before I realized none of the new material was in there. The agent had done exactly what I asked. It had done it to the wrong file.

Clean structure does not prevent every error. But it makes this particular category of error almost impossible, because the roles are unambiguous before the agent reads anything.

## The Setup Command

Here is the thing I actually paste when starting a new project. One bash command creates the entire structure:

```bash
mkdir -p .ai/{workflows,schemas,sessions,audit}
mkdir -p src current research templates staging build outputs archive
touch _PROJECT_INDEX.md AGENTS.md _MANIFEST.md PROJECT_RULES.md status.md session-handoff.md .ai/manifest.yaml
```

Run it once. Then populate the files that need content before an agent touches the project: `AGENTS.md`, `_MANIFEST.md`, `PROJECT_RULES.md`. Leave the folders empty if they are empty. The structure is the thing; the files follow.

The misconception I most often have to correct: "this is too much for a small project." Most of these folders will stay empty for most projects. That is fine. Empty folders with meaningful names still communicate roles. And the cost of creating them is one bash command. The cost of retrofitting a messy project is much higher.

## What Would Change My Mind

I would revise this chapter if agent tools adopted a reliable, cross-vendor project metadata standard that made folder role names unnecessary. If every major CLI agent recognized a `pyproject.toml`-style config that declared file roles explicitly, the folder naming convention would become a fallback rather than the primary mechanism. Today, no such standard exists at useful breadth. Plain folder structure remains the most portable metadata layer — it works with every tool that reads files, because every tool that reads files can read a path.

## Still Puzzling

Should `.ai/` become a real community convention, the way `.github/` became standard for repository configuration? There is an obvious argument for it — a shared namespace for agent workflow metadata would make tooling interoperable across projects. The counterargument is that premature standardization locks in assumptions before anyone knows which assumptions are right.

Whether `current/` is useful in software repos is genuinely unclear to me. In writing and research projects, the distinction between "active draft" and "canonical source" is meaningful. In a software repo with good branch discipline, `current/` may be redundant — the branch *is* the signal. The folder might be more useful in projects where git is not the versioning mechanism.

And the build system naming problem has no clean solution I can see. When your build system already uses `dist/`, or `target/`, or `out/` with defined semantics, adding `outputs/` for agent-generated deliverables creates a two-namespace situation that requires documentation to navigate. The honest answer is: document it in `_MANIFEST.md` and accept the friction.

---

## Exercises

**Warm-up**

1. *(Recall)* Name the four root-level files from this chapter and give a one-sentence description of what each one is for. *Tests whether you can distinguish between the operational instruction layer (AGENTS.md), the map (_MANIFEST.md), the constraint layer (PROJECT_RULES.md), and the entry point (_PROJECT_INDEX.md).*

2. *(Identification)* Given a flat folder containing `report.md`, `report-v2.md`, `data.json`, `prompt.txt`, and `notes.md`, assign each file to the correct subfolder in the clean structure from this chapter. Explain your reasoning for any file where the assignment is ambiguous. *Tests your ability to read role signals in filenames and make principled decisions under ambiguity.*

3. *(Recall)* What is the difference between what `.gitignore` does and what folder naming conventions do? Why does that distinction matter for agent workflows? *Tests comprehension of the security-vs.-cleanliness distinction.*

**Application**

4. *(Translation)* You have a project with the following files in a flat folder: `analysis.py`, `analysis_old.py`, `cleaned_data.csv`, `raw_data.csv`, `viz.html`, `final_report.pdf`, `scratchpad.md`, `requirements.txt`. Create the clean folder structure for this project. For each file, state its destination and a one-sentence justification. *Tests your ability to apply folder-as-metadata reasoning to a software/data hybrid project.*

5. *(Writing)* Write the content for `AGENTS.md` for a project where an AI assistant is drafting sections of a technical book. The agent should write drafts to `current/`, treat `research/` as read-only input, never touch `archive/`, and deliver finished chapters to `outputs/`. *Tests whether you can translate project constraints into agent instructions.*

6. *(Analysis)* A teammate argues that `_PROJECT_INDEX.md` is unnecessary overhead for a solo project. Write a two-paragraph response that either agrees or disagrees, with specific reasoning from the chapter. *Tests ability to evaluate the tradeoff between structure cost and retrieval benefit.*

**Synthesis**

7. *(Design)* You are starting a research project that will involve an AI agent collecting and summarizing papers, a human writer drafting arguments, and a build pipeline that generates a formatted PDF. Design the complete folder structure. Identify which folders each actor (agent, human, build system) reads from and writes to. Note any naming conflicts with existing conventions you anticipate and how you would document them. *Tests integration of all three concepts: folder roles, root files, and workspace indexing.*

8. *(Retrofit)* You have been handed a colleague's project with 47 files in a flat folder. Describe your process for retrofitting it into clean structure. What do you do first? What is the hardest decision you will face? What do you put in `_MANIFEST.md` after the retrofit is complete? *Tests ability to apply the framework to messy real-world conditions rather than greenfield setup.*

**Challenge**

9. *(Open-ended)* This chapter argues that folder structure is metadata the agent reads before reading content. But some agent frameworks — particularly those using vector embeddings over flat file stores — do not read path structure at all; they retrieve by semantic similarity. Does the folder-as-metadata argument still hold for those systems? If not, what replaces it? If so, why? *Engages the chapter's deepest contestable claim: that folder structure is the most portable metadata layer available.*

## Prompts

### Figure 2.1 - Flat folder vs. structured project tree

Create a standalone D3 v7 HTML figure for "Flat folder vs. structured project tree" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: Single flat folder on the left containing 8 unlabeled files vs. structured folder tree on the right with current/, outputs/, archive/ — arrow from each right-side file to its role label. Caption: The same eight files, two folder strategies. The left gives the agent no signal before it reads; the right answers the quest Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 2.2 - Root instruction files

Create a standalone D3 v7 HTML figure for "Root instruction files" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: Four files at project root, each with a one-sentence role label and an arrow indicating which entity reads it — AGENTS.md and PROJECT_RULES.md read primarily by agents; _PROJECT_INDEX.md and _MANIFEST.md read by both human and agent. Caption: Root files are the project's instruction layer. They exist to answer question Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 2.3 - Silent staleness timeline

Create a standalone D3 v7 HTML figure for "Silent staleness timeline" using the pinned CDN. Use a timeline / progression structure with 4-6 labeled elements inferred from this concept: Timeline showing three phases — Week 1: agent uses correct source file (report.md); Week 3: report-v2.md added to flat folder; Week 4: agent uses report.md (stale) instead of report-v2.md — output looks fine but is wrong. Caption: The failure mode is not noise, it is silent staleness. The agent produces plausible outpu Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
