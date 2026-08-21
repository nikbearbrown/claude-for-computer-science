# Chapter 15 — Does Any of This Actually Help? Measuring Your Workflow
*The honest question is not whether your setup feels better. It is whether it works better.*

You have built the system. `AGENTS.md`, `_MANIFEST.md`, project rules, session handoffs, an archive discipline, MCP scoping. The project feels more organized. Sessions feel cleaner. You are more confident that the agent is working with the right material.

But does it actually produce better output? Does it save time? Does it prevent the failures it was designed to prevent?

Those questions are not ungrateful. They are the point. Instruction files are not sacred. They cost context. They can help, hurt, or do nothing depending on the task, the tool, the repository, and the quality of the rules themselves. A rules file that took two hours to write and makes no measurable difference to output is two hours of context cost with no return. The honest practice is to measure.

## What Measurement Actually Requires

The instinct, when something feels better, is to trust the feeling. And the feeling is not useless — a session that consistently produces wrong outputs or constant re-explanation has something wrong with it, and noticing that is real information. But feelings conflate many things. A project that feels more organized might be producing better agent output, or it might just have a cleaner folder structure that satisfies a different instinct entirely. You cannot tell from the feeling alone.

What measurement requires is a comparison: the same task, with and without the intervention, with outcomes you can record. This does not require a research lab. It requires a disposable copy of the repository, a task suite you can run twice, and a handful of metrics you write down.

Recent empirical work has begun testing agent manifests, AGENTS-style files, and behavior-driven agent evaluation — results are preliminary and details will change, but the direction is consistent: the effect of instruction files is task-dependent, tool-dependent, and sensitive to rule quality in ways that "this rule sounds wise" does not predict. [verify exact results before publication: arXiv 2601.20404, arXiv 2509.14744, arXiv 2604.03362] The lesson that is already stable: do not assume a rule helps because it sounds wise. Measure it.

![Rule effectiveness by task type](images/15-does-any-of-this-actually-help-measuring-your-workflow-fig-01.png)
*Figure 15.1 - Rule effectiveness by task type*

<!-- → [CHART: Scatter plot or bar chart showing rule effectiveness by task type — x-axis: task categories (repo summary, bug fix, multi-file feature, test repair, security change), y-axis: improvement in acceptance rate with rules vs. without; show that effectiveness varies by task type rather than being uniformly positive; caption: "Instruction file benefit is not uniform across task types. The tasks where rules help most are not always the ones you expect."] -->

## The Mini-Benchmark

The benchmark is not a research study. It is a structured comparison designed to catch obvious waste and verify obvious wins. Run the same five-task suite twice — once with your rules file active, once without it on a disposable copy of the repository:

```text
1. Repo summary
2. Narrow bug fix
3. Multi-file feature
4. Test repair
5. Security-sensitive change
```

These five tasks are chosen because they span different kinds of agent behavior. A repo summary tests whether the agent orients correctly. A narrow bug fix tests precision. A multi-file feature tests scope management. A test repair tests whether the agent understands the test contract. A security-sensitive change tests whether the rules prevent risky behavior — the one category where a rules file is most likely to earn its keep.

For each run, record:

```text
- Acceptance rate: how many outputs were accepted without modification?
- Manual edits: how many post-generation changes were required?
- Time to valid diff: how long from task issue to acceptable output?
- Token or cost estimate: what did the session cost?
- Risky behavior: did the agent touch files it should not have?
- Out-of-scope files touched: what did it reach for that was not on the task?
```

Then measure the maintenance cost of the rules file itself:

```text
- Rules file word count
- Number of adapters generated or hand-maintained
- Number of stale rules found
- Number of tool-specific claims needing re-verification
- Number of private, large, or generated files accidentally loaded
```

The second list is easy to skip and important not to. A rules file that prevents one risky action per week is worth its context cost. A rules file that causes two failed runs per week because it contains a stale build command is not. The maintenance cost is part of the ledger.

## Reading the Results

Here is what a real benchmark found on a 4,800-word rules file:

```text
Rules file before:
4,800 words
duplicates README
contains old build command
contains useful no-delete rule
contains useful test command
```

The benchmark:

```text
No-delete rule: prevented one risky action across five tasks.
Old build command: caused two failed runs.
Style guidance: made no measurable difference to output quality.
README duplication: no effect on behavior, context cost of several hundred tokens.
```

