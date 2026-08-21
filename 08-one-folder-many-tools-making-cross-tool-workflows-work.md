# Chapter 8 — One Folder, Many Tools: Making Cross-Tool Workflows Work
*Why the durable parts of your project should be readable by any tool that ever touches it, and what to do about the parts that aren't.*

Here is a situation that happens more often than it should. You build a working agent setup in Claude Code. Instructions are solid. The folder structure is clean. The agent behaves the way you want. Then a colleague wants to run the same project in Cursor. Or you want to try Codex on one piece of it. Or you open the project in Aider for a quick edit.

Each of those tools looks for instructions in a different place. Claude reads `CLAUDE.md`. Codex reads `AGENTS.md`. Cursor reads `.cursor/rules`. Aider reads `.aider.conf.yml` or a conventions file you point it at. You now have three options: maintain four separate instruction files that slowly drift apart, paste your rules into every tool individually and accept that they will immediately diverge, or think carefully about which parts of your instructions are about the project and which parts are about the tool.

The third option is the one worth understanding.

## What "Tool-Agnostic" Actually Means

Tool-agnostic does not mean tool-free. Every AI tool has its own behavior, its own configuration surface, its own way of doing things that has no equivalent elsewhere. Cursor has specific indexing behavior. Claude Code has plan mode. Aider has voice, interactive, and auto-commit modes. You are not going to pretend those differences do not exist.

What tool-agnostic means is that the durable intent — the rules that describe the project, not the rules that tune the tool — lives in one place that any tool can reach. The tool-specific configurations are adapters: thin shims that point at the canonical source and add only what is genuinely tool-specific on top.

The practical distinction is not always obvious in the moment, but it becomes clear when you ask: if I switched to a different tool tomorrow, would this rule still apply? "Never delete files; archive instead" — yes, that applies regardless of what tool you are using. "Use plan mode before broad edits" — that is Claude Code–specific behavior that has no meaning in Aider. The first kind of rule belongs in the canonical source. The second belongs in an adapter.

![Shared instruction source and tool adapters](images/08-one-folder-many-tools-making-cross-tool-workflows-work-fig-01.png)
*Figure 8.1 - Shared instruction source and tool adapters*

<!-- → [DIAGRAM: Center node labeled AGENTS.md with four arrows pointing outward to CLAUDE.md, .cursor/rules, GEMINI.md, and .aider.conf.yml. Each adapter node has a small label: "imports + Claude-specific additions," "imports + Cursor-specific additions," etc. Caption: One canonical source, multiple thin adapters. Each tool reads its own file; each adapter points back to the shared source for project truth. Tool-specific rules live only in the adapter that needs them.] -->

## The Canonical File

The best current candidate for the portable canonical file is `AGENTS.md`. OpenAI Codex documents it as the project instruction format. It is tool-name-neutral, which matters — a file called `CLAUDE.md` signals to every other tool that it was written for something else. A file called `AGENTS.md` signals that it was written for agents, plural.

What belongs in `AGENTS.md` is project truth: the rules that any reasoning tool should follow when working in this folder, regardless of which tool it is.

```markdown
# AGENTS.md

Read _MANIFEST.md first.
Never delete; archive instead.
Do not treat outputs/ as source.
Run tests before reporting done.
```

Four rules. Each one is about the project. None of them describe Claude's behavior specifically or Cursor's behavior specifically. They describe how this project works.

What does not belong in `AGENTS.md` is anything that only makes sense in one tool's context. Plan mode before broad edits — that is Claude Code's vocabulary. Auto-commit behavior — that is Aider's vocabulary. Those go in adapters.

## Building Adapters

An adapter is the minimal file each tool needs that points to `AGENTS.md` and adds only what the tool requires.

For Claude Code, `CLAUDE.md` can import `AGENTS.md` directly:

```markdown
@AGENTS.md

## Claude Code
Use plan mode before broad edits.
```

