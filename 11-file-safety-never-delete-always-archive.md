# Chapter 11 — File Safety: Never Delete, Always Archive
*How a single ambiguous verb can erase work that git never saw, and why the only robust answer is making deletion structurally impossible.*

The instruction was reasonable. The project had accumulated files over several weeks — drafts, experiments, notes that had been superseded. "Clean up the project" is the kind of thing you might ask a capable assistant. It is also the kind of thing that, when an agent executes it, can delete the only copy of a file you had not yet committed.

The dangerous word was not "delete." The word "delete" never appeared. The dangerous word was "clean."

Agents are good at executing ambiguous verbs. That is precisely the problem. "Clean," "tidy," "simplify," and "remove clutter" all have natural interpretations that involve making files disappear. The agent is not being careless — it is doing what the instruction implied. The problem is that the instruction implied something you did not fully intend, and the operation is irreversible.

Understanding file safety for agent workflows means understanding this asymmetry: agent execution is fast and uninhibited, while the consequences of a destructive file operation are permanent. The solution is not to make agents slower. It is to make certain operations structurally impossible before they get to the execution stage.

## Why "Just Use Git" Is Not Enough

The instinct is to say: this is a solved problem. Git exists. If the agent deletes something, check out the old version.

This works for files that were tracked and committed. It does not work for the file you were about to commit. It does not work for the research note you had been accumulating for three days and had not yet staged. It does not work for the generated artifact that took forty minutes to build and is not in the repo because you added it to `.gitignore`.

Git is a recovery layer for a specific subset of your project: the subset that has been explicitly added to version control and has at least one commit. That subset is often smaller than people assume, because the workflow of AI-assisted development tends to produce a lot of working material that is ahead of commits — exactly the material most likely to be "cleaned up."

![Git protection vs. project contents](images/11-file-safety-never-delete-always-archive-fig-01.png)
*Figure 11.1 - Git protection vs. project contents*

<!-- → [DIAGRAM: Venn diagram with two circles. Left circle: "Files in the project folder." Right circle: "Files recoverable from git." The overlap is labeled "tracked and committed." Outside the right circle but inside the left: "untracked files, uncommitted changes, .gitignored files, generated artifacts." Caption: Git protects the overlap. File safety policy has to protect everything else.] -->

The implication is that git is necessary but not sufficient. A file safety policy that relies on git for recovery has a gap that is exactly the size of your working state at any given moment.

## The Default Rule

The rule I use and recommend encoding in `AGENTS.md` for any project where an agent might touch files:

```text
Never permanently delete files. Move them to archive/ instead.
Ask before bulk renames, overwrites of canonical files, or structural moves.
```

Two sentences. The first covers the core case — file removal. The second covers the high-blast-radius operations that are not technically deletions but have similar consequences: renaming twenty files at once, overwriting a file that is a canonical source with a generated version, reorganizing a folder structure without approval.

The archive destination is intentional. Moving a file to `archive/` is reversible. The file is still there. The agent has not destroyed anything; it has relocated it. If the relocation was wrong, you move it back. The cost of getting it wrong is a file move, not a recovery operation.

This is the operational version of a principle that comes up elsewhere in engineering: prefer reversible operations. The preference is not because irreversible operations are always wrong — sometimes you genuinely want to delete something — but because the decision to make an operation irreversible should be deliberate, not implicit in an ambiguous verb.

## The Difference Between Advisory and Deterministic

Writing the rule in `AGENTS.md` is the floor, not the ceiling. An instruction file is advisory: it tells the agent what behavior is preferred. Whether the agent follows it consistently is a function of how clearly the instruction is written, how much competing context there is, and the specifics of what the agent was asked to do.

For a rule you actually care about, you want a deterministic guard — something that blocks the operation independently of whether the agent decided to follow the instruction.

Claude Code supports this through its hooks system. A `PreToolUse` hook runs before a tool call is executed. You can write a hook that intercepts any `Bash` tool call matching `rm` and either blocks it, redirects it to a move operation, or escalates it to human approval.

A no-delete hook can be permissive where permissiveness makes sense. Regenerable targets — `build/`, `__pycache__/`, `*.pyc`, `*.bak` — are safe to delete because nothing is lost. The hook allows `rm` against those patterns and denies it against everything else: source files, data files, logs, chapters, anything that represents human or accumulated work.

![No-delete hook decision flow](images/11-file-safety-never-delete-always-archive-fig-02.png)
*Figure 11.2 - No-delete hook decision flow*

<!-- → [DIAGRAM: Flowchart showing a file operation arriving at a PreToolUse hook. Decision diamond: "Does path match regenerable pattern?" If yes: "Allow." If no: second decision diamond: "Is this a move to archive/?" If yes: "Allow." If no: "Block and escalate to human approval." Caption: A no-delete hook intercepts destructive operations before execution. The logic is simple: regenerable targets can be deleted; everything else is moved or escalated.] -->

