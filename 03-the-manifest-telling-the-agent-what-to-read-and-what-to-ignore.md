# Chapter 3 — The Manifest: Telling the Agent What to Read and What to Ignore
*The project without a map is just a pile of files waiting to mislead.*

Here is something that happens more than it should. You ask an agent to find the latest source for a claim. It starts scanning. It reads the active chapters folder, then the pantry folder, then the archive because archive is just another directory and all directories look the same to it. Forty minutes later it has summarized something from a note you wrote two years ago and superseded three times. The session is heavier, slower, and the output is wrong.

Nothing went wrong with the intelligence. What went wrong was that the agent had no idea which files matter and which files are graveyard.

That is the problem a manifest solves — not by enforcing anything, not by locking files away, but by giving the agent a map at the start of every session. A map that says: read these first, reach for these when the task calls for it, and leave those alone unless I ask you specifically to go there.

## What a Manifest Actually Is

A manifest is a small index. One file. Usually a few dozen lines.

It does not store content. It does not replace your README. It does not explain the project history or describe your methodology. All of those things belong somewhere else. A manifest describes the *role* of each major file or folder in terms an agent can act on. If you have to scroll to read the whole thing, it is already too long.

The distinction matters because agents have token budgets. Every file they read during a session occupies space that could have been used for actual work. A large manifest that tries to be comprehensive punishes you the same way a missing manifest does — by front-loading context the agent did not need yet. The goal is just-in-time context: the agent knows where to look, and it looks only when the task calls for it.

![Three-tier manifest access model](images/03-the-manifest-telling-the-agent-what-to-read-and-what-to-ignore-fig-01.png)
*Figure 3.1 - Three-tier manifest access model*

<!-- → [DIAGRAM: Three-tier manifest structure — visual hierarchy showing Tier 1 (always read), Tier 2 (task-triggered), Tier 3 (ignored unless requested); annotate with token-budget rationale; caption: "The manifest controls what enters the context window and when."] -->

The pattern is not new. It appears in every serious tool that connects agents to large file systems. Anthropic Skills separate small metadata headers from detailed resources, loading the latter on demand. Claude Code memory uses compact entry points that fan out to detail only when needed. Aider's repo map gives the model a token-budgeted index of files and symbols before it ever reads source code. `AGENTS.md` is converging toward a portable instruction standard for agents across different tools. What you are building when you write a manifest is a lightweight version of the same idea, without vendor lock-in.

## The Three-Tier Structure

The most useful manifest I have found organizes everything into three tiers based on one question: *when should the agent reach for this?*

**Tier 1 — Canonical.** Read this at the start of every session, no matter what. These are the files that define ground truth for the project: the rules, the current state, the structure. An agent that starts a session without reading these is working without context. Usually three to five files. Never more than ten.

**Tier 2 — Task-relevant.** Read this when the task calls for it. Folders or files that matter for specific kinds of work — the active chapters folder when editing, the research folder when checking evidence, the templates folder when generating repeatable output. The agent does not preload these; it reaches for them when the task requires.

**Tier 3 — Ignore unless requested.** Archive folders. Build artifacts. Generated outputs. These exist and have value, but they are not source material for active work. Without explicit instruction from the archive, the agent should not go there. This tier is where most of the project's mass lives, and the manifest's job is to keep it quiet.

```markdown
# _MANIFEST.md - read this first

## Tier 1 - Canonical
| File | Purpose |
|---|---|
| PROJECT_RULES.md | Project-specific rules |
| status.md | Current state |
| outline.md | Structure/source of truth |

## Tier 2 - Task Relevant
| Folder | Use when |
|---|---|
| current/ | Editing active material |
| research/current/ | Checking evidence |
| templates/ | Creating repeatable outputs |

## Tier 3 - Ignore Unless Requested
| Folder | Rule |
|---|---|
| archive/ | Ignore unless explicitly requested |
| build/ | Generated, not source |
| outputs/ | Deliverables, not working source |
```

| Step | Without manifest | With manifest | Effect |
|---|---|---|---|
| Start | Agent scans broadly. | Agent reads canonical entry files. | Less irrelevant context. |
| Source selection | Archive and drafts compete. | Active sources are named. | Fewer wrong-file errors. |
| Evidence lookup | Whole folders are sampled. | Task-relevant folders are used. | Lower token cost. |
| Output | Plausible but brittle. | Grounded in declared sources. | Higher review confidence. |

*A manifest turns broad search into staged retrieval.*

