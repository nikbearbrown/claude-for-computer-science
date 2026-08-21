# Chapter 5 — The Five Context Cost Types: Where Your Tokens Are Going
*Why your agent gets worse before you've typed a word, and what you can actually do about it.*

There is a moment in any long agent session that feels like the lights dimming. The responses slow down slightly. The agent starts hedging where it used to be precise. It retrieves the wrong version of something, or stops following an instruction it had been following correctly for an hour. Nothing has changed in what you are asking. The degradation feels mysterious.

It is not mysterious. It is arithmetic.

A language model's context window is finite. When it fills, one of two things happens: either the model starts dropping earlier material to make room, or the sheer density of what is already there starts diluting the signal of what you just said. Either way, quality falls. The session that felt sharp at turn five feels different at turn sixty, not because the model changed, but because what is competing for its attention has accumulated.

What makes this tractable is that context cost is not one thing. There are five distinct types, each with a different origin, a different growth rate, and a different fix. The single most useful thing you can do with a struggling agent session is figure out which type is dominating — because the right intervention for one type is useless against another.

## The Budget Metaphor, Used Carefully

Anthropic's context engineering guidance treats the context window as a finite attention budget. That is a useful frame, but it can mislead if you take it too literally. A financial budget is fungible — a dollar saved on lunch is a dollar you can spend on dinner. Context is not quite that simple. It matters what is in the context and where, not just how much.

Research on what Chroma calls "context rot" makes this concrete: as long inputs accumulate, reliability does not degrade uniformly. It degrades because distractors accumulate. The model is not failing because it ran out of tokens; it is failing because it cannot reliably find the signal inside the noise. A window filled with relevant, well-organized material behaves better than a smaller window stuffed with stale logs and duplicate instructions.

So the budget metaphor is useful for sizing decisions — do I have room for this file? — but the quality problem is really a signal-to-noise problem. Both matter, and the five cost types affect them differently.

![Context window cost composition](images/05-the-five-context-cost-types-where-your-tokens-are-going-fig-01.png)
*Figure 5.1 - Context window cost composition*

<!-- → [DIAGRAM: A horizontal bar representing a context window, segmented into five color-coded regions labeled Fixed, Per-task, Accumulated, Tool-output, and Connector/MCP. A second bar below shows the same window after optimization — same total size, but proportions shifted, with fixed and accumulated costs reduced. Caption: Context is finite and the composition matters as much as the total. Each cost type grows and shrinks differently; the optimization strategies do not transfer between them.] -->

## Fixed Cost: What You Pay Before You Type

The first type is fixed cost: everything that loads before you ask your first question.

This includes instruction files — `AGENTS.md`, `CLAUDE.md`, system prompts, any rules the agent loads at startup. It includes tool schemas: the structured descriptions of every MCP server or function the agent has access to. If you have connected GitHub, Slack, Drive, and a database to your agent workspace, the descriptions of all four are probably visible at session start, whether or not any of them are relevant to what you are about to do.

Fixed cost is seductive to ignore because it is invisible. It does not appear in the conversation. You do not watch it accumulate turn by turn. It simply takes up space before the first word you type.

The magnitude can surprise you. A reasonably detailed `AGENTS.md` might be five hundred words. A set of four MCP server schemas might add several thousand tokens more. None of that is unreasonable in isolation. Together, before any conversation has happened, you have already spent a meaningful fraction of whatever budget you are working within — in quality terms, not just raw token counts.

The fix is the one that feels counterintuitive: make your instructions shorter and more precise, and disable the tool integrations you do not need for this particular task. A session about drafting chapter text does not need the database MCP server loaded. A session about reviewing code does not need the calendar integration. The cost is real whether you call the tool or not.

| Fixed cost source | Typical size | Primary reduction strategy |
|---|---:|---|
| System prompt / AGENTS.md | Hundreds to low thousands of tokens | Cut redundancy; use references instead of repeated examples. |
| Always-loaded tool schemas | Hundreds to thousands per server | Disable unused servers before session start. |
| Duplicate rules across files | Variable and often invisible | Audit overlap between AGENTS.md, CLAUDE.md, and adapters. |

