# Chapter 1 — Why Your Agent Reads the Wrong File

You asked your agent to summarize the current product plan. It answered confidently. It sounded useful. Then you noticed: the answer was drawn from `product-plan-old.md`, not `product-plan.md`.

Nothing in the response looked broken. The model did not say it was using the wrong file. It did what agents do — it inferred from nearby files, filenames, retrieval hits, timestamps, and whatever context had already been loaded. The bad answer did not begin in the model. It began in the folder.

That is the first thing to understand about working with agents on local files: the filesystem is part of the prompt.

---

Here is what actually happens when an agent starts a task involving your project. It does not enter with your mental model. It has no memory of which file is canonical, no recollection of why there are three drafts of the same document, no sense of which one you were editing last Tuesday. A human sees `draft-final.md`, `draft-final-real.md`, and `draft-final-june.md` and immediately reconstructs the story — because they lived through the story. An agent sees three plausible candidates for the same role.

The agent then does something entirely reasonable given what it has: it infers. It looks at filenames for clues. It checks paths for hierarchical signals. It considers what was retrieved by a search and what was already loaded in context. If one file was recently touched, that is a weak signal toward currency. If a filename contains "old," that is a weak signal toward irrelevance. The agent weighs these signals and chooses.

The problem is that these signals are weak, and in a typical working folder, they point in several directions at once. The word "final" appears in the name of your least-final draft. The file you're actively using hasn't been touched since a long edit session last month, so its timestamp looks old. The archived file has a longer, richer document structure than the active one, so it looks more authoritative to a model that interprets length as signal.

None of this is a bug. It is exactly what you would want an agent to do if you could not give it explicit instructions. The problem is that you usually can give it explicit instructions — not by writing a better prompt, but by building a better folder.

---

Let's think about what the agent is actually doing when it reads a directory. It is not doing what a file explorer does when it shows you a list of names. It is building a model of your project from everything it can observe: the names, the paths, the structure of the hierarchy, the content it has sampled, and anything it has already retrieved in this session. That model becomes the working context within which it interprets your instruction.

This is why Anthropic's context-engineering guidance treats paths, filenames, and folder hierarchy as part of how agents build useful context — not metadata about context, but context itself. The path `archive/2025-drafts/chapter-v2.md` carries a different semantic load than `chapter-v2.md` sitting directly in the project root, even if the file contents are identical. One path says "this is historical." The other says "this belongs here, at the top level, with everything else that matters."

The implication is significant. You can communicate to an agent before you've written a single instruction file. You can communicate through the shape of the folder. And conversely, a badly shaped folder will miscommunicate to the agent before you've written anything — including your careful prompt explaining which file to use.

---

There is a second problem layered on top of the wrong-canonical-file problem, and it operates differently. It is what Chroma's context-rot research describes: adding more context does not monotonically improve performance. There is a point past which more material in the context window makes the agent's answer worse, not better.

The mechanism is competition. When an agent is holding a lot of text in context, relevant material has to compete with irrelevant material for the model's attention. This is not a failure of attention in the human sense — it is a structural property of how large language models process sequences. Information near the beginning and end of a context tends to be weighted more heavily than information in the middle. If your canonical file gets loaded into a long, cluttered context, it may be present but effectively buried.

This matters for folder design because the most common response to "the agent got confused" is to add more material — more explanation, more files, a README, a manifest. Sometimes that helps. Often it makes the problem worse by giving the agent more to weigh. The better intervention is subtraction: move the material that shouldn't be in context out of reach, so the agent never loads it in the first place.

The three recurring failures, stated plainly, are:

**Wrong canonical file.** The agent selects an old or secondary file and treats it as source of truth. This usually happens because the canonical and non-canonical files are topically similar and live in the same folder.

**Over-broad reading.** The agent reads too many files and loses signal in noise. This usually happens when a task prompt is broad enough to invite a whole-folder search, and the folder contains many plausible-looking files.

**Archived-vs-current confusion.** Old work sits beside active work, so the agent treats both as live. This is the most common form of the first two failures combined, and it is almost always a folder structure problem.

---

Consider what a folder looks like before and after this kind of structural intervention.

Before:

```text
project/
  chapter.md
  chapter-final.md
  chapter-final-OLD.md
  chapter-notes.md
  chapter-output.html
```

The agent is asked to revise the chapter. Which file should it read? A human who built this folder knows. The agent has to infer from names alone. `chapter.md` might be the live version, or it might be an early draft that was superseded by `chapter-final.md`. `chapter-final-OLD.md` appears to be old, but "final" also appears in its name — the "OLD" suffix was applied after the fact, which the agent cannot know. `chapter-output.html` is probably generated output, but it might also contain the canonical version if someone edited it directly.

After:

```text
project/
  current/
    chapter.md
  research/
    chapter-notes.md
  outputs/
    chapter-output.html
  archive/
    2026-06-old-drafts/
      chapter-final.md
      chapter-final-OLD.md
```

Now the structure teaches the rule before any instruction is read. `current/` is active work. `archive/` is historical. `outputs/` is generated material. `research/` is supporting context, not the work itself. An agent navigating this directory receives a clear, redundant signal about what belongs where. The signal is not in a prompt. It is in the shape of the thing.

A good folder structure is a prompt the agent cannot miss.

---