Notice what this does not include. It does not explain why the project exists. It does not list individual files within folders. It does not describe what goes *into* each file — that is the file's own job. The manifest only names roles and access rules. Everything else is noise.

## A Concrete Case

The task arrives: "Update Chapter 4 with current sources."

Without a manifest, the agent scans everything it can find:

```text
chapters/
pantry/
archive/
outputs/
notes-old/
```

It does not know that `archive/` holds superseded drafts. It does not know that `outputs/` is generated material, not source. It reads all of it because all of it looks like potentially relevant context. By the time it reaches Chapter 4, it is carrying several thousand tokens of context from files that have nothing to do with the task.

With a manifest, it reads:

```text
TIKTOC.md
status.md
pantry/04-project-rules-and-instruction-precedence_notes.md
chapters/04-project-rules-and-instruction-precedence.md
```

That is enough. The archive stays quiet. The agent reaches `chapters/04-...` with a clear picture of the current project state and the specific research that supports it — and nothing else.

The manifest did not prevent the agent from being wrong. It prevented the agent from being wrong *because it looked in the wrong place*.

## The Machine-Readable Twin

The Markdown manifest is for human readability and agent sessions. But some tools work better with structured metadata than with prose. A YAML twin gives you the same information in a form that can be parsed, validated, and eventually integrated with tooling:

```yaml
name: project-alpha
canonical:
  - PROJECT_RULES.md
  - status.md
  - outline.md
task_relevant:
  - current/
  - research/current/
  - templates/
ignore:
  - archive/
  - build/
  - outputs/
entrypoints:
  test: "npm test"
  build: "npm run build"
invariants:
  - "Never treat outputs/ or build/ as source."
  - "Move superseded files to archive/, never delete."
```

| Property | _MANIFEST.md | .ai/manifest.yaml | Tradeoff |
|---|---|---|---|
| Primary reader | Human and agent | Tooling and agent | Markdown is legible; YAML is parseable. |
| Best use | Session start and review | Automation and validation | Use both when workflow maturity justifies it. |
| Failure mode | Too verbose | Too brittle | Keep each representation small. |
| Update rhythm | Manual with project changes | Generated or linted | Avoid divergent truth sources. |

*Two representations of the same index serve different readers.*

The `invariants` field is worth noting. These are statements that should remain true throughout the project, regardless of what any single session does. They belong in both documents. An agent that reads the invariants before acting is less likely to write into `outputs/` as if it were source, or to delete files from `archive/` because they look redundant.

Keep both files. They serve different readers.

## What a Manifest Cannot Do

This is where I have to be careful, because the manifest is a popular idea and the temptation is to treat it as enforcement.

It is not. A manifest informs behavior; it does not enforce it. If you write `archive/: ignore unless explicitly requested`, the agent will treat that as a strong contextual signal — but not a lock. An agent with sufficient reason, or an ambiguous instruction, can still reason its way into the archive. The manifest lowers the probability of that mistake; it does not eliminate it. Enforcement requires something the manifest does not provide: permissions, hooks, tests, wrappers, or a human in the loop.

The manifest should also not try to list every file. Projects change. Files move. If the manifest lists `chapters/04-chapter-four.md` and that file gets renamed, the manifest now points to a ghost. Role descriptions survive longer than file paths. Write `chapters/: active chapter drafts` rather than enumerating each one.

And most importantly: a manifest is not finished when you write it. A living project has a living manifest. Every time a canonical file changes, every time a folder's role shifts, every time you move something to archive — the manifest should reflect it. A stale manifest does something worse than no manifest: it confidently points the agent in the wrong direction.

## Three Misconceptions Worth Naming

**"The manifest enforces behavior."** No. It informs behavior. The distinction matters practically: if you are relying on the manifest to *prevent* the agent from doing something, you are using the wrong tool.

**"The manifest should list every file."** No. It should list roles and canonical entry points. The moment you start cataloging individual files, you have turned the manifest into maintenance debt.

**"Once written, it is done."** No. A manifest is a living index. The failure mode is not writing a bad one — it is writing a good one and then never touching it again.

## What Would Change My Mind

I would revise this chapter if tools converged on an enforceable cross-vendor manifest standard with reliable ignore semantics. Until then, Markdown plus YAML is the most portable compromise.

## Still Puzzling

- Should `_MANIFEST.md` and `.ai/manifest.yaml` be generated from one source?
- How often should a manifest be audited?
- Can tools warn when a manifest points to missing files?

---

## LLM Exercises