*Fixed cost is invisible but not free. It is the baseline you pay before any task begins.*

## Per-Task Cost: What This Particular Job Loaded

The second type is per-task cost: the files, searches, and attachments loaded specifically for the current task.

This one has an obvious failure mode. The agent is asked to "look at the research folder," and it reads the research folder — all of it, including everything that has nothing to do with the current task. Or a search returns ten results when two were relevant, and the agent reads all ten. Per-task cost is well-intentioned loading that overshoots.

The fix here is precision before retrieval. Instead of reading a folder, read a file. Instead of returning all search results, filter before loading. The principle is to narrow first and load second — which means the instructions you write for your agent need to encode narrowing strategies, not just "go find what you need."

There is an ordering principle that applies here and generalizes across all five types: keep stable material early in the context and variable material late. If your instructions come before your task-specific files, the stable part can benefit from prompt caching — a mechanism where the model recognizes an identical prefix and avoids re-processing it. The moment the prefix changes, the cache breaks. So task-specific material, which changes with every task, belongs after stable rules, which do not. Even in tools that do not expose caching directly, this ordering makes the prompt easier to reason about because stable context accumulates at the beginning rather than getting interspersed with variable content.

## Accumulated Cost: The Weight of History

The third type is accumulated cost: the conversation history from all the turns that have already happened.

This is the one people notice most visibly, because they can watch it grow. Every exchange adds to it. By turn fifty, a significant portion of the context window contains the transcript of turns one through forty-nine — including the wrong answers, the abandoned approaches, the instructions the agent needed at the beginning but not now.

Accumulated cost has a particular failure mode distinct from fixed cost: it is not just volume but relevance decay. Turn three might have contained a detailed description of the task context that the agent needed at the time. By turn forty, the agent has been executing on that context for a while, and re-reading the original description on every turn is mostly noise. The signal-to-noise ratio drops even if the raw token count seems manageable.

The standard interventions are compaction and fresh sessions. Compaction means asking the agent to summarize what has happened so far, replacing the full conversation history with a dense summary that preserves what matters. A session handoff means capturing current state in `session-handoff.md` and starting a fresh context, so you are not dragging the weight of the old session into the new one.

The important thing to understand is that compaction and fresh sessions help accumulated cost but do nothing about fixed cost. I see this mistake often: a session has become slow and unreliable, so the user compacts it, and nothing improves — because the problem was never the conversation length, it was that four MCP servers were loaded for a task that needed one, or that `AGENTS.md` had grown to three thousand words by accumulation.

![Context cost growth over conversation turns](images/05-the-five-context-cost-types-where-your-tokens-are-going-fig-02.png)
*Figure 5.2 - Context cost growth over conversation turns*

<!-- → [DIAGRAM: Line graph with x-axis showing conversation turns (0–80) and y-axis showing percentage of context window consumed. Three lines: one for fixed cost (flat horizontal line from turn 0); one for accumulated cost (gradual upward slope); one for tool-output cost (irregular spikes). Caption: The three dynamic cost types grow at different rates and in different patterns. Fixed cost is set at session start; accumulated cost grows steadily; tool-output cost spikes unpredictably. Knowing which line is dominant tells you which fix to apply.] -->

## Tool-Output Cost: What Comes Back

The fourth type is tool-output cost: the logs, command results, search returns, and API responses that the agent receives when it calls a tool.

This one can spike without warning. A test suite run returns nine hundred lines of log output. A file read returns a two-thousand-line configuration file when you needed three lines from the middle of it. A web search returns full page content when you needed the title and a summary.

The underlying problem is that tools are often designed to return complete information, and completeness is not the same as relevance. The agent requested an action; the tool returned everything associated with that action; most of it is noise.

The fix is filtering and summarization, either in the tool itself or as an explicit step after the tool runs. Instead of reading a log in full, extract the failures and discard the passes. Instead of returning a full JSON payload, structure the tool to return only the fields the agent is likely to need. The general principle: the right moment to trim is before the output lands in context, not after.

