# Chapter 13 — The Failure-Mode Catalog: Root Causes and Mitigations

Your agent keeps reading archive files despite the manifest. You add a louder rule. It happens again.

The rule was not the root cause. The root cause was that your task prompt said "summarize everything," and the archive was physically inside the searched tree. No rule fixes that. Moving the archive does.

---

The most common mistake in diagnosing agent failures is stopping at the symptom. The agent used the wrong file. The agent trusted stale state. The agent over-reported in a review. Each of these is a symptom — an observable output — and the temptation is to treat it as a cause. Add a louder rule. Repeat the instruction. Ask more carefully. These interventions sometimes work, and when they do, they reinforce the habit. Then the same symptom appears in a different situation and the louder rule does nothing, because the underlying structure that produced the symptom has not changed.

A failure mode has three parts: a pattern, a trigger, and a mitigation. The pattern is the workflow structure that is supposed to be working — manifest tiers, ignore files, a reviewer prompt, a session handoff. The trigger is the specific condition that caused the pattern to break. The mitigation is the structural change that prevents the trigger from firing again. If you skip the trigger, you guess at fixes. If you find the trigger, the mitigation is usually obvious.

---

Let's work through the catalog.

**Manifest tiers, failure: agent reads Tier 3 material.** The pattern is a tiered manifest that designates some files as canonical and others as archive. The trigger is a vague task prompt — "summarize everything," "check the project" — combined with the archive being physically inside the searched directory tree. The manifest says "treat these as archive," but the search includes them anyway because the scope of the task invited a whole-tree search. The mitigation is to scope the prompt more tightly, move archived material outside the active directory physically, and use tool-native ignore configuration where it is available. The louder rule does not help because the agent is not defying the rule — it is executing a search whose scope you did not restrict.

**Ignore files, failure: secrets remain visible to the agent.** The trigger is confusing `.gitignore` with a security boundary. `.gitignore` tells git which files not to track. It does not necessarily tell an agent which files not to read. An agent with broad file-access permissions in a workspace that contains `.env` files, API keys, or credential files may read them regardless of what `.gitignore` says. The mitigation is to keep secrets outside the workspace entirely — not in a gitignored file within the project tree, but in a location the agent cannot reach.

**Tool adapters, failure: rules duplicate or drift across tool-specific files.** The trigger is maintaining multiple hand-written instruction files for different tools. Each file starts as a copy of the canonical rules, acquires tool-specific additions, and begins diverging. Six months later the Cursor rules say one thing, the Claude rules say another, and neither matches the project's current state. The mitigation is a canonical source with generated adapters — one file that is the truth, several files that are outputs. Do not edit the outputs by hand. Add a do-not-edit header to generated files and, for teams, a CI check that flags stale generated adapters.

**Memory index, failure: critical pointer too low.** The trigger is an index that has grown long enough that the most important pointers — the canonical files, the current phase, the acceptance criteria — are no longer near the top. Long context weights material near the beginning and end more heavily than material in the middle. If your index has accumulated entries over many sessions and the critical ones have migrated toward the middle, they will be less reliably acted on. The mitigation is periodic auditing of the top section of any index file to ensure the highest-priority pointers remain prominent.

**`status.md`, failure: agent trusts stale state.** The trigger is a status file that was not updated at the end of a session. The agent opens the next session, reads `status.md`, and acts on a description of the project that was accurate two sessions ago. The mitigation is an end-of-session ritual — a prompt or checklist that ensures the status file is updated before context clears. The status file is only as good as the last update, and the last update only happens if updating it is a deliberate step rather than something that happens when you remember.

**Handoff, failure: continuity lost across sessions.** The trigger is a missing handoff — specifically, missing decisions and the reasons behind them. A handoff that lists next actions but omits the reasoning behind the current approach will produce a fresh session that rediscovers the same dead ends the previous session already eliminated. The mitigation is a template that requires decisions and their rationale, not just a task list.

**Long session, failure: quality drifts.** The trigger is accumulated distractors — the dead-end debugging paths, the superseded plans, the redundant file reads that build up over a long session until the signal-to-noise ratio drops below useful. The mitigation is a handoff plus a fresh session, not a compaction that preserves the noise in compressed form.

**Reviewer, failure: over-reports gaps.** The trigger is an unscoped critique. When a reviewer is asked to find everything that could be better, it finds everything that could be better, which is everything. The result is a long list of observations with no signal about which ones matter. The mitigation is to scope the reviewer to specific criteria — correctness against requirements, unmet acceptance criteria, factual risks — and nothing else.

**Vendor figures, failure: bad optimization target.** The trigger is treating unverified benchmark claims as design parameters. You optimize for a context-window size, a speed number, or a cost figure published by a vendor, and the figure does not match what you observe locally. The mitigation is to measure locally before optimizing. Vendor figures are useful for rough planning. They are not substitutes for empirical measurement in your specific workflow.

**Large data, failure: agent reads full dataset or the repository bloats.** The trigger is committing full data files to the project tree or leaving them in the active context. The mitigation is to ship a sample, document the fetch procedure for the full dataset, add the full data files to ignore configuration, and summarize before reading when the data is large enough to flood context.

**Generated adapters, failure: hand-edited generated file.** The trigger is the absence of a clear signal that a file is generated. Someone edits the generated Cursor rules directly, the edit gets overwritten on the next build, and the change is lost. The mitigation is a prominent do-not-edit header at the top of every generated file and, for collaborative projects, a CI check that detects when a generated file has been modified directly rather than through the source.