The result is not that the rules file was useless. It is that most of it was useless, and one part of it was actively harmful. The fix was not to rewrite the rules file. It was to prune it:

```text
Rules file after:
900 words
keeps: no-delete rule, test command, manifest rule
removes: stale build command, duplicated README, style guidance that moved nothing
```

| Rule | Word count | Benchmark result | Keep or remove | Reason |
|---|---:|---|---|---|
| Read manifest first | 24 | Improved source selection. | Keep | Changes behavior on most tasks. |
| Never delete files | 18 | Prevented unsafe cleanup. | Keep | Low cost, high safety value. |
| Long style manifesto | 640 | No measurable improvement. | Remove | High context cost. |
| Tool-specific command list | 310 | Helped only one workflow. | Defer | Move to adapter or task guide. |

The lesson from this example is not that 900-word rules files are always right. It is that the pruning criterion is behavioral: keep rules that change behavior, remove rules that only make you feel safer. "Never delete files without archiving first" changed behavior. "Write clear, well-commented code" did not.

## What Small Benchmarks Can and Cannot Do

A five-task benchmark is noisy. Two failed runs on a stale build command is suggestive, not conclusive. The old build command might have been triggered by an unusual path in the task suite; it might not fail on the tasks you run most often. Small benchmarks are calibrated for pruning obvious waste and verifying obvious wins. They are not calibrated for declaring that your setup is correct.

The honest scope of a mini-benchmark: it catches rules that are actively harmful (causing failures), rules that are invisible (making no difference), and rules that earn their cost (preventing risky behavior). For everything in between — rules that sometimes help, rules whose effect is subtle, rules that matter for rare but high-stakes tasks — you need more data than five tasks can give you.

Run the benchmark every few months, not every week. Run it when you have made significant changes to the rules file. Run it when you suspect a rule has gone stale. The goal is not continuous measurement. The goal is periodic calibration.

## Four Misconceptions Worth Naming

**"If a rule is good, more detail is better."** Detail costs context. A rule that needed three sentences in the first draft rarely needs ten sentences six months later. Move detail into task-specific documents when it only applies to specific tasks. The global rules file should stay short enough to read before every session without cost.

**"One successful run proves the setup."** One run proves that the setup did not fail on that run. Repeated tasks, or at least comparable before/after trials, are the minimum unit of evidence. A single success on a security-sensitive task might mean the no-delete rule worked, or it might mean the task did not trigger the relevant behavior.

**"Measurement is only for teams."** A single practitioner can still track time to valid diff, number of manual edits, and whether the agent touched out-of-scope files. The ledger does not require a team to maintain. It requires the habit of writing things down.

**"If it is in the guide, it belongs in my project."** The implementation order in this book is staged deliberately. A solo writing project may need folder structure, a manifest, status files, and a no-delete rule. It probably does not need generated adapters, MCP governance, or a structure-audit script running in CI. The full framework is a menu, not a checklist. Take what the current project needs; leave what it does not.

![Rule keep-prune-defer decision tree](images/15-does-any-of-this-actually-help-measuring-your-workflow-fig-02.png)
*Figure 15.2 - Rule keep-prune-defer decision tree*

<!-- → [INFOGRAPHIC: Decision tree for "does this rule belong in my project?" — starting from "have you measured its effect?" branching to "does it change behavior on tasks you run?" and "what is its maintenance cost?" ending in keep/prune/defer decisions; caption: "The question is not whether the rule sounds right. It is whether it earns its context cost."] -->

## What Would Change My Mind

I would revise this chapter if strong, replicated studies showed that well-structured instruction files consistently improve outcomes across tools without meaningful context cost. Until then, the safest claim is local measurement. The current empirical picture is that effects are real but variable, tool-dependent, and sensitive to rule quality in ways that theory does not predict well. Measure locally. Prune often.

## Still Puzzling

- Which task types benefit most from rules?
- What metrics best capture review burden?
- Can tools automatically report whether rules were used?

---

## LLM Exercises

> These exercises require a real project and a disposable copy of the repository. The point is to generate actual data, not to reason about what the data would show.

**Exercise 1: Run the mini-benchmark.** Make a disposable copy of your most active project. Run the five-task suite — repo summary, narrow bug fix, multi-file feature, test repair, security-sensitive change — once with your rules file active and once without it. Record all six output metrics for both runs. Write two sentences summarizing what the comparison showed.

