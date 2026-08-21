# Chapter 10 — Agent Workflow Patterns That Actually Work

You ask: "Write this article, check it, and improve it." The agent writes a draft, says it checked it, and makes a few cosmetic edits. The result still has the same structural problems.

The issue is not that the agent cannot revise. The issue is that the builder and reviewer were the same context with the same blind spots.

---

There is a pattern that runs through almost every place where human organizations produce reliable work: the person who builds something is not the person who checks it. Not because builders are careless, but because building a thing and evaluating a thing require different orientations. The builder is anchored to the decisions made along the way — the structural choice made in the first hour, the framing that seemed right when the draft was fresh, the test that was skipped because the fix seemed obvious. An evaluator without that history sees the output as it actually is.

Agents do not automatically separate these roles. When you ask an agent to write something and then check it, the checking happens in the same context as the writing, with the same accumulated assumptions, the same fatigue toward the same blind spots. The agent will find surface errors. It will not reliably find the structural problem it built in.

The solution is to make role separation explicit in how you design agent work. Good agent work is patterned work — it uses recurring structures that assign distinct jobs to distinct phases, so that the thing that evaluates an output does not share the history of producing it.

---

The patterns that come up repeatedly in practice are not complicated, and they map onto recognizable shapes of work.

The builder-validator pattern is the most general. A builder produces output into a staging area. A validator checks that output against explicit criteria and returns only what is missing or wrong — not a general impression, not encouragement, not cosmetic suggestions. The validator has a job, not a mood. Then the builder revises. The separation works because the validator is given criteria, not latitude. An unscoped critic will over-report — it will find everything that could be different, which is everything. A scoped validator asks only whether the output satisfies the stated requirements, which is a much more tractable question.

The researcher-writer-editor pattern applies when the deliverable depends on both evidence and voice. A researcher gathers sources, checks claims, and returns a structured summary. A writer drafts from that summary. An editor checks the draft against the original sources and the writer's stated goals. Each role has a distinct input and a distinct output. The editor does not need to have been present for the research — in fact, it is better if they were not, for the same reason a fresh reviewer is better than a tired one.

The planner-executor-reviewer pattern is for multi-step work where the cost of a wrong direction is high. The planner reads the relevant files and produces a plan. The plan gets approved — by you, by a checklist, by an automated gate — before any execution begins. The executor works the plan. The reviewer checks the diff against the plan. The review is not "did this look good?" — it is "did this do what the plan said it would do?" The precision of that question is what makes the review meaningful.

The subagent delegation pattern keeps exploratory work from flooding the main context. When a task requires investigation — reading many files, trying several approaches, pursuing a line of research that may dead-end — a subagent does the exploration and returns a focused summary. The main context receives the conclusion, not the trail. This preserves the signal-to-noise ratio in the primary session and prevents the context rot that comes from loading everything the subagent touched.

The staging-before-output pattern applies whenever a draft must pass validation before it is treated as a deliverable. The draft goes to `staging/`. Validation runs against the project rules or an explicit anatomy. Only after validation passes does the output move to `outputs/`. This is not bureaucratic overhead — it is the same principle as a test suite that must pass before a merge. The staging directory is the gate.

The diff-first pattern is for changes with a broad blast radius. Before any edits are made, the agent states what it intends to change and why. You review the intention. The edits follow. This catches the case where the agent has interpreted a broad instruction in a way you did not intend — better to catch it before the files have changed than after.

The test-before-write pattern applies when code behavior is uncertain. Write the test first, confirm it fails, then write the implementation, confirm it passes. The test is the specification. This is not new advice — it is the core of test-driven development — but it has a specific application to agent work: when you cannot read the code closely enough to evaluate correctness, the test result is the signal you are actually relying on.