There is an important qualification here. Structure reduces the risk of wrong-file selection; it does not eliminate the possibility, and it does not enforce any constraint. An agent with broad file-access permissions can still read `archive/` if it decides to. A poorly scoped task prompt can still invite whole-project retrieval. Structure is a probabilistic intervention, not a security boundary. For operations where the cost of a wrong-file error is high — overwriting production configuration, generating output that goes directly to users, modifying files that trigger downstream processes — you still need explicit gates, tests, hooks, or human review steps. The folder shape gets you most of the way toward correct behavior in ordinary cases. It does not substitute for verification in high-stakes cases.

This distinction matters because there is a temptation, once you understand folder structure as a communication tool, to over-rely on it. The reasoning goes: "I have a clean `current/` directory, so the agent will always use the right file." That is mostly true for most tasks. It is not a guarantee, and treating it as one creates exactly the kind of silent failure that started this chapter — the agent answered confidently, the answer sounded useful, and no one checked.

---

Three misconceptions come up often enough to address directly.

The first is that the agent will read everything and figure it out. Reading everything is expensive, and as the context-rot research makes clear, it can make the answer worse. The context window is not a neutral bucket. Irrelevant material competes with relevant material, and a big flat folder of similar-looking files is one of the best ways to trigger that competition.

The second is that bigger context windows solve this. Larger windows do help at the margins — they delay the point at which the window fills entirely, and they allow more material to be held simultaneously. But context rot is gradual. A huge window full of stale or similar files is still noisy. The degradation happens earlier than the formal limit, and it happens in proportion to the density of topically similar content. Bigger windows are not a substitute for folder hygiene.

The third is that a filename containing "old" will cause the agent to ignore a file. Maybe. But "old" in a filename is weak metadata — it is a note left by a human for other humans, not a structured signal that agents reliably interpret as "exclude from context." `archive/` is structurally stronger because it is a path component, not a name suffix, and because path-based organization is something agents are trained to interpret as hierarchical meaning. If you need an agent to reliably ignore a file, move it out of the active directory rather than renaming it.

---

The practical lesson is a small one, and it is worth stating plainly: before you diagnose a bad agent answer as a model problem, check whether it is a folder problem.

A quick diagnostic: find the folder you used. List every file that could plausibly be mistaken for the canonical one — not every file you know is irrelevant, but every file an agent without your context might reach for. If there are more than one or two, the structure is ambiguous. Move old drafts into `archive/`. Move generated output into `outputs/` or `build/`. Write one sentence in a `_MANIFEST.md` or `CLAUDE.md` that tells a future agent which file is canonical and why.

That single sentence is often enough. Not because the agent reads it first — it may not — but because the act of writing it forces you to think about what you actually want the agent to know. The problem, more often than not, is not that you forgot to tell the agent something. It is that the folder forgot to tell it first.

---

Several things I do not yet know well enough to be confident about. I do not know how much different tools weight timestamps versus path names when both are available, and whether that ratio shifts significantly across models or tool versions. I do not know which folder names are most reliably interpreted as "archive" or "active" across the range of models in common use — `archive/`, `old/`, `deprecated/`, and `legacy/` probably all carry roughly similar semantic loads, but I have not seen controlled comparisons. And I do not know whether future agents that can access IDE edit history will infer canonical status from human behavioral patterns — recent edits, cursor position, open tabs — better than from folder structure alone. If they can, the advice in this chapter will need revision. Until then, the folder is the best communication channel you have, and most working directories do not use it.

| Before path | After path | Signal sent to the agent |
|---|---|---|
| project/chapter-final.md | current/chapter.md | This is the live source. |
| project/chapter-old.md | archive/chapter-old.md | Historical only unless requested. |
| project/output.html | outputs/output.html | Deliverable, not source. |
| project/notes.md | research/notes.md | Evidence or background. |

---

## LLM Exercises

1. Paste a directory listing from one of your current projects into a conversation and ask the model: "Which file would you read first if asked to summarize this project's current state, and why?" Compare its reasoning to your own.

2. Describe a real confusion an agent had with your files — wrong version used, stale content cited, irrelevant material included. Ask the model to diagnose whether the root cause was a folder structure problem, a prompt problem, or a model problem. Push back on its diagnosis.

3. Ask the model to propose a folder structure for a project type you work on regularly (book chapters, code, research notes, client deliverables). Then critique its proposal: what would still confuse an agent? What signals does it fail to give?

![Agents choose better source files when project structure separates current work, generated output, and archive material.](images/01-why-your-agent-reads-the-wrong-file-fig-01.png)
*Figure 1.1 - Ambiguous Folder vs Structured Folder*

![Adding irrelevant files to context can bury the relevant file even when it is present.](images/01-why-your-agent-reads-the-wrong-file-fig-02.png)
*Figure 1.2 - Context Competition*

## Prompts

### Figure 1.1 - Ambiguous Folder vs Structured Folder

Create a standalone D3 v7 HTML figure for "Ambiguous Folder vs Structured Folder" using the pinned CDN. Use a comparison panels structure with 4-6 labeled elements inferred from this concept: Agents choose better source files when project structure separates current work, generated output, and archive material. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 1.2 - Context Competition

Create a standalone D3 v7 HTML figure for "Context Competition" using the pinned CDN. Use a conceptual map structure with 4-6 labeled elements inferred from this concept: Adding irrelevant files to context can bury the relevant file even when it is present. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