The limit is that hooks are tool-specific. A Claude Code `PreToolUse` hook does not exist in Aider. A shell wrapper that intercepts `rm` commands works in any environment where you control the shell, but requires setup and has its own failure modes. The principle — deterministic guard, not advisory instruction — is portable. The implementation is not, which is why the instruction file is still necessary as the portable baseline.

## How to Ask for Destructive Work Safely

The problem with "clean up the project" is that it gives the agent both the judgment call and the execution authority simultaneously. The fix is to separate those.

Unsafe:

```text
Clean up this folder.
```

This asks the agent to decide what clutter is and act on that decision immediately. Both are risky.

Safer:

```text
Identify files that appear superseded. Do not delete or move anything.
Return a proposed archive plan with source path, destination path, and reason.
Wait for approval.
```

This asks the agent for the judgment call only. The execution authority is withheld. The agent returns a plan; you review it; you decide whether the judgment is right.

After approval:

```text
Move superseded files to archive/2026-06-14-cleanup/.
Report every move.
```

Now execution is authorized, scoped to specific files, targeted to a dated archive folder, and logged. If something goes wrong, you know exactly what was moved and where.

The dated archive subfolder is worth making a habit. `archive/` as a flat dump becomes a problem quickly — after a few cleanup passes, it is full of files without context for why they are there. `archive/2026-06-14-cleanup/` tells you when the operation happened. Combined with a brief note in `status.md`, it tells you why.

## The Safety Layer Stack

No single mechanism covers everything. The robust approach combines several layers, each with a different scope and strength.

The instruction file layer — the rule in `AGENTS.md` — is portable and applies to every tool that reads it. It is advisory, which means it can be overridden by an ambiguous prompt, but it is the only layer that works across tools without additional setup.

The hook or permission layer is deterministic: it blocks specific operations regardless of what the agent decided. It is tool-specific, so it requires per-tool implementation. In Claude Code, `PreToolUse` hooks are the mechanism. In other environments, shell wrappers or permission profiles serve the same function.

Git is the recovery layer for committed work. Its scope is limited to what has been tracked and committed, but within that scope it is extremely reliable. The practice of committing frequently — especially before any cleanup or structural operation — expands the scope of what git protects.

The archive folder is the recovery layer for work that git does not cover. A file in `archive/` is not gone; it is relocated. The archive folder is not a substitute for git, but it covers the gap.

Human approval is the judgment layer: the gate you place before any operation that is irreversible or high-blast-radius. The limit is that overusing it slows everything down. The right scope for human approval is operations that destroy or fundamentally reorganize things — not every file edit.

| Layer | What it does | Scope | Limit | When to rely on it |
|---|---|---|---|---|
| Instruction file | States preferred behavior. | Tools that read it. | Advisory only. | Baseline for every project. |
| Hook / permission | Blocks specific actions. | Tool-specific. | Requires setup. | Deterministic enforcement. |
| Git commit | Enables recovery. | Tracked committed files. | Misses untracked work. | Frequent checkpoints. |
| Archive folder | Makes cleanup reversible. | Any moved file. | Can grow noisy. | Default deletion alternative. |
| Human approval | Adds judgment. | High-blast-radius actions. | Slows flow if overused. | Irreversible structural changes. |

## Large Files and a Different Safety Problem

There is a distinct file safety issue that comes up in AI workflows and is worth separating from the deletion problem: large files that should not be in the repository.

The confusion usually sounds like this: "I added the dataset to `.gitignore` so the agent won't see it." That is two separate things conflated into one. `.gitignore` stops git from tracking the file going forward and prevents it from being committed to future history. It does nothing to prevent a local agent from reading the file, because the agent reads the local filesystem, not the git index.

If a file should not be read by the agent, it needs to be outside the workspace, not merely gitignored. `.gitignore` is a version control boundary. Agent access is a filesystem boundary. They are different.

The pattern I use for large data files: commit a small sample to the repo, document the command that fetches the full dataset, and keep the full data outside the workspace or in a tool-specific ignore file. The agent works with the sample. The sample is representative enough for development. The fetch command is documented so anyone who needs the full data can get it.

This matters for safety because it also avoids a different kind of agent error: reading a large file that it did not need to read, burning context on data that is not relevant to the current task. The sample approach solves both the version control problem and the context cost problem simultaneously.

## What Would Change My Mind

I would revise this chapter if all major tools provided reliable, cross-platform, inspectable rollback for every file operation — including untracked files, including gitignored files, including files outside version control. A filesystem-level undo that worked across shells and tools would make the archive pattern less necessary. Today, no such mechanism exists at useful breadth. Archive and git remain the durable portable layers, and the instruction-based approach remains the baseline for tools that do not support deterministic hooks.