| Tool output type | Reduction strategy |
|---|---|
| Test logs | Extract failures only; discard routine passes. |
| Search results | Filter to the top two or three relevant results. |
| JSON API payloads | Return only needed fields, not the full response. |
| File reads | Use line ranges or section headings instead of whole files. |
| Command output | Summarize before returning output to the agent. |

*Tool-output cost is best controlled before the output lands in context.*

## Connector and MCP Cost: The Integration Tax

The fifth type is connector and MCP cost, which has two components that are easy to conflate but behave differently.

The first component is schema cost: the structured description of what a tool can do, loaded at session start. This is a species of fixed cost — it exists whether or not you call the tool. Anthropic's work on MCP notes that raw tool schemas can become substantial context overhead, particularly if you have connected several servers and all of them are loaded simultaneously.

The second component is payload cost: what the tool returns when called. This is a species of tool-output cost — variable, unpredictable, and controlled by how the tool is designed.

The reason to treat these as a distinct category is that the intervention is different from both pure fixed cost and pure tool-output cost. For connector cost, the fix is scoping: enable only the servers you need for this task, design aggregator tools that return synthesized results rather than raw API payloads, and batch related calls so you are not making five separate round trips when one would do.

The practical implication is that every MCP server you connect to an agent carries a permanent per-session cost even on tasks that never invoke it. Three servers for a task that needs one means you are paying the schema overhead of two servers for no benefit.

## Reading a Broken Session

Here is a real audit structure applied to a session that started producing bad results.

```text
Fixed:
  AGENTS.md: long, containing both rules and examples
  CLAUDE.md: imports AGENTS.md plus duplicated rules
  MCP: GitHub, Slack, Drive, database all loaded

Per-task:
  Agent read entire docs/ folder

Accumulated:
  74 turns

Tool-output:
  900-line test log still in context
```

Every category has a problem. The question is fix order.

Fixed cost is the one to address first, because it affects every subsequent session, not just this one. Trim the duplicate rules between `AGENTS.md` and `CLAUDE.md`. Disable the MCP servers that are not relevant to the current task. This reduces the baseline before any task-specific loading happens.

Per-task cost is second. Replace "read the whole docs folder" with targeted file reads directed by whatever search or index is available. Identify why the agent went broad and add a narrowing instruction.

Accumulated cost comes third: start a fresh session using `session-handoff.md` to capture what matters from the current state, then let the old context go. Compaction is a partial measure; for 74 turns with degraded signal, a fresh start is more effective.

Tool-output cost is the quick win: replace the 900-line log with a failure excerpt. Five lines of relevant failures beats nine hundred lines of mixed passes and failures every time.

The lesson is not that you must fix everything at once. It is that naming the type before choosing the cure prevents you from applying the wrong fix — compacting a session that was broken by fixed cost from the first turn.

## What Would Change My Mind

I would revise the five-type model if tools exposed a better shared accounting system that divided context differently and proved more useful for intervention. The five categories work because each one maps to a distinct fix that practitioners can actually apply. If a better division emerged from tooling that made different categories visible and actionable, I would update the model. Today, no such accounting exists in standardized form across the tools in common use.

## Still Puzzling

How much fixed cost each tool hides from the user is genuinely unclear to me. Some tools expose context breakdowns; most do not. Estimating fixed cost for a given session requires either tool-specific instrumentation or working backward from visible behavior, neither of which is satisfying.

Whether tool retrieval will eventually replace always-loaded schemas is an open question with real stakes. If an agent could load a tool schema on demand — only when a call to that tool is imminent — the distinction between fixed cost and per-task cost would dissolve for connectors. Some systems are moving in this direction. Whether the latency trade-off is acceptable for real work is still being tested.

What context budget threshold predicts visible quality drop in real work is the question I find most practically important and most difficult to answer. The relationship between context window utilization and output quality is not linear, and it almost certainly varies by task type, model, and what exactly is in the context. The five-type model does not answer this; it answers what to do once you notice degradation, not when to start worrying.

