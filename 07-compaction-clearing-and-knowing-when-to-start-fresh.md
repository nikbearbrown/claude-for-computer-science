# Chapter 7 — Compaction, Clearing, and Knowing When to Start Fresh

You are halfway through a migration. The session is getting sluggish, so you type the reset command. The tool clears the context. Now the agent no longer knows the accepted plan, the files already touched, or the test failure you were chasing.

The problem was real. The intervention was wrong.

---

There are three things people do when a session degrades, and they are not interchangeable. Compaction summarizes and continues. Clearing resets. Starting a fresh session begins from a written handoff. Each one is the right move in a different situation, and using the wrong one produces a specific kind of damage — not a visible crash, but a quiet loss of state that only shows up later, when the agent makes a decision that contradicts something it no longer knows.

Understanding why they are different requires thinking about what a session actually is. A session is accumulated context: the history of what was asked, what was tried, what was decided, what failed, and what is currently believed to be true. That history is useful precisely because it is not just a record — it is the basis on which the agent interprets the next instruction. When you ask "can we proceed with the plan," the agent answers relative to a plan it learned about somewhere in that history. If the history is gone, the question lands without a referent.

The degradation problem is that history has costs. Every token in context competes for the model's attention. A long session full of dead-end debugging paths, superseded decisions, and redundant file reads is noisy — the relevant state is present, but it is buried. The session slows not because the window is literally full, though that can happen, but because the signal-to-noise ratio has dropped. The agent starts answering relative to old assumptions because the old assumptions occupy more of the context than the current ones.

The three interventions address this problem at different levels of aggression, and choosing among them depends entirely on what state you need to survive.

---

Compaction is the least aggressive intervention. It says: keep this session going, but summarize the history into something denser and more focused. The agent stops, generates a compressed representation of the session state, and continues from that representation rather than from the raw history. Dead-end paths disappear. Repeated context collapses. The decisions that mattered are preserved.

The mistake practitioners make with compaction is treating it as automatic. Most tools offer a compaction command — `/compact` in Claude Code, variants in other tools — and the temptation is to issue it the way you restart a slow application: just run it and hope the problem resolves. Sometimes it does. Often the result is a compacted context that preserved the wrong things — the verbose debugging trail, the obsolete plan from three hours ago — while dropping the things that actually mattered, because the compaction algorithm had no way to know which was which.

The fix is to give compaction a focus. Not just "compact this," but "compact this, keeping specifically these things and dropping everything else." The difference between a bad compaction prompt and a good one is the difference between asking someone to summarize a meeting and asking them to summarize the meeting with particular attention to the three decisions made and the two open questions that weren't resolved.

A bad compaction prompt:

```text
/compact
```

A better one:

```text
/compact Keep only: accepted plan, files changed, unresolved test failure,
current assumptions, and next two actions. Drop dead-end debugging paths.
```

The second prompt gives the compaction process a schema. It names the categories of state that must survive and explicitly licenses dropping everything else. The result is a compact summary that is actually useful rather than a dense version of the same noise.

The rule for when to use compaction is: same task continues, history contains decisions worth preserving, and you can name what to keep. If you cannot name what to keep, that is a signal — either the session has genuinely lost its shape and compaction will not help, or you have not thought carefully enough about the current state to be directing the agent at all.

---

Clearing is the more aggressive intervention. It removes history. The agent starts the next exchange without any of the prior session in context. This is the right move when the prior session is done, when the context has become actively misleading, or when you have already written a handoff and want a clean read of the project state from the handoff alone.

The failure mode that makes clearing dangerous is using it when compaction was the right move — which is exactly what happened in the opening scenario. The session was degraded, the tool offered a reset command, and the reset removed the plan, the files-touched record, and the unresolved test failure. The session was slow because it needed compaction. It got clearing instead.

The diagnostic question before clearing is: have I written down everything the next session needs to know? If the answer is no, and the task is not finished, clearing destroys state. The handoff document is not optional — it is the mechanism by which continuity survives a context reset. Without it, clearing is just forgetting.

A minimal handoff before clearing:

```text
session-handoff.md:
- Accepted plan: [summary]
- Files changed: [list]
- Unresolved: [failing test, with reproduction steps]
- Current assumptions: [what we believe to be true right now]
- Next two actions: [what happens when the session resumes]
```

That document becomes the seed of the next session. When you open a fresh session, you load `session-handoff.md`, the manifest, and the project rules, and the agent begins with the essential context without any of the noise. The history is gone, but the state is not.

---

A fresh session is not the same as clearing, though they both start with a clean context window. Clearing is an action taken on a continuing task — you reset and resume. A fresh session is a deliberate choice to approach something without the builder's accumulated assumptions. It is what you use when the task domain changes substantially, when you want an independent reviewer who has not been anchored by the same decisions you have been anchored by, or when a session has accumulated so many dead ends that the history is more misleading than helpful and even a focused compaction would not rescue it.

The reviewer use case is worth dwelling on because it is counterintuitive. When you have been working on something for a long session, you have a strong mental model of why decisions were made, what was tried, and what the current situation means. An agent that has been in that session with you has a similar accumulated model. If you ask that agent to review the current state for problems, it will review through the lens of everything it watched you do — which means it will tend to rationalize rather than question.