| Pattern name | Use when | What the validator needs | What to avoid |
|---|---|---|---|
| Builder-validator | Output quality matters. | Acceptance criteria and test commands. | Letting the builder validate itself. |
| Researcher-writer-editor | Evidence and prose both matter. | Source list and claim checklist. | Mixing evidence collection with drafting. |
| Planner-executor | The task spans many files. | A scoped plan and stop conditions. | Over-planning trivial edits. |
| Triage-then-act | Failure cause is unclear. | Symptoms, logs, and reproduction steps. | Jumping to fixes before diagnosis. |

---

Selecting among these patterns requires asking one question before anything else: who needs to evaluate this, and what do they need to not have seen?

For outputs where quality depends on judgment rather than rule-checking — a draft, a design, an argument — the evaluator should not have built the thing. Use builder-validator with a fresh context for the validator, or write the validator criteria so explicitly that the same agent can apply them without relying on its own taste. The criteria do the work that fresh eyes would otherwise do.

For outputs where correctness depends on whether something was done rather than how well it was done — a migration, a refactoring, a data transformation — the reviewer needs the plan and the diff, not the implementation history. The question is not "is this good?" but "does this match the specification?" That question can be answered without having been present for the work.

For exploratory tasks — research, debugging, competitive analysis — the main context should receive summaries, not logs. Design the subagent return value before you design the subagent task. "Return a concise summary with sources and open risks. Do not dump raw files" is not a stylistic preference; it is an instruction that controls what enters your primary context.

Three pattern prompts written out:

```text
PLANNER-EXECUTOR-REVIEWER
Read relevant files only. Produce a plan. Do not edit until approved.
After editing, review the diff against the plan.

STAGING-BEFORE-OUTPUT
Write draft to staging/. Validate against PROJECT_RULES.md.
Move to outputs/ only after validation.

SUBAGENT SUMMARY
Investigate X. Return a concise summary with sources and open risks.
Do not dump raw files.
```

These are starting points. The specific wording matters less than the structural principle each embodies: explicit roles, explicit criteria, explicit staging.

---

Patterns add overhead. A three-phase builder-validator-editor workflow is the wrong tool for a one-line edit, a quick lookup, or a task whose output you can evaluate at a glance. The overhead of role separation is worth it when the cost of a missed error is high, when the output will be relied on without further review, or when the task is complex enough that no single pass is likely to catch all the problems. It is not worth it for routine work where the simplest prompt is also the most efficient one.

The skill is calibration. Most practitioners err in one direction or the other: either they treat every task as simple and wonder why complex outputs have persistent problems, or they layer every task with multi-agent scaffolding and create coordination overhead that slows them down without improving results. The pattern vocabulary is useful precisely because it lets you name the structure you need for a task rather than reinventing it each time or defaulting to a one-size approach.

---

There is a layer below prompts worth mentioning here, because agent workflows increasingly reach it. When agents, scripts, and humans all need to call the same command-line interface, the design of that interface matters for agent reliability in the same way that folder structure matters for file reliability. A CLI that was designed for human eyes — colored output, progress spinners, verbose prose errors — will break when an agent tries to parse it. The agent reads `stdout` for results. If the result is mixed with log output, color codes, and progress indicators that were meant for a terminal, the agent is parsing noise.

The design rules for agent-friendly CLIs are the same principle at a different layer: make state explicit, make output parseable, make dangerous actions reversible or gated.

Primary results belong on `stdout`. Errors, logs, and progress belong on `stderr`. Exit codes should be meaningful — zero for success, nonzero for failure, with distinct codes for distinct failure modes. A `--json` flag should produce structured output that an agent can parse without string matching. A `--dry-run` flag should show what would happen without doing it. A `--quiet` flag should suppress everything except results. The business logic should live behind the CLI adapter, not inside the argument parsing:

```text
core logic -> CLI adapter -> terminal/shell adapter -> packaging adapter
```

This separation means the core logic is testable without invoking the CLI, and the CLI is testable without the packaging layer. The agent that calls your CLI receives the same guarantees as the human who does — and the human who designed it actually thought about what both needed.

---

The same principle that motivates pattern separation in prompts motivates separation in how reusable material is organized. There is a tendency to put everything into project rules because project rules are easy to add to and always loaded. The cost is that everything-always-loaded means the context is always carrying weight that most tasks do not need.