---

## Exercises

**Warm-up**

1. *(Recall)* Name all five context cost types and give a one-sentence description of each. For each type, name the primary fix. *Tests whether you have the basic taxonomy and the corresponding interventions as separate, accurately paired items.*

2. *(Identification)* A colleague says their agent session has become slow and unreliable at turn sixty-two. They compact the conversation history, but nothing improves. What is the most likely explanation, and what would you tell them to check? *Tests comprehension of why compaction helps accumulated cost but not fixed cost.*

3. *(Recall)* What is the ordering principle for prompt caching, and why does stable material belong before variable material? *Tests retention of the cache-prefix rule and its practical implication for instruction layout.*

**Application**

4. *(Audit)* Using the token audit checklist from this chapter, audit a real agent session you have access to — or construct a plausible hypothetical session with at least three distinct context sources. Identify which cost type is largest, and describe one concrete action that would reduce it. *Tests ability to apply the five-type framework as a diagnostic tool rather than a taxonomy exercise.*

5. *(Rewriting)* A test run returned the following log to the agent: nine hundred lines of output, eight hundred of which are passing tests with a standard success message, one hundred of which contain actual failures with file names and line numbers. Rewrite the tool-output into a five-line diagnostic summary. *Tests the core skill for reducing tool-output cost: extraction of failure signals, discarding of noise.*

6. *(Design)* You are configuring a writing assistant agent that needs access to Google Drive (for source files), a web search tool, and a calendar (for scheduling context). You are working on a single chapter draft. Which of these three connectors should be active, which should be disabled, and what fixed cost does disabling one of them save? *Tests ability to apply the scoping fix to connector/MCP cost.*

**Synthesis**

7. *(Integration)* A session contains the following: a 4,000-word `AGENTS.md` (of which 1,200 words are worked examples that apply only to a different book project); 55 turns of conversation; three MCP servers active, one of which has not been called; and a search result set of twelve documents loaded in full when only two were needed. Diagnose which cost type is dominant, which interventions apply in what order, and what you would preserve in a `session-handoff.md` before starting fresh. *Tests integration of all five cost types as a unified diagnostic framework, plus the handoff mechanism from Chapter 2.*

8. *(Analysis)* The chapter argues that context degradation is a signal-to-noise problem, not just a volume problem. Using the concept of "context rot" from Chroma's research, explain why a context window filled to 60% capacity could produce worse results than a window filled to 80% capacity. Under what conditions would this be true? *Tests whether you understand the distractor mechanism, not just the budget metaphor.*

**Challenge**

9. *(Open-ended)* The chapter proposes five cost types based on origin and fix. A critic argues that this taxonomy is arbitrary — that all five are just "tokens that consume context," and the only real intervention is "load less." Construct the strongest version of this counterargument, then explain why the five-type model is still useful despite it. *Engages the chapter's foundational claim: that type-specific interventions produce better outcomes than generic reduction strategies, and tests whether you can hold the model provisionally while understanding its limits.*

## Prompts

### Figure 5.1 - Context window cost composition

Create a standalone D3 v7 HTML figure for "Context window cost composition" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: A horizontal bar representing a context window, segmented into five color-coded regions labeled Fixed, Per-task, Accumulated, Tool-output, and Connector/MCP. A second bar below shows the same window after optimization — same total size, but proportions shifted, with fixed and accumulated costs reduced. Caption: Context Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 5.2 - Context cost growth over conversation turns

Create a standalone D3 v7 HTML figure for "Context cost growth over conversation turns" using the pinned CDN. Use a timeline / progression structure with 4-6 labeled elements inferred from this concept: Line graph with x-axis showing conversation turns (0–80) and y-axis showing percentage of context window consumed. Three lines: one for fixed cost (flat horizontal line from turn 0); one for accumulated cost (gradual upward slope); one for tool-output cost (irregular spikes). Caption: The three dynamic cost types grow  Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
