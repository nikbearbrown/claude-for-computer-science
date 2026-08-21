# Introduction

Clone a repository you have never seen, open it in a terminal, and type `ls`. In the first second of output, before you have read a single file, you already know something about the person who built it. A folder with `src/`, `current/`, `archive/`, and a `README.md` at the top tells you where the work is and where the history went. A folder with `final.md`, `final2.md`, `final-FINAL.md`, and `notes(1).md` tells you that someone kept postponing a decision — and that you are now going to have to make it for them.

An AI agent reads that same listing and reaches the same kind of conclusion, except it cannot ask you which file you meant. It infers. It picks. And when it picks wrong, the answer it gives you will not look wrong. It will look finished.

That is the problem this book is about, and the surprising thing is where the solution lives. It does not live in a cleverer prompt. It lives in the shape of the folder.

## The central claim

When an agent works on your local files, **the filesystem is part of the prompt.** The names of your folders, the depth of your paths, which files sit beside which, what is archived and what is current — all of it is context the agent reads before it reads your instruction, and all of it shapes the answer. The durable skill in working with AI on real projects is not prompt-crafting. It is building a project that explains itself to any tool you point at it.

"Any tool" is the other half of the claim. The agent landscape changes monthly — Claude Code, Cursor, Aider, Codex, Gemini, Copilot, and whatever ships next quarter each read a different instruction file, each with its own quirks. A workflow welded to one vendor's features is a workflow with an expiration date. So everything here is built to survive a tool change: the durable intent lives in plain files you control, and each tool gets a thin adapter that points at them. That is what *CLI-agnostic* means — the conventions work in a terminal, in any tool that reads a terminal's files, and in the one you will switch to next year.

## The basic principles

The chapters expand and qualify these, but the whole book rests on a short list. If you internalize nothing else, internalize this:

1. **The filesystem is part of the prompt.** Folder names, paths, and adjacency are context the agent reads first. Shape them deliberately.
2. **Structure communicates before instructions do.** A clean `current/` and a quiet `archive/` tell the agent which file is live without a single word of prompt. A good folder is a prompt the agent cannot miss.
3. **Context is finite, and composition matters more than size.** More material is not more help; past a point it is noise that buries the signal. Load the smallest set of high-signal files the task needs.
4. **Separate the canonical from the transient.** Source, active drafts, generated output, and retired work are four different things. When they live in one flat folder, the agent cannot tell them apart — and neither can you, six months later.
5. **Instructions are advisory; enforcement is structural.** A rule in a markdown file is a strong suggestion, not a circuit breaker. For anything irreversible you need a hook, a gate, a test, or a human — not a more strongly worded sentence.
6. **Never delete; archive.** Prefer reversible operations. Moving a file to `archive/` costs nothing and undoes cleanly. Deleting the one copy you had not committed yet does not.
7. **One source of truth, thin adapters.** Write your rules once, in a tool-neutral file, and let each tool read from it. Four hand-maintained copies will drift; one canonical file cannot.
8. **Measure; do not assume.** A rule that *sounds* wise still costs context every session. Keep the rules that change behavior and prune the ones that only make you feel organized.

None of these is exotic. Together they are the difference between an agent that quietly uses the wrong file and one that reaches for the right one before you have finished typing.

## What you are actually looking at

Because this is a book you can put to work immediately, here is the shape it keeps returning to — the layout that, when you `ls` it in a terminal, answers the agent's questions before they are asked:

```text
project/
├── AGENTS.md          # instructions any tool should read first
├── _MANIFEST.md       # the map: what's canonical, what to ignore
├── status.md          # where the project stands right now
├── current/           # active work — the live drafts
├── src/               # canonical source
├── research/          # evidence and notes, not deliverables
├── outputs/           # finished deliverables, not source
├── build/             # generated, regenerable, never edited by hand
└── archive/           # superseded work — ignored unless asked
```

Every name is a role the agent can infer. `current/` is live. `archive/` is history. `outputs/` is finished. `build/` is disposable. You are not memorizing a standard — there isn't one — you are using names that mean the same thing to a person, a script, and a language model. By the end of Chapter 2 you will be able to create this skeleton with a single command and explain why each folder earns its place.

## A note on where these ideas come from

The conventions in this book are deliberately tool-agnostic and lightly opinionated — meant to work everywhere and offend no one. They are the portable distillation of a deeper, more opinionated framework I have written about separately: the **Snickerdoodle principles**, an agent-operating-system that treats a project as a contract between human judgment and AI execution, with named principles, hard gates, a recipe lifecycle, and an attestation discipline. Where this book says "prefer reversible operations" and moves on, that framework names the gate, specifies who clears it, and requires that it be logged. If after reading this you want the strict, governed version — the one that makes the rules enforceable rather than advisory — that detailed treatment is the place to go. This book is the on-ramp; Snickerdoodle is the system.

## How this book is organized

