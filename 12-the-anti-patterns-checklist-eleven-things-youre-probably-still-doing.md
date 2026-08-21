# Chapter 12 — The Anti-Patterns Checklist: Eleven Things You're Probably Still Doing
*Boundary failures do not announce themselves. They accumulate.*

You added a manifest. You shortened the rules file. You moved old drafts to archive. You wrote a status file and started keeping handoffs. You have done the work, chapter by chapter, and the sessions are better than they were.

But something still feels off. Not broken — just not clean. The agent occasionally drifts. The project root feels heavier than it should. A session that should take twenty minutes takes forty.

This chapter is the wall checklist. The thing you run when you have fixed the obvious problems and something is still wrong. Not because anything is mysterious — most agent workflow failures are not mysterious — but because the failures tend to cluster around the same structural problems, and when you have solved a few you can miss the ones that remain.

## Why the Same Failures Keep Appearing

The pattern, once you see it, is consistent across projects, tools, and domains. Almost every degraded agent session traces back to a boundary that was not maintained: the project boundary that lets unrelated work bleed in, the source/output boundary that lets generated files be treated as canonical, the current/archive boundary that lets old drafts compete with live ones, the session boundary that lets unrelated tasks accumulate in one conversation, the tool boundary that lets one provider's behavior become a dependency.

None of these failures are dramatic. They do not produce errors. They produce noise — a gradual thickening of the context that makes the agent slightly less reliable, slightly slower to converge, slightly more prone to reviving dead approaches. By the time the problem is visible, it has been accumulating for a while.

The checklist exists to catch them before they accumulate.

![Anti-patterns by boundary type](images/12-the-anti-patterns-checklist-eleven-things-youre-probably-still-doing-fig-01.png)
*Figure 12.1 - Anti-patterns by boundary type*

<!-- → [DIAGRAM: Radial diagram with "Agent reliability" at center and eleven anti-patterns arrayed around it as spokes, grouped by boundary type (project, source/output, session, tool); caption: "The eleven anti-patterns cluster into five boundary types. Fix the boundary, not just the symptom."] -->

## The Checklist

Run this as a fifteen-minute audit. Check each item against your most active project. Mark the ones that apply. Fix the highest-risk three first.

```text
[ ] 1. Pointing the agent at a giant parent folder
    Fix: one bounded project root.

[ ] 2. Mixing unrelated projects in one session
    Fix: separate sessions and handoffs.

[ ] 3. Keeping old drafts beside current drafts
    Fix: archive/ for superseded work.

[ ] 4. Letting outputs become source
    Fix: mark build/ and outputs/ as non-canonical.

[ ] 5. Huge global memory or instruction files
    Fix: short rules; move detail to task-specific docs.

[ ] 6. Many unrelated tasks in one session
    Fix: one task per thread.

[ ] 7. Dumping full logs or JSON into context
    Fix: summarize, filter, aggregate.

[ ] 8. Asking "what is in this folder?" every session
    Fix: maintain status.md.

[ ] 9. Vague tasks such as "clean this up"
    Fix: target, constraint, done criteria.

[ ] 10. Letting the agent delete or rename without review
    Fix: archive-not-delete and destructive gates.

[ ] 11. Depending on tool-specific behavior
    Fix: encode intent in portable files plus adapters.
```

Each item is a short diagnosis and a structural repair. The diagnosis names what is wrong; the repair names the boundary that needs to be re-established. What follows is the logic behind each one — why it fails, what it costs, and what the fix actually does.

## Walking the List

**1. Pointing the agent at a giant parent folder.** When the project root is a folder containing five other projects, the agent has no way to know which material is relevant. It reads across project boundaries because they are not marked. The fix is a bounded root — a folder that contains exactly the active project and nothing else.

**2. Mixing unrelated projects in one session.** A session that handles two different projects is carrying two contexts simultaneously. The agent may confuse terminology, apply constraints from one project to another, or produce work that blends both. The fix is session isolation: one project per session, with a clean handoff when switching.

**3. Keeping old drafts beside current drafts.** This is the most common failure in document-heavy projects. A folder that contains both `chapter-04-v1.md` and `chapter-04.md` does not tell the agent which one is current. It reads both. The old draft competes with the live one. The fix is a strict archive discipline: superseded files move to `archive/` immediately, not eventually.

**4. Letting outputs become source.** A generated EPUB, a compiled PDF, a rendered HTML file — these are outputs, not source. When they live beside the source files, the agent may treat them as canonical. It will update the wrong file, derive facts from a stale build, or confuse what it is supposed to edit with what it already produced. The fix is a hard boundary: `build/` and `outputs/` are non-canonical. The manifest says so explicitly.

| Anti-pattern | Boundary violated | Signal that it is happening | Structural repair |
|---|---|---|---|
| Flat project folder | Project boundary | Agent reads stale or ambiguous files. | Separate current, archive, outputs, and build. |
| Duplicate source files | Source boundary | Two files look equally canonical. | Declare one source of truth. |
| Unscoped prompt | Task boundary | Agent searches everywhere. | Name target files and exclusions. |
| Archive beside current | Time boundary | Old work is treated as live. | Move archive out of default path. |