The `@AGENTS.md` line tells Claude Code to read that file as part of the instruction context. Everything below it is Claude-specific. If Claude Code is the only tool you use and you never plan to add others, this structure still has value: it separates project rules from tool-tuning, which makes both easier to reason about when one of them needs to change.

For Aider, the adapter is as thin as configuration allows:

```yaml
read: AGENTS.md
```

For Gemini or any tool that reads a markdown convention file:

```markdown
# GEMINI.md
Read AGENTS.md as the project instruction source.
```

The pattern is always the same: the adapter's only job is to route the tool to the canonical source and add the minimum tool-specific configuration on top. The moment an adapter starts duplicating project rules — copy-pasting content from `AGENTS.md` rather than importing it — the adapter becomes a liability. Now you have two sources of truth and no mechanism for keeping them in sync.

| Tool | Canonical instruction file it reads natively | Adapter filename | Import/reference syntax |
|---|---|---|---|
| Claude Code | CLAUDE.md | CLAUDE.md | @AGENTS.md |
| OpenAI Codex | AGENTS.md | No adapter needed | Native |
| Aider | .aider.conf.yml or conventions file | .aider.conf.yml | read: AGENTS.md |
| Cursor | .cursor/rules | .cursor/rules | Manual reference or import |
| Gemini | GEMINI.md | GEMINI.md | Read AGENTS.md |

## Three Levels of Cross-Tool Robustness

There is a progression here, and knowing where you are on it helps you decide how much infrastructure to build.

The first level is reference: one file tells the agent to read another. "Read `AGENTS.md` first" somewhere in `CLAUDE.md`. This works for solo projects where you are the only one setting up tools. It requires nothing beyond a line of text and the discipline to add it.

The second level is bridge: a tool file imports or actively points to the canonical file using the tool's native import mechanism. Claude Code's `@filename` syntax is a bridge. Aider's `read:` config key is a bridge. The tool actually loads `AGENTS.md` at session start rather than relying on the agent to read it when instructed. This is more robust — you are not depending on the agent to follow an instruction in order to load the instructions.

The third level is compile: generated adapters are built from shared instruction modules. Instead of hand-writing `CLAUDE.md` and `.cursor/rules` separately, you maintain a source-of-truth module and generate both adapters from it, the way a build system generates platform-specific binaries from shared source. This makes drift structurally impossible: the adapters are not maintained, they are produced.

Use reference for quick solo work. Use bridge for any project with more than one tool in regular use. Reach for compile when you have multiple adapters that are already drifting or when you are building a shared workflow that others will fork and adapt.

## Ignore Files Are Adapters Too

Every major tool has some mechanism for telling it which files to skip. And almost every tool uses a different filename for this.

| Tool | Ignore file |
|---|---|
| Aider | `.aiderignore` |
| Cursor | `.cursorignore` / `.cursorindexingignore` |
| Continue | `.continueignore` |
| Cline | `.clineignore` |
| Claude Code | `.gitignore` plus permissions and settings |

There is no cross-tool `.llmignore` standard today. Which means if your `_MANIFEST.md` says "treat `archive/` as off-limits by default," you have to express that intent separately in each tool's ignore mechanism — if the tool has one, and if the mechanism is actually enforced rather than advisory.

| Tool | Ignore file |
|---|---|
| Claude Code | Tool-specific memory and ignore behavior. |
| Codex | Respects project files and sandbox configuration. |
| Aider | .aiderignore |
| Cursor | .cursorignore / editor indexing controls |
| Gemini | Tool-specific ignore configuration |
| Cross-tool standard | None currently exists. |

Two things worth being explicit about: first, `.gitignore` controls version control, not agent context. A file in `.gitignore` is still locally readable. Second, ignore files of all kinds should be treated as best-effort context shaping, not security boundaries. If a file contains sensitive information, keeping it out of the folder is the right answer. An ignore file is not a vault.

## The Portability Matrix

It helps to be honest about which parts of a workflow travel well and which do not.

Folder architecture is fully portable. A `current/` folder means the same thing to Claude as it does to Aider as it does to a bash script. Path structure has no tool-specific vocabulary.