The chapters move from the smallest unit — a folder — outward to the hardest problems: governance and measurement. Each is self-contained enough to read out of order if you have a specific problem, but they build.

- **Chapter 1 — Why Your Agent Reads the Wrong File.** Why the filesystem is part of the prompt, and how weak signals in a messy folder lead an agent to confident, wrong answers.
- **Chapter 2 — The Greenfield Project.** What a clean project looks like from scratch, and why folder structure is metadata the agent reads before content.
- **Chapter 3 — The Manifest.** Writing a small index that tells the agent what to read first, what to reach for, and what to ignore.
- **Chapter 4 — Project Rules and Instruction Precedence.** Why instruction files are not universal, the difference between loading a rule and enforcing it, and how to keep one source of truth across tools.
- **Chapter 5 — The Five Context Cost Types.** Where your tokens actually go — fixed, per-task, accumulated, tool-output, and connector cost — and the distinct fix each one needs.
- **Chapter 6 — Session Hygiene and State Files.** Why long sessions degrade, and how `status.md` and a session handoff let you start fresh without losing the thread.
- **Chapter 7 — Compaction, Clearing, and Starting Fresh.** Three different interventions for a tired session, and how to tell which one a situation calls for.
- **Chapter 8 — One Folder, Many Tools.** Making the durable parts of a project readable by any tool, with thin adapters for the parts that aren't portable.
- **Chapter 9 — MCP Servers and Context.** The silent tax every connected tool charges before you type, and how to scope the tool surface to the task.
- **Chapter 10 — Agent Workflow Patterns That Actually Work.** Builder-validator, planner-executor-reviewer, staging-before-output, and when the overhead of role separation is worth it.
- **Chapter 11 — File Safety: Never Delete, Always Archive.** Why "just use git" is not enough, and how to make destructive operations structurally hard instead of merely discouraged.
- **Chapter 12 — The Anti-Patterns Checklist.** Eleven recurring failures, the boundary each one violates, and the structural repair for each.
- **Chapter 13 — The Failure-Mode Catalog.** Diagnosing agent failures by pattern, trigger, and mitigation instead of stopping at the symptom.
- **Chapter 14 — Governance and Security.** What changes the moment an agent has shell access: prompt injection, tool poisoning, least privilege, and treating MCP servers as third-party code.
- **Chapter 15 — Does Any of This Actually Help?** How to measure whether your setup earns its cost, with a small benchmark that prunes what doesn't.

## How to read it

If you are new to working with AI agents on local files, read in order — the early chapters build the vocabulary the later ones assume. If you arrived with a specific problem, jump to its chapter: a confused agent is Chapter 1, a messy project is Chapter 2, a slow session is Chapters 5 through 7, a deletion scare is Chapter 11, an agent with shell access is Chapter 14. Every chapter ends with two things worth not skipping: a *What Would Change My Mind* section that states honestly where the advice is provisional, and exercises — many designed to be run with a live agent open, because the fastest way to believe any of this is to watch an agent behave differently when you change the folder rather than the prompt.

## Getting started: four things to try before Chapter 1

These take a few minutes each and need nothing but a terminal and an agent you already use. They are here so the ideas land as experience, not assertion.

1. **Read a folder the way an agent does.** Pick a real project. Run `ls -R` (or open the tree). For each file, ask: *if I had no memory of this project, which file would I think is the current, canonical one?* Count how many you genuinely could not tell apart. That number is how much ambiguity your agent is resolving by guessing.

2. **Ask the agent to choose.** Paste that same directory listing into your agent and ask: *"Which file would you read first to summarize this project's current state, and why?"* Compare its reasoning to yours. Where it guessed wrong, notice that the cause was almost never the model — it was the folder.

3. **Build the skeleton.** In an empty directory, run:
   ```bash
   mkdir -p src current research outputs build archive
   touch AGENTS.md _MANIFEST.md status.md
   ```
   You have just created a project that explains itself. Most of these folders will stay empty for a while. That is fine — an empty folder with a meaningful name still tells the agent a role.

4. **Write one sentence the agent will obey.** Open `AGENTS.md` and write a single line: `Read _MANIFEST.md first. Never delete files — move them to archive/ instead.` It will not be perfectly enforced, and Chapter 11 explains why. But notice what writing it did: it forced you to decide what you actually want the agent to know. That, more than the sentence itself, is the work.

The rest of the book is these four moves, made deliberate and made to scale. Start with the folder. The prompt can wait.

![The book moves from project structure to context control to repeatable multi-tool workflows.](images/01-introduction-fig-01.png)
*Figure 1.1 - Book Argument Roadmap*

## Prompts

### Figure 1.1 - Book Argument Roadmap

Create a standalone D3 v7 HTML figure for "Book Argument Roadmap" using the pinned CDN. Use a timeline / progression structure with 4-6 labeled elements inferred from this concept: The book moves from project structure to context control to repeatable multi-tool workflows. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