**5. Huge global memory or instruction files.** A rules file that has accumulated months of additions becomes a dense instruction set that the agent must parse before every session. Contradictions creep in. Old rules that were relevant to a finished project still apply. The fix is ruthless pruning: short global rules for things that are always true, task-specific documents for things that are only sometimes true.

**6. Many unrelated tasks in one session.** Research, drafting, image prompts, and build debugging in one session is four different context loads competing for the same window. Each task pivot carries the previous task's output into the new task's context. The fix is the one-task-per-thread discipline: when the task changes significantly, open a new session and hand off.

**7. Dumping full logs or JSON into context.** A 2,000-line log file pasted into the conversation occupies the context window for the rest of the session. The agent processes it once and then carries it forever. The fix is pre-processing: summarize the log, filter the JSON, aggregate the records before they enter context. Chapter 9 covered this for MCP payloads; the same principle applies to anything you paste directly.

**8. Asking "what is in this folder?" every session.** If you or the agent has to reconstruct the project state from a directory listing at the start of every session, `status.md` is not being maintained. The cost is small per session and large over a project lifetime. The fix is a current, accurate status file that answers the question before it is asked.

**9. Vague tasks such as "clean this up."** "Clean this up" is three decisions deferred to the agent: what to target, what constraints apply, and when the task is done. The agent makes those decisions — not necessarily the same way you would. The fix is a task specification with a target (what file or section), a constraint (what must not change), and a done criterion (what the output should look like).

**10. Letting the agent delete or rename without review.** A deletion is irreversible in the session. A rename that touches ten files is hard to audit after the fact. The fix is a structural gate: the agent archives rather than deletes, proposes renames as a list before executing, and treats any action that cannot be undone as requiring explicit confirmation.

**11. Depending on tool-specific behavior.** A workflow that relies on one provider's memory format, one client's folder convention, or one tool's specific output structure is fragile. The fix is to encode intent in portable files — manifests, rules files, status files — and use thin adapters for tool-specific behavior. The logic lives in files; the adapter makes the files work with the tool.

| Anti-pattern | Boundary violated | Signal that it is happening | Structural repair |
|---|---|---|---|
| Session-only memory | Session boundary | Next session repeats old confusion. | Write status and handoff files. |
| Bloated rules | Instruction boundary | Agent ignores long instructions. | Trim to durable rules and references. |
| Adapter copies | Tool boundary | Different tools behave differently. | Generate thin adapters from one source. |
| Always-on connectors | Tool boundary | Context is crowded at session start. | Scope MCP servers per task. |
| Direct deletion | Safety boundary | Recovery depends on luck. | Archive first; delete only regenerable files. |
| No validator | Quality boundary | Plausible output slips through. | Add review/test passes. |
| No measurement | Evaluation boundary | Rules accumulate by vibes. | Benchmark and prune. |

## Running the Audit on a Real Project

A book repository: no manifest, old chapter drafts mixed with current ones in `chapters/`, a generated EPUB in the same folder, and a long chat session that has handled research, drafting, image prompts, and build debugging.

The checklist catches four items immediately:

```text
3: old drafts beside current drafts
4: outputs (EPUB) becoming source
6: unrelated tasks in one session
8: no status.md
```

The fix order matters. Start with the boundary causing the most immediate damage:

1. Move the EPUB to `output/`. The agent will stop treating it as canonical.
2. Move old chapter drafts to `archive/`. The current drafts are now the only drafts.
3. Write `status.md`. The next session starts with a clear picture of the project state.
4. Close the current session, write a handoff, start fresh.

The session work — research, drafting, image prompts, build debugging — cannot be un-mixed. But the next session starts clean, with a bounded project root, current files only, and a status file that answers the orientation questions before they are asked.

The lesson here is not that the checklist catches everything. It catches the structural failures. Judgment catches the rest.

## Three Misconceptions Worth Naming

**"I need to fix all eleven now."** No. Fix the highest-risk three. The checklist is a triage tool, not a to-do list. Running all eleven fixes at once in a live project is more likely to introduce new problems than to resolve the old ones. The question to ask for each item is not "should I fix this?" but "is this causing the current failure, and if so, how much does fixing it cost?"

**"Anti-patterns mean I did something wrong."** They mean the project evolved faster than its structure. Almost every item on this list is the result of a reasonable decision made at a smaller scale that became a problem at a larger one. Adding a second project to a folder made sense when both were small. It stopped making sense when one grew. That is not a mistake. It is a project that needs a structural update.

**"A checklist is beginner material."** Experts use checklists because repeated work fails in repeated ways. The checklist is not a substitute for expertise — it is a tool that expertise uses to not miss the obvious things while focused on the hard ones. Aviation, surgery, and software deployment all use them for the same reason.

## What Would Change My Mind