`_MANIFEST.md` is portable with adaptation. Any tool can read a markdown file. But whether the tool actually reads it at session start depends on whether you have configured the adapter to point at it.

`status.md` and session handoffs are fully portable. Plain text. Any tool can read them. The ritual of updating them is human; the reading is trivially transferable.

`PROJECT_RULES.md` is portable if referenced. The content is tool-neutral. Whether it is loaded depends on whether `AGENTS.md` or the tool's adapter includes a reference to it.

Hooks — things like "run this script before a session starts" or "run linting on every save" — are tool-specific. Each tool has its own hook mechanism, and they do not translate.

Compaction commands are tool-specific. Claude Code has `/compact`. Other tools have different summarization mechanisms or none at all.

MCP configuration is concept-portable, config tool-specific. The idea that tools should have scoped access to external services is portable. The JSON or YAML that configures it is not.

| Workflow element | Portability classification |
|---|---|
| Folder architecture | Fully portable |
| _MANIFEST.md | Portable with adaptation |
| status.md and session-handoff.md | Fully portable |
| PROJECT_RULES.md | Portable if referenced |
| AGENTS.md | Portable canonical source |
| Hooks | Tool-specific |
| Compaction commands | Tool-specific |
| MCP config | Concept portable, config tool-specific |

## What Drift Looks Like

The reason to care about all of this is that the alternative — four hand-maintained instruction files that each started as copies of each other — produces a specific and unpleasant failure mode.

Three months into a project, you update the rule about how to handle archived files. You update it in `CLAUDE.md`. You forget to update it in `.cursor/rules`. Now Cursor and Claude have different rules about the same question. The project's behavior becomes tool-dependent in a way that is invisible until something breaks.

![Instruction copy drift vs. adapter pattern](images/08-one-folder-many-tools-making-cross-tool-workflows-work-fig-02.png)
*Figure 8.2 - Instruction copy drift vs. adapter pattern*

<!-- → [DIAGRAM: Timeline showing a project starting with four identical instruction files at month 0. At month 1, CLAUDE.md is updated with a new rule. At month 3, Aider config is updated differently. At month 6, all four files have diverged — shown with different colored text segments. A second parallel timeline shows the adapter pattern: only AGENTS.md changes, adapters remain thin, no divergence. Caption: Hand-maintained copies drift. The adapter pattern prevents divergence by making AGENTS.md the only file that contains project rules.] -->

The adapter pattern prevents this by construction. If project rules live only in `AGENTS.md`, updating a project rule means updating one file. The adapters do not contain project rules; they contain a pointer to the file that does. There is nothing to drift.

## What Would Change My Mind

I would revise this chapter if a single cross-vendor instruction mechanism became mandatory and consistently implemented across the major tools. If every AI coding assistant agreed to read `AGENTS.md` natively, the adapter layer would become unnecessary — the canonical file would work directly in every tool. That conversation is happening; `AGENTS.md` has growing support. Today, adapters remain necessary for any project that uses more than one tool.

## Still Puzzling

How many tools will adopt `AGENTS.md` natively is genuinely unclear. The momentum is real — Codex documents it, and other tools are watching. But "growing support" is not the same as universal support, and the tools that resist adoption will require adapters indefinitely.

Whether adapters should be generated automatically is the question I find most practically important for teams. Hand-maintained adapters drift; that is a structural property, not a discipline problem. Generated adapters cannot drift by definition. The obstacle is that someone has to build and maintain the generator, which is non-trivial infrastructure for a project that is not already large enough to justify it.

How teams should test for instruction drift is a genuine open problem. The symptom — different tools behaving differently in the same project — is easy to observe after the fact and hard to catch before it matters. A linter that checks for duplicate project content across instruction files would help. I am not aware of a good one.

---

## Exercises

**Warm-up**

1. *(Recall)* Explain the difference between project truth and tool-specific configuration. Give one example of each from the chapter. *Tests whether you can apply the core distinction that drives the adapter pattern before getting into mechanics.*

