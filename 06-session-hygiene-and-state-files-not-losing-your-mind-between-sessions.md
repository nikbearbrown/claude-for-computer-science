# Chapter 6 — Session Hygiene and State Files: Not Losing Your Mind Between Sessions
*The agent didn't get worse. The conversation got crowded.*

The first hour was sharp. The agent found the right files, made the right edits, held your constraints without being reminded. You were moving.

Four hours later it revived an approach you rejected in hour two. It cited a command you superseded. It started contradicting decisions that felt settled. You spent twenty minutes re-explaining context the session had already worked out once.

Nothing failed. No error was thrown. The model's underlying capability did not degrade. What degraded was the conversation itself — the accumulated weight of tool outputs, intermediate results, discarded drafts, and rolled-back decisions that the agent was still carrying as if they were live context. The session became crowded, and crowded sessions make bad decisions.

This is the problem session hygiene exists to solve. Not by making the agent smarter, but by making forgetting safe.

## What Actually Happens in a Long Session

A language model does not have memory the way you do. It has a context window — a fixed region of text that it attends to when generating each response. Everything in that window has equal standing. A decision you made in hour one and a half-formed idea you typed and deleted two minutes later both occupy tokens. Both influence what comes next.

Chroma's context-rot research made this concrete: long contexts with distractors — material that is present but no longer relevant — measurably degrade reliability even before the window fills. The model is not ignoring the noise. It is weighting it, constantly, against the signal. Anthropic's context-engineering guidance draws the same conclusion from the other direction: what you keep out of context matters as much as what you put in.

![Session reliability with and without state files](images/06-session-hygiene-and-state-files-not-losing-your-mind-between-sessions-fig-01.png)
*Figure 6.1 - Session reliability with and without state files*

<!-- → [CHART: Line chart showing reliability vs. session length for two conditions — clean session with state files vs. unmanaged session; x-axis: session hours 1–6, y-axis: task accuracy or decision consistency; the curve for the unmanaged session bends down; caption: "Context-rot is not a cliff — it's a slope. The managed session stays flatter longer."] -->

The implication is uncomfortable: your instinct to keep the session running — to preserve continuity, to avoid re-loading context — is exactly backward. A long session is not more coherent than a fresh one. It is less coherent, because it carries everything that happened, not just what matters. The session that ends well and hands off cleanly is better than the session that runs until it breaks.

## The Three Files

Session hygiene is a discipline, not a tool. It is a habit of moving durable state out of the conversation and into files before the conversation gets heavy. Three files do most of the work:

```text
_MANIFEST.md         tells the agent what to read
status.md            tells the agent the current state
session-handoff.md   tells the next session where to resume
```

Each file has a different job. The manifest is architectural — it describes the project's structure and does not change session to session unless the project itself changes. The status file is a snapshot — it records where the project stands right now, what is decided, what is open, what comes next. The handoff is a letter — it is written at the end of one session for the benefit of the next one, and its job is to make starting fresh as cheap as possible.

![State files across the session lifecycle](images/06-session-hygiene-and-state-files-not-losing-your-mind-between-sessions-fig-02.png)
*Figure 6.2 - State files across the session lifecycle*

<!-- → [DIAGRAM: Three-file system showing how information flows — _MANIFEST.md feeds into session start, status.md updates at session end, session-handoff.md bridges between sessions; annotate with when each file is read and written; caption: "Each file serves a different moment in the session lifecycle."] -->

Together they create a loop. A session starts by reading the manifest and the handoff from the previous session. It works. It ends by updating the status file and writing a new handoff. The next session starts by reading those files. Nothing important lives only in the chat.

## What a Good Handoff Looks Like

The handoff is the file most people skip and most sessions most need. Here is one written at the end of a research session on a CLI tooling book:

```markdown
# Session Handoff - CLI Tooling Book - 2026-06-14

## Goal
Draft chapter research notes from TIKTOC.md.

## Done
- Notes written for chapters 1–15 in pantry/.
- Eight local library files copied with _lib_ prefix.

## Next actions
1. Draft chapters from notes.
2. Keep citations inline.
3. Log word counts.

## Do not reload
- Raw web search results.
- Entire MD library.
```