> These exercises are designed to be completed with a live language model session open. The goal is not to find the right answer in isolation — it is to observe how the model behaves differently when context changes.

**Exercise 1: Map a real project.** Open a project directory you work in regularly. Write a `_MANIFEST.md` for it from scratch using the three-tier structure. Do not look at the chapter while you write it. When you are done, compare what you produced to the structure described here. What did you include that the chapter would not? What did you leave out?

**Exercise 2: Stress-test Tier 3.** Take the manifest you wrote in Exercise 1. Move one folder from Tier 2 to Tier 3 and write one sentence explaining why. Then start a session with an agent and give it a task that would previously have required that folder. Observe whether the agent reaches for it anyway, and if so, why.

**Exercise 3: Write the YAML twin.** Convert your `_MANIFEST.md` into a `.ai/manifest.yaml`. Add at least two `invariants` — statements that must remain true about your project regardless of what any session does. Test one: give the agent an instruction that would violate an invariant and see whether it flags the conflict.

---

![A manifest reduces context waste by turning a broad search into a staged retrieval process.](images/03-the-manifest-telling-the-agent-what-to-read-and-what-to-ignore-fig-02.png)
*Figure 3.2 - Unguided vs Manifest-Guided Session*

## Exercises

**Warm-up**

1. *(Basic recall)* Name the three tiers of a manifest and the rule governing each. For each tier, give one example of a file or folder that belongs there in a typical writing project. *What this tests: whether you can reproduce the structure without looking.*

2. *(Vocabulary)* What is the difference between a manifest that "informs" behavior and one that "enforces" it? Give a concrete example of a situation where that distinction matters. *What this tests: understanding of the manifest's actual scope.*

3. *(Comparison)* In what way does a project manifest resemble Aider's repo map? In what way is it different? *What this tests: ability to locate the manifest within the broader landscape of agent tooling.*

**Application**

4. *(Design)* You are starting a research project with the following folders: `drafts/`, `sources/`, `old-versions/`, `final-submission/`, `notes/`. Write a `_MANIFEST.md` that assigns each folder to a tier and includes one canonical file. Justify each placement in one sentence. *What this tests: applying the three-tier structure to a real scenario.*

5. *(Failure mode)* You wrote a manifest six months ago. Since then, the file `outline.md` was renamed `structure.md`, and a new folder `review-drafts/` was added for content under active peer review. What are the specific failure modes created by the stale manifest? How would you fix each one? *What this tests: understanding of staleness as the primary failure mode.*

6. *(Tradeoffs)* A colleague argues that the YAML manifest is redundant — everything in it is already in `_MANIFEST.md`. Make the strongest case for keeping both. Then make the strongest case for dropping one. Which do you actually recommend, and why? *What this tests: reasoning about the tradeoffs between human-readable and machine-readable formats.*

**Synthesis**

7. *(Integration)* Chapter 2 introduced the concept of project rules. How does `PROJECT_RULES.md` belong in a manifest, and what happens if the rules and the manifest give conflicting guidance to an agent? Sketch a resolution protocol. *What this tests: connecting manifest structure to the broader instruction hierarchy.*

8. *(Design extension)* The manifest as described has no mechanism for versioning. A file promoted from Tier 2 to Tier 1 three months ago might now be obsolete again. Propose a lightweight convention — one that does not require a tool — for tracking when a manifest entry was last reviewed. *What this tests: extending the framework to handle long-running projects.*

**Challenge**

9. *(Open-ended)* The chapter claims that enforcement requires "permissions, hooks, tests, wrappers, or a human in the loop." Design a minimal enforcement layer for one of those mechanisms — your choice — that a solo practitioner could realistically implement without dedicated infrastructure. What does it protect against? What does it leave open? *What this tests: ability to move from description to implementation, and honest accounting of what remains unresolved.*

## Prompts

### Figure 3.1 - Three-tier manifest access model

Create a standalone D3 v7 HTML figure for "Three-tier manifest access model" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: Three-tier manifest structure — visual hierarchy showing Tier 1 (always read), Tier 2 (task-triggered), Tier 3 (ignored unless requested); annotate with token-budget rationale; caption: "The manifest controls what enters the context window and when." Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 3.2 - Unguided vs Manifest-Guided Session

Create a standalone D3 v7 HTML figure for "Unguided vs Manifest-Guided Session" using the pinned CDN. Use a comparison panels structure with 4-6 labeled elements inferred from this concept: A manifest reduces context waste by turning a broad search into a staged retrieval process. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