## Still Puzzling

Whether archive folders should live inside or outside the repo is a question I have not resolved to my own satisfaction. Inside the repo: the archive is portable, moves with the project, and is visible to collaborators. Outside: the repo stays clean, the archive does not inflate repository size, and there is a clearer boundary between current work and retired work. My current practice is inside the repo and gitignored, which is a compromise that satisfies neither consideration completely.

How agents should handle large generated files that are genuinely safe to delete — intermediate build artifacts, cached results, temporary processing files — is an open design question. The no-delete rule should have a safe exception for regenerable targets, but defining "regenerable" precisely enough for a hook to act on it requires knowing the project's build semantics. A generic rule is too conservative; a project-specific rule requires maintenance.

Whether a generic destructive-action classifier could work across shells is the most speculative of the three puzzles. The idea would be a tool that recognizes patterns of commands with high blast radius — not just `rm` but `mv` with wildcards, `truncate`, `overwrite` operations in various forms — and escalates them consistently regardless of which shell or tool is running them. I have not seen this done well. The classifier would need to understand context, not just syntax, which makes it hard.

---

## Exercises

**Warm-up**

1. *(Recall)* Name the five safety layers from the chapter and give a one-sentence description of what each does. For each one, name its primary limit. *Tests whether you have the stack as a unified structure with gaps, not as a list of interchangeable options.*

2. *(Identification)* A colleague says: "I don't need an archive rule because I always commit before I let the agent do anything structural." Identify the specific case this does not protect against, and explain why. *Tests comprehension of the gap between git's scope and a project's full working state.*

3. *(Recall)* What is the difference between what `.gitignore` does and what keeping a file outside the workspace does, with respect to agent access? *Tests the version control vs. filesystem boundary distinction.*

**Application**

4. *(Writing)* Write the two-sentence no-delete rule from this chapter in your own words for a project that is a research writing project — not a software repo. Then write the corresponding `AGENTS.md` entry, including which operations require human approval and what the archive destination should be. *Tests ability to apply the canonical rule to a non-software context with appropriate adaptation.*

5. *(Rewriting)* The following prompt is unsafe: "Remove any files that look like old drafts." Rewrite it as a safe two-phase prompt: first a plan phase (no execution), then an execution phase (approved, scoped, logged). *Tests the core technique for separating judgment from execution authority in destructive work.*

6. *(Design)* You want to implement a no-delete hook for a Claude Code project. The project contains: `src/` (source code), `current/` (active drafts), `build/` (generated), `outputs/` (finished deliverables), `archive/` (retired files), `__pycache__/` (Python cache). Write the logic for the hook: which paths should be allowed to delete, which should redirect to archive, and which should escalate to human approval. *Tests ability to apply the regenerable-vs.-irreplaceable distinction to a real project structure.*

**Synthesis**

7. *(Integration)* You are setting up file safety for a project that three people will work on, using Claude Code, Cursor, and a bash-based automation script. Design the complete safety stack for this project. Identify which layers apply across all three tools, which are tool-specific, and what each person needs to do to set up their environment. *Tests integration of the full safety layer stack with the cross-tool portability concepts from Chapter 8.*

8. *(Analysis)* The chapter recommends dated archive subfolders (`archive/2026-06-14-cleanup/`) rather than a flat `archive/` folder. Describe two failure modes that the dated subfolder prevents, and one failure mode it does not prevent. What additional practice would address the one it misses? *Tests whether you can reason about the mechanics of the archive pattern beyond simple application.*

**Challenge**

9. *(Open-ended)* The chapter argues that the no-delete rule should have an exception for regenerable targets — files the project can reconstruct without human effort. Design a principled definition of "regenerable" that is specific enough for a hook to act on, general enough to work across different project types (software repos, writing projects, data pipelines), and honest about where the definition breaks down. What cases live at the boundary, and how should the hook behave when it cannot classify a file? *Engages the chapter's most practically difficult open problem and tests whether you can reason carefully about policy design under uncertainty.*

## Prompts

### Figure 11.1 - Git protection vs. project contents

Create a standalone D3 v7 HTML figure for "Git protection vs. project contents" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: Venn diagram with two circles. Left circle: "Files in the project folder." Right circle: "Files recoverable from git." The overlap is labeled "tracked and committed." Outside the right circle but inside the left: "untracked files, uncommitted changes, .gitignored files, generated artifacts." Caption: Git protects the o Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 11.2 - No-delete hook decision flow

Create a standalone D3 v7 HTML figure for "No-delete hook decision flow" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: Flowchart showing a file operation arriving at a PreToolUse hook. Decision diamond: "Does path match regenerable pattern?" If yes: "Allow." If no: second decision diamond: "Is this a move to archive/?" If yes: "Allow." If no: "Block and escalate to human approval." Caption: A no-delete hook intercepts destructive opera Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