A reviewer in a fresh session, given only the diff, the acceptance criteria, and the project rules, has no such anchor. It sees the current state without the narrative that produced it. That is often exactly what you want. The reviewer does not need the builder's whole trail. It needs enough context to evaluate the output independently.

```text
Fresh session reviewer prompt:
Read session-handoff.md, _MANIFEST.md, and PROJECT_RULES.md.
Do not read raw logs unless the failing test still reproduces.
```

That prompt is deliberately sparse. It gives the reviewer the state without the history, the acceptance criteria without the debugging path, the rules without the decisions that bent them along the way.

| Situation | Right intervention | What you must do first |
|---|---|---|
| Long but coherent session | Compact | Preserve current decisions and open tasks. |
| Stale or conflicting context | Clear | Write a handoff file with current truth. |
| New task or changed goal | Fresh session | Name canonical files and scope. |
| Tool output has flooded context | Summarize or trim | Extract only the relevant failures or results. |

---

The decision logic is not complicated once you have named what each intervention does.

Use compaction when the same task is ongoing, the history contains decisions worth preserving, and you can name what to keep. Compaction is not magic — it is only as good as the focus you provide. An unfocused compaction is just a slower version of the original noise.

Use clearing when the task is done, when the context has become actively misleading, or when the handoff has already been written and you want a clean read from it. Do not clear without writing the handoff first if the task is not finished. The handoff is not a nice-to-have; it is what makes clearing safe.

Use a fresh session when the task domain changes, when you need an unbiased reviewer, or when the session has accumulated enough dead ends that even focused compaction would not restore it. A fresh session is not a failure — it is the right tool for tasks that benefit from clean eyes.

---

A few things complicate this in practice. The first is that tool-specific compaction behavior changes. The commands, the default preservation heuristics, and the degree of opacity in what was actually kept all vary across tools and across versions of the same tool. What `/compact` preserves in one version of Claude Code may not be what it preserves in the next. This is not a reason to avoid compaction — it is a reason to write focused compaction prompts rather than relying on the tool's defaults, and to treat any specific claim about compaction behavior as "last verified" rather than permanently true.

The second complication is auto-compaction. Several tools will automatically compact a session when it approaches the context limit, without being explicitly asked. This is a reasonable fallback, and it prevents the worst case — a session that simply stops working because the window is full. But auto-compaction is not a workflow. It runs when the tool decides it needs to, not when you decide the session needs it, and it applies whatever preservation heuristics the tool has rather than the ones you would choose. Treating auto-compaction as a substitute for session hygiene is the same error as treating a hard drive backup as a substitute for deliberate version control — it catches catastrophic failure but not the quiet drift.

The third is a question that remains genuinely open: can reviewers be too context-starved? The fresh-session reviewer prompt is deliberately sparse, but there is a minimum below which a reviewer cannot do its job — if it does not know what the task was, it cannot evaluate whether the output satisfies it. The handoff document has to be complete enough to support independent evaluation. Getting that threshold right is harder than it looks, and I have seen both over-specified handoffs that just recreate the builder's context in a new session, and under-specified ones that leave the reviewer without enough to work with. There is no formula here yet, only the heuristic that the handoff should contain state, not history.

---

What would change this picture is tools that expose inspectable, reliable compaction summaries with clear guarantees about what was preserved and what was dropped. Current compaction is partly opaque — you issue the command, the session continues, and you infer from subsequent behavior whether the right things were kept. A tool that showed you a structured diff of context before and after compaction would let you verify the preservation rather than guess at it. No major tool currently does this well. Until they do, the focused compaction prompt is the best instrument you have for making compaction do what you actually need.

---

## LLM Exercises

1. Take a current or recent session and write the compaction focus prompt you would have used at the midpoint — the specific list of what to keep and what to drop. Ask the model to evaluate whether that list would have preserved enough to continue the task, and what it might have missed.

2. Write a `session-handoff.md` for a task you are currently working on, as if you were about to clear context. Then ask the model to read only that file and tell you what it knows about the current project state. Identify any gaps — things you assumed were in the handoff that were not.

3. Design a reviewer prompt for a task where you have strong opinions about the current approach. Deliberately exclude your reasoning and rationale. Ask the model, reading only the handoff and acceptance criteria, to identify the three most questionable decisions in the current state. Note where its critique aligns with doubts you already had and where it surfaces something you had stopped seeing.

![The right reset action depends on whether the current session contains useful state, stale context, or task drift.](images/07-compaction-clearing-and-knowing-when-to-start-fresh-fig-01.png)
*Figure 7.1 - Session Reset Decision Tree*

![A session becomes less reliable when stale assumptions and irrelevant material accumulate.](images/07-compaction-clearing-and-knowing-when-to-start-fresh-fig-02.png)
*Figure 7.2 - Context Decay Over a Session*

## Prompts

### Figure 7.1 - Session Reset Decision Tree

Create a standalone D3 v7 HTML figure for "Session Reset Decision Tree" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: The right reset action depends on whether the current session contains useful state, stale context, or task drift. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 7.2 - Context Decay Over a Session

Create a standalone D3 v7 HTML figure for "Context Decay Over a Session" using the pinned CDN. Use a timeline / progression structure with 4-6 labeled elements inferred from this concept: A session becomes less reliable when stale assumptions and irrelevant material accumulate. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