I would revise this checklist if empirical studies showed different failure clusters in real agent sessions. The eleven items here synthesize the strongest recurring failures in current practice and research, but they are not derived from a controlled study. If systematic analysis of degraded agent sessions revealed a different distribution — if, say, tool-boundary failures were rare and instruction-file failures were dominant — the list should reflect that. For now, it reflects what practitioners report most consistently.

## Still Puzzling

- Which anti-pattern predicts the most downstream failures?
- Should teams score anti-patterns in CI?
- How often should the checklist be rerun?

---

## LLM Exercises

> Run these with a live agent session and a real project. The goal is to observe how the agent responds to structural problems, not to audit in the abstract.

**Exercise 1: Run the full audit.** Apply the checklist to your most active project. For each of the eleven items, mark pass or fail and write one sentence describing the evidence. When you are done, you should have a list of failures with evidence for each. Do not fix anything yet.

**Exercise 2: Prioritize by impact.** Take your list of failures from Exercise 1. For each failure, estimate: how much context does this add per session? How often does it cause a wrong output? How long would the fix take? Rank the failures by the ratio of damage to fix cost. Fix the top three. Run the checklist again after one week.

**Exercise 3: Observe a vague task.** Give your agent a deliberately vague instruction — "review this chapter and improve it" or "clean up the folder." Record what it does: what it targets, what constraints it assumes, what done looks like to it. Then rewrite the same task using target, constraint, and done criteria. Give the rewritten task to the agent and compare the outputs. What changed?

---

![The practical response to an anti-pattern is to repair the boundary that allowed it, not just to correct one output.](images/12-the-anti-patterns-checklist-eleven-things-youre-probably-still-doing-fig-02.png)
*Figure 12.2 - Anti-Pattern to Fix Path*

## Exercises

**Warm-up**

1. *(Basic recall)* Name the five boundary types that the anti-patterns cluster into. For each boundary type, name one anti-pattern from the checklist that violates it. *What this tests: ability to organize the eleven items into the underlying structural framework.*

2. *(Vocabulary)* What does "archive-not-delete" mean as a practice? Why is it the recommended fix for anti-pattern 10 rather than a confirmation prompt? *What this tests: understanding the rationale behind the structural fix, not just its name.*

3. *(Identification)* A project has the following structure: `project/chapters/`, `project/chapters/old/`, `project/chapters/outputs/epub/`, `project/notes/`, `project/notes/archive/`. Which anti-patterns from the checklist are visible in this structure? Name each and explain the evidence. *What this tests: reading a real structure against the checklist.*

**Application**

4. *(Audit)* Run the checklist on a real or described project of your choice. Mark each item pass or fail with one sentence of evidence. Identify the three highest-risk failures and explain why you ranked them that way. *What this tests: applying the checklist as a triage tool.*

5. *(Task rewrite)* Rewrite the following vague task as a specification with target, constraint, and done criteria: "Go through the draft and tighten it up." You may choose what the draft is about. *What this tests: applying the fix for anti-pattern 9 to a concrete instruction.*

6. *(Fix sequencing)* You have identified four anti-patterns in a project: old drafts beside current ones, a vague global rules file, unrelated tasks in the current session, and no status file. In what order would you fix them, and why? What is the cost of fixing them in the wrong order? *What this tests: understanding that fix order matters and why.*

**Synthesis**

7. *(Integration)* Anti-patterns 3, 4, and 8 all have fixes that were introduced in earlier chapters (archive discipline in Chapter 3's manifest tier structure, non-canonical output marking in Chapter 3, status.md in Chapter 6). What does the recurrence of these fixes across chapters tell you about the structure of the framework? Why do the same tools keep appearing as the answer to different problems? *What this tests: connecting the checklist to the broader architecture of the book.*

8. *(Design)* Anti-pattern 11 says to "encode intent in portable files plus adapters." Describe what this would look like in practice for a project that currently depends on a single provider's memory format. What goes in the portable file? What does the adapter do? What is lost in the translation? *What this tests: moving from the abstract principle to a concrete implementation.*

**Challenge**

9. *(Open-ended)* The "Still Puzzling" section asks whether anti-patterns should be scored in CI. Design a minimal CI check — something that could run in under thirty seconds on a commit — that catches at least three of the eleven anti-patterns automatically. What does it check, what does a failure look like, and what does it not catch? What would a false positive cost, and how would you reduce false positive rate without reducing coverage? *What this tests: moving from a diagnostic checklist to an automated enforcement system, with honest accounting of what automation cannot replace.*

## Prompts

### Figure 12.1 - Anti-patterns by boundary type

Create a standalone D3 v7 HTML figure for "Anti-patterns by boundary type" using the pinned CDN. Use a systems diagram structure with 4-6 labeled elements inferred from this concept: Radial diagram with "Agent reliability" at center and eleven anti-patterns arrayed around it as spokes, grouped by boundary type (project, source/output, session, tool); caption: "The eleven anti-patterns cluster into five boundary types. Fix the boundary, not just the symptom." Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 12.2 - Anti-Pattern to Fix Path

Create a standalone D3 v7 HTML figure for "Anti-Pattern to Fix Path" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: The practical response to an anti-pattern is to repair the boundary that allowed it, not just to correct one output. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