2. *(Identification)* A project has the following files: `CLAUDE.md` (500 words of project rules and Claude-specific settings), `.cursor/rules` (the same 500 words copy-pasted plus Cursor-specific settings), and `.aider.conf.yml` (points to a now-deleted conventions file). Identify what is wrong with each file and what the correct state of each should be. *Tests comprehension of the adapter pattern applied to a realistic broken state.*

3. *(Recall)* Name the three levels of cross-tool robustness from the chapter (reference, bridge, compile) and describe in one sentence when each is appropriate. *Tests retention of the progression and its practical decision criteria.*

**Application**

4. *(Translation)* You have a working `CLAUDE.md` that contains the following rules: (a) read `_MANIFEST.md` first; (b) never delete files, archive instead; (c) use plan mode before broad edits; (d) write all chapter drafts to `current/`; (e) run the linter before reporting done. Create an `AGENTS.md` that contains the portable rules, and a new `CLAUDE.md` that imports `AGENTS.md` and retains only the Claude-specific rule. *Tests ability to perform the canonical/adapter separation on a realistic mixed-content instruction file.*

5. *(Design)* A project uses Claude Code for writing, Cursor for code review, and Aider for quick terminal edits. Design the complete instruction file set for this project: what goes in `AGENTS.md`, what goes in each adapter, and what format each adapter uses. *Tests ability to apply the adapter pattern across three tools simultaneously.*

6. *(Analysis)* Your `_MANIFEST.md` says the `archive/` folder should be ignored by agents unless explicitly requested. You want this to apply to Aider and Cursor. What files do you create or edit, and what are their contents? What is the limit of what this achieves? *Tests understanding of tool-specific ignore files as adapters, plus the security-vs.-context-shaping distinction.*

**Synthesis**

7. *(Integration)* Three months into a project, you discover that `CLAUDE.md` and `.cursor/rules` have diverged: Claude has been archiving files before deletion; Cursor has been deleting them directly. Describe exactly what happened, what the structural cause was, how you would fix the current state, and what you would change about the project setup to prevent recurrence. *Tests integration of the drift failure mode, the adapter pattern, and the three-level robustness framework.*

8. *(Evaluation)* The portability matrix classifies MCP configuration as "concept portable, config tool-specific." Write a paragraph explaining what this means in practice for a team that wants to use the same external tool integrations (a project management API, a web search tool) across Claude Code and a second agent tool. What can they share? What must they rewrite? *Tests nuanced application of the portability matrix to a realistic multi-tool integration scenario.*

**Challenge**

9. *(Open-ended)* The compile level — generating adapters from shared instruction modules — solves the drift problem structurally but adds infrastructure overhead. Design a minimal compile system for a project with three tools. What is the canonical source format? What does the generator do? What triggers a regeneration? Where does this system break down, and under what conditions would it cause more problems than it solves? *Engages the chapter's most forward-looking claim and tests whether you can reason about the trade-offs of a system that does not yet have a standard implementation.*

## Prompts

### Figure 8.1 - Shared instruction source and tool adapters

Create a standalone D3 v7 HTML figure for "Shared instruction source and tool adapters" using the pinned CDN. Use a systems diagram structure with 4-6 labeled elements inferred from this concept: Center node labeled AGENTS.md with four arrows pointing outward to CLAUDE.md, .cursor/rules, GEMINI.md, and .aider.conf.yml. Each adapter node has a small label: "imports + Claude-specific additions," "imports + Cursor-specific additions," etc. Caption: One canonical source, multiple thin adapters. Each tool reads its  Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 8.2 - Instruction copy drift vs. adapter pattern

Create a standalone D3 v7 HTML figure for "Instruction copy drift vs. adapter pattern" using the pinned CDN. Use a timeline / progression structure with 4-6 labeled elements inferred from this concept: Timeline showing a project starting with four identical instruction files at month 0. At month 1, CLAUDE.md is updated with a new rule. At month 3, Aider config is updated differently. At month 6, all four files have diverged — shown with different colored text segments. A second parallel timeline shows the adapter pat Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