Four sections. Goal, done, next, do-not-reload. The last section is the one that earns its keep: it explicitly names what the next session should leave out of context. Raw web search results are large, stale, and already processed into notes. The entire MD library is enormous and no longer needed in bulk. Naming them prevents the next session from helpfully loading them again.

The handoff is not a log. It is not a summary of everything that happened. It is a targeted document that answers one question: *what does the next session need to know to start clean and move immediately?*

The failure mode is omitting the reason behind a decision. "Rejected the two-pass approach" is less useful than "Rejected the two-pass approach because it breaks on files over 100MB — single-pass only." The next session that reads the first version may reverse the decision. The next session that reads the second version understands the constraint.

## The Session Workflow

The workflow is short enough to run from memory once it becomes habit:

```text
Start:
1. Read _MANIFEST.md.
2. Read Tier 1 files.
3. Identify task domain.
4. Load only relevant Tier 2 files.
5. State a short plan.

During:
1. Keep tool output small.
2. Avoid unrelated pivots.
3. Record major decisions.

End:
1. Update status.md.
2. Write session-handoff.md.
3. Move superseded files to archive/.
4. Update _MANIFEST.md if canonical files changed.
```

The discipline lives in steps four and five of the start phase, and in all four end steps. Most sessions skip both. They load every file that might be relevant instead of only the files the task actually calls for. They close without writing anything down.

The cost of skipping shows up in the next session, not the current one. That delay makes the habit hard to build: the punishment is deferred, the effort is immediate.

![Start-work-end session lifecycle](images/06-session-hygiene-and-state-files-not-losing-your-mind-between-sessions-fig-03.png)
*Figure 6.3 - Start-work-end session lifecycle*

<!-- → [INFOGRAPHIC: Session lifecycle as a horizontal flow — Start → Work → End — with specific actions listed at each phase; highlight the "Do not load" principle at Start and the four-step end routine; caption: "The end-of-session routine is what makes the next session's start fast."] -->

## The `status.md` Template

The status file is simpler than the handoff. It describes where the project stands, not what happened today. A minimal template:

```markdown
---
project: project-alpha
status: active
updated: 2026-06-14
next:
  - draft chapter 4
  - verify Cursor rule docs
blocked_by: null
---

# Status
Current state, latest decisions, open questions, and next actions.
```

The YAML frontmatter makes the file machine-scannable. A tool or an agent that reads the first ten lines knows the project name, whether it is active, when it was last touched, what comes next, and whether it is blocked — without reading the prose body at all. The prose body is for the human.

Update this file at the end of every session. It should reflect the project as it stands now, not as it stood last week. A status file that is three sessions out of date is worse than no status file: it confidently describes a state that no longer exists.

## Three Misconceptions Worth Naming

**"The chat is the memory."** The chat is expensive memory. It costs tokens to reload, it does not persist across sessions, and it cannot be audited or versioned. A state file costs almost nothing to read and can be diffed, archived, and inspected at any time.

**"Fresh sessions waste time."** Fresh sessions save time when the handoff is good. The cost of a fresh start is a few minutes reading the handoff. The cost of a broken long session is an hour of re-explanation and error correction.

**"I should summarize everything."** No. Summarize decisions, not events. The next session does not need to know that you tried three approaches and the third one worked. It needs to know which approach you are using and why the other two are off the table.

## What Would Change My Mind

I would revise this chapter if agent tools developed transparent, accurate, portable long-term memory that preserved only useful decisions and reliably discarded noise. Current memory features are too tool-specific and too opaque to replace explicit state files. When a system can show me exactly what it retained from a previous session and why, and when that retention is accurate across tools, I will reconsider.

## Still Puzzling

- What is the ideal length for a handoff?
- Should session notes be archived by date automatically?
- Can agents reliably detect when a session should end?

---

## LLM Exercises

> These exercises are designed to be completed with a live language model session open. The point is not to answer correctly in isolation — it is to observe how session structure affects agent behavior in practice.

**Exercise 1: Write and use a handoff.** At the end of your next working session with an agent, write a `session-handoff.md` following the four-section structure from this chapter. Open a fresh session the following day, load only the handoff and the manifest, and note what the agent gets right immediately and what requires re-explanation. What was missing from your handoff?