The categories are worth keeping distinct:

A global instruction applies to every project, every time. It belongs in tool settings or a global config file.

A project rule applies to one project. It belongs in `PROJECT_RULES.md` or `CLAUDE.md`.

A template is a reusable output skeleton — the shape a deliverable should take. It belongs in `templates/`.

A skill or workflow is a reusable procedure with steps and checks. It belongs in `skills/` or `workflows/`, loaded only when the relevant task is active.

A status file captures current state and next actions. It is ephemeral and session-specific.

A manifest maps what to read and what to ignore. It is project-specific and loaded early.

Keeping these separate reduces the fixed context cost of every session and makes each artifact easier to test independently. A template that answers "what shape should the output have?" is different from a workflow that answers "what steps should the agent follow?" — and conflating them in one rules file means both are always loaded, whether the current task needs them or not.

---

Three misconceptions are common enough to name.

More agents is not more agentic. More agents can mean more coordination overhead, more context fragmentation, and more places for state to get lost. The right number of agents for a task is the minimum needed to separate the roles that need to be separated. For many tasks, that number is one.

A critic should not find all problems. An unscoped critic will over-report, and over-reporting is its own failure mode — you spend time triaging false positives and lose the signal in the noise. Ask for correctness against criteria, unmet requirements, or factual risks. The more specific the scope, the more useful the output.

Testing is not only for software. A chapter draft can be tested against its structural requirements, its citation accuracy, its claim density, its adherence to a stated voice. A plan can be tested against the acceptance criteria it is supposed to satisfy. Validation is a general concept. The criteria just need to be written down.

---

Two questions in this space remain genuinely open. The first is when subagents become coordination overhead — at what point does the cost of spawning, briefing, and integrating a subagent exceed the value of keeping the exploration out of the main context? The threshold is probably task-dependent and tool-dependent, and I do not think there is a principled answer yet. The second is whether tool-native review modes will eventually replace custom validator prompts — whether future agents will have reliable self-evaluation capabilities that make role separation unnecessary for ordinary tasks. The current evidence and practice still favor structured workflows. Whether that is because self-evaluation is genuinely hard or because we have not built the right tools for it is not yet clear.

---

## LLM Exercises

1. Take a task where you asked an agent to produce and evaluate something in the same prompt and got a weak result. Rewrite it as builder-validator with explicit validator criteria. Ask the model to apply the validator criteria to the original output and report only what was missing or wrong.

2. Write a validator checklist for a chapter draft in this book series — the specific structural requirements, factual risks, and quality markers a validator should check. Then ask the model to apply that checklist to one chapter you have already produced. Notice which problems the checklist surfaces that a general "review this" prompt would not.

3. Identify one task in your current work where a simple one-shot prompt is genuinely sufficient — where the overhead of builder-validator separation would cost more than it saves. Write a sentence explaining why. Then identify one task where you have been using one-shot prompts that should probably have been builder-validator. Write the validator criteria for it.

![Reliable agent workflows separate generation from validation so one pass creates work and another checks it against constraints.](images/10-agent-workflow-patterns-that-actually-work-fig-01.png)
*Figure 10.1 - Builder-Validator Workflow*

![Different agent workflows fit different task risks: writing, research, refactoring, validation, and delivery need different control structures.](images/10-agent-workflow-patterns-that-actually-work-fig-02.png)
*Figure 10.2 - Pattern Selection Matrix*

## Prompts

### Figure 10.1 - Builder-Validator Workflow

Create a standalone D3 v7 HTML figure for "Builder-Validator Workflow" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: Reliable agent workflows separate generation from validation so one pass creates work and another checks it against constraints. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 10.2 - Pattern Selection Matrix

Create a standalone D3 v7 HTML figure for "Pattern Selection Matrix" using the pinned CDN. Use a process flowchart structure with 4-6 labeled elements inferred from this concept: Different agent workflows fit different task risks: writing, research, refactoring, validation, and delivery need different control structures. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