**Exercise 2: Audit the rules file.** After the benchmark, go through your rules file rule by rule. For each rule, classify it as: changed behavior in the benchmark, made no measurable difference, or caused a failure. Remove every rule in the second and third categories. Record the before and after word counts.

**Exercise 3: Add one enforcement mechanism.** Identify the one rule from your pruned rules file that matters most — the one whose violation would cause the worst outcome. Design a lightweight enforcement mechanism for it: not a rule that asks the agent to follow it, but a structural constraint that makes violation harder. What does the mechanism look like? What does it not protect against?

---

## Exercises

**Warm-up**

1. *(Basic recall)* What are the six output metrics the mini-benchmark records for each run? For each metric, state what it measures in one sentence. *What this tests: ability to reproduce the benchmark's measurement framework.*

2. *(Vocabulary)* What does "context cost" mean in the context of a rules file? Why does a rule that "sounds wise" but makes no behavioral difference still have a cost? *What this tests: understanding the ledger — that passive context consumption is a real cost even when nothing fails.*

3. *(Reading results)* In the worked example, the style guidance made no measurable difference. What are two possible explanations for why a rule could exist in a file for months without producing measurable effect? *What this tests: reasoning about null results, not just successes.*

**Application**

4. *(Benchmark design)* You are working on a data pipeline project that processes financial transactions. Adapt the five-task suite to your project: replace each generic task with a specific equivalent that would actually exercise your rules file. For each task, name one rule from your (hypothetical) rules file that the task is most likely to trigger. *What this tests: applying the benchmark structure to a concrete domain.*

5. *(Pruning)* A rules file contains the following entries: "Always run tests before committing," "Never modify files in `outputs/` directly," "Write docstrings for all functions," "Use the project's linter configuration," "Prefer explicit variable names," "Archive superseded files rather than deleting them." Apply the pruning criterion from this chapter — keep rules that change behavior, remove rules that only make you feel safer. Which would you keep and which would you remove? Justify each decision. *What this tests: applying the behavioral pruning criterion to a realistic set of rules.*

6. *(Maintenance cost)* A rules file has 3,200 words, references three tool-specific configuration paths that may have changed, duplicates the project README's getting-started section, and was last updated eight months ago. Estimate the maintenance cost items from the checklist. Which item would you address first, and why? *What this tests: applying the maintenance cost ledger to a concrete case.*

**Synthesis**

7. *(Integration)* Chapter 12 introduced the anti-patterns checklist as a diagnostic tool. Chapter 15 introduces the mini-benchmark as a measurement tool. How do these two tools complement each other? If you ran the anti-patterns checklist and found no failures, but the benchmark showed no improvement from your rules file, what would you conclude? *What this tests: understanding the relationship between structural diagnosis and behavioral measurement.*

8. *(Framework scope)* The chapter says the full framework is "a menu, not a checklist." A solo practitioner working on a technical writing project asks which parts of the framework they need. Using the staged implementation logic from this chapter, describe what you would recommend they implement first, second, and defer. Justify the sequence. *What this tests: applying the framework selectively rather than wholesale.*

**Challenge**

9. *(Open-ended)* The "Still Puzzling" section asks whether tools could automatically report whether rules were used. Design a lightweight logging convention — something that does not require tool vendor support — that would let a practitioner reconstruct, after a session, which rules in their rules file were relevant to the agent's decisions. What would the log look like? What would it take to produce it? What could it not capture, and why does that matter? *What this tests: designing an instrumentation layer for an inherently opaque process, with honest accounting of what remains unobservable.*

## Prompts

### Figure 15.1 - Rule effectiveness by task type

Create a standalone D3 v7 HTML figure for "Rule effectiveness by task type" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: Scatter plot or bar chart showing rule effectiveness by task type — x-axis: task categories (repo summary, bug fix, multi-file feature, test repair, security change), y-axis: improvement in acceptance rate with rules vs. without; show that effectiveness varies by task type rather than being uniformly positive; caption: Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 15.2 - Rule keep-prune-defer decision tree

Create a standalone D3 v7 HTML figure for "Rule keep-prune-defer decision tree" using the pinned CDN. Use a process flowchart structure with 4-6 labeled elements inferred from this concept: Decision tree for "does this rule belong in my project?" — starting from "have you measured its effect?" branching to "does it change behavior on tasks you run?" and "what is its maintenance cost?" ending in keep/prune/defer decisions; caption: "The question is not whether the rule sounds right. It is whether it earns  Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