| Pattern | Failure | Trigger | Mitigation |
|---|---|---|---|
| Wrong source | Agent edits stale file. | Similar drafts in one folder. | Separate current and archive; name canonical source. |
| Context flood | Agent loses instruction discipline. | Too many files or logs loaded. | Narrow retrieval and summarize tool output. |
| Adapter drift | Tools follow different rules. | Copied instruction files diverge. | Use one shared source plus thin adapters. |
| Unsafe operation | File is deleted or overwritten. | No approval or archive policy. | Add archive-first rules and approval gates. |

---

If you are applying this catalog to an existing project, the order in which you make structural fixes matters. The temptation is to start with the most sophisticated control — a permission profile, an audit log, a CI check for stale adapters. These are useful, but they are not the highest-leverage intervention for a project that still has old drafts beside current drafts and no manifest.

Build in this order. First, create a clean folder structure — current work physically separated from archive, outputs separated from source, generated files separated from hand-maintained files. Second, add the four core files: `_MANIFEST.md`, `PROJECT_RULES.md`, `status.md`, and `session-handoff.md`. Third, make one file the canonical instruction source and add thin adapters for each tool you use. Fourth, add a structure-audit step — even a simple script that checks whether the manifest exists and whether the archive directory is outside the active tree. Fifth, add validator checklists for the outputs that matter most. Sixth, add governance controls: permission profiles, allowlists, audit logging.

This order matters because each layer builds on the one before it. A permission profile applied to a project with no manifest is governance without structure — it constrains the agent's actions without giving the agent a reliable picture of what it is working in. The manifest comes first because every other control depends on there being a coherent description of the project.

---

The failure-mode catalog is a diagnostic tool, not a checklist. The same symptom can have multiple causes, and the catalog is organized by pattern and trigger rather than by symptom precisely because the symptom-to-cause mapping is not one-to-one. An agent that uses the wrong file might be failing at manifest tiers, or at ignore configuration, or at long-session quality drift, or at stale status — and the mitigation for each is different. Diagnosing by symptom alone means you might fix one cause and leave the others untouched, observe apparent improvement, and conclude the problem is solved until the other causes fire.

The discipline is to write failure reports with all three parts: symptom, trigger, mitigation. Not "the agent used the old outline" but "the agent used the old outline because `outline.md` and `TIKTOC.md` disagreed and neither was marked canonical in the manifest." The trigger is where the information lives. The mitigation is almost always to put that information in the structure rather than in a prompt.

Prompt fixes fade. Structural fixes persist. A louder rule in `PROJECT_RULES.md` will be followed until the conditions that make it hard to follow reappear, and then it will not be followed again. A manifest that marks `TIKTOC.md` as Tier 1 and moves `outline-old.md` to `archive/` changes what the agent sees before any instruction is read.

---

Two things complicate the catalog in practice. The first is that not every failure has one cause. Some failures combine stale state, a vague prompt, and tool-specific loading behavior, and the mitigation requires addressing all three. The catalog is useful for identifying which causes are in play, not for guaranteeing that fixing one cause fixes everything.

The second is that tool loading and ignore behavior differ across tools, which means a failure that appears in one tool may not appear in another using the same project structure — and a mitigation that works in one tool may not generalize. `.gitignore` behavior, instruction-file precedence, and context-loading heuristics are all tool-specific. Before concluding that a structural fix is sufficient, verify it in the tool or tools you actually use.

---

Three open questions sit underneath this catalog and do not yet have clean answers. The first is whether tools will eventually detect stale manifests automatically — whether an agent will flag "the manifest says this file is Tier 1 but it has not been modified in six months and there are newer files with similar names" without being asked. That would catch a large class of stale-state failures before they produce bad outputs. The second is which failures are most amenable to structural prevention versus behavioral intervention — whether there are failure modes where the mitigation really is a better prompt rather than a structural change. My current belief is that most persistent failures are structural, but I hold that loosely. The third is how teams should collect failure reports without creating bureaucracy — how to make the discipline of writing symptom-trigger-mitigation reports light enough that it actually happens rather than being skipped under time pressure. The answer probably involves making the report format very short and very specific, but the right format has not been established.

---

## LLM Exercises

1. Take one agent mistake from your recent work — something the agent did that it should not have. Write a three-part failure report: symptom, trigger, mitigation. Ask the model to identify whether the mitigation you proposed addresses the trigger or only the symptom.

2. Take your current project's instruction files — all of them, from all tools — and ask the model to identify which entries are advisory rules that depend on the agent's cooperation, and which could be converted into structural constraints. For each advisory rule, propose what the structural equivalent would be.

3. Pick one failure mode from the catalog and design a test for it — a specific scenario you could run in your current project to determine whether the mitigation is working. The test should produce a clear pass or fail, not a general impression. Ask the model to propose a second test for the same failure mode that your first test might miss.

![Agent failures become easier to fix when symptoms are traced to root causes and then matched to mitigation patterns.](images/13-the-failure-mode-catalog-root-causes-and-mitigations-fig-01.png)
*Figure 13.1 - Failure Diagnosis Pipeline*

![Failures can originate from project structure, context selection, tool configuration, permissions, or validation gaps.](images/13-the-failure-mode-catalog-root-causes-and-mitigations-fig-02.png)
*Figure 13.2 - Failure Root-Cause Layers*

## Prompts

### Figure 13.1 - Failure Diagnosis Pipeline

Create a standalone D3 v7 HTML figure for "Failure Diagnosis Pipeline" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: Agent failures become easier to fix when symptoms are traced to root causes and then matched to mitigation patterns. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 13.2 - Failure Root-Cause Layers

Create a standalone D3 v7 HTML figure for "Failure Root-Cause Layers" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: Failures can originate from project structure, context selection, tool configuration, permissions, or validation gaps. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