**Exercise 2: Test context-rot directly.** Run a session for at least two hours on a real task. At the one-hour mark, ask the agent to state the current approach and any constraints you have established. Record its answer. At the two-hour mark, ask the same question again without prompting. Compare the two answers. What drifted?

**Exercise 3: Write a status file.** Write `status.md` for one active project using the template from this chapter. Then open a fresh session, load only `_MANIFEST.md` and `status.md`, and ask the agent to describe what the project needs next. How close is its answer to what you actually need to do?

---

## Exercises

**Warm-up**

1. *(Basic recall)* What are the three state files introduced in this chapter, and what is the specific job of each? State each job in one sentence. *What this tests: ability to distinguish the three files without conflating them.*

2. *(Vocabulary)* What does "context-rot" mean, and why does it happen before the context window is full? *What this tests: understanding the mechanism, not just the symptom.*

3. *(Identification)* Look at the sample handoff in this chapter. Which section is described as the one that "earns its keep"? Why? *What this tests: close reading and understanding of the handoff's design logic.*

**Application**

4. *(Writing)* Write a `session-handoff.md` for a real project you are working on. Use all four sections. For the "Do not reload" section, name at least two specific things the next session should leave out of context and explain why each one should stay out. *What this tests: applying the handoff structure to a real scenario.*

5. *(Template use)* Write a `status.md` for the same project using the template from this chapter. Fill in the YAML frontmatter completely. In the prose body, write three sentences describing current state, one open question, and two next actions. *What this tests: populating the template with real project state.*

6. *(Failure mode)* A colleague's handoff reads: "We tried a few approaches and settled on the good one. Next: keep going." What is missing? Rewrite the handoff to include the information actually needed for the next session to start clean. *What this tests: recognizing and correcting an underspecified handoff.*

**Synthesis**

7. *(Integration)* Chapter 3 introduced `_MANIFEST.md`. Chapter 6 introduces `status.md` and `session-handoff.md`. How do these three files divide responsibility? Is there any information that could reasonably live in more than one of them? If so, where should it live, and why? *What this tests: understanding the system as a whole, not just its parts.*

8. *(Workflow design)* The session workflow in this chapter has a "During" phase that says "record major decisions." Design a lightweight convention for doing this inside a live session — something an agent could follow without stopping work. What should be recorded, in what format, and where? *What this tests: extending the framework to the in-session phase, which the chapter leaves underspecified.*

**Challenge**

9. *(Open-ended)* The chapter argues that fresh sessions are better than long ones when the handoff is good. But "good handoff" is doing a lot of work in that sentence. What would it take to make handoff quality measurable? Propose a rubric — at least three criteria, each with a way to evaluate it — that a practitioner could use to score their own handoffs over time. *What this tests: moving from intuition to operational definition, and honest accounting of what is hard to measure.*

## Prompts

### Figure 6.1 - Session reliability with and without state files

Create a standalone D3 v7 HTML figure for "Session reliability with and without state files" using the pinned CDN. Use a timeline / progression structure with 4-6 labeled elements inferred from this concept: Line chart showing reliability vs. session length for two conditions — clean session with state files vs. unmanaged session; x-axis: session hours 1–6, y-axis: task accuracy or decision consistency; the curve for the unmanaged session bends down; caption: "Context-rot is not a cliff — it's a slope. The managed session  Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 6.2 - State files across the session lifecycle

Create a standalone D3 v7 HTML figure for "State files across the session lifecycle" using the pinned CDN. Use a process flowchart structure with 4-6 labeled elements inferred from this concept: Three-file system showing how information flows — _MANIFEST.md feeds into session start, status.md updates at session end, session-handoff.md bridges between sessions; annotate with when each file is read and written; caption: "Each file serves a different moment in the session lifecycle." Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 6.3 - Start-work-end session lifecycle

Create a standalone D3 v7 HTML figure for "Start-work-end session lifecycle" using the pinned CDN. Use a process flowchart structure with 4-6 labeled elements inferred from this concept: Session lifecycle as a horizontal flow — Start → Work → End — with specific actions listed at each phase; highlight the "Do not load" principle at Start and the four-step end routine; caption: "The end-of-session routine is what makes the next session's start fast." Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
