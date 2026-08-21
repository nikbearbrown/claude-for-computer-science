# Chapter 4 — Project Rules and Instruction Precedence

You put "never delete files" in `CLAUDE.md`. Claude Code mostly obeys. Then you open the same project in another tool and the rule disappears. The project did not change. The tool's instruction-loading model did.

That is the problem with building a workflow on one vendor's instruction file and mistaking it for architecture.

---

Instruction files are not universal. This is the thing practitioners discover the hard way, usually after a rule they were confident in silently fails when they switch tools. Claude Code reads `CLAUDE.md`. OpenAI Codex uses `AGENTS.md`. Aider can read convention files when configured. Cursor, Gemini, Copilot, and Cline each have their own rule locations, their own precedence models, and their own opinions about which files to load automatically and which require explicit configuration. `AGENTS.md` is currently the strongest cross-agent candidate — there is active effort to establish it as a shared standard — but not every tool treats it the same way even now, and the landscape shifts with each major release.

None of this is a design failure on any particular vendor's part. Instruction-loading is an active design space, and the tools are not converging as fast as practitioners are building workflows that depend on stability. The practical consequence is that a rule you wrote once, in one file, for one tool, is a local rule dressed in the clothes of a project rule. When the project outlives the tool, or when the team uses more than one tool, the rule becomes unreliable.

The answer is not to write more rules. It is to think carefully about what a rule actually is — and where rules live versus where they are enforced.

---

There is a distinction worth drawing early, because collapsing it causes most of the confusion in this area. An instruction file is context. Loading it means the model can see it. Enforcing it means something in the system actually prevents violations.

These are not the same thing. A rule that says "never permanently delete files" in a `CLAUDE.md` is a strong, clear instruction. If the model is in a cooperative mode and the task is unambiguous, it will probably follow the rule. If the task is broad, if the model has inferred a different interpretation, or if the tool processes instructions in a way that deprioritizes the rule file, the model may not follow it — not because the model is broken, but because context is probabilistic, not deterministic. The rule was loaded. It was not enforced.

This matters enormously for how you design project infrastructure. For rules that are advisory — style preferences, workflow conventions, labeling habits — instruction files are exactly the right tool. For rules that are high-risk — anything involving irreversible operations, production systems, files that trigger downstream processes — instruction files are the wrong tool. Those rules need hooks, permission profiles, approval gates, or tests. A sentence in a markdown file is not a circuit breaker.

Keeping this distinction clean prevents a specific failure mode: you write careful rules, the tool loads them, the model follows them most of the time, you develop confidence, and then one unusual task produces an exception that the rules did not catch. The confidence was real but misplaced. You were measuring how often the rule was followed in ordinary cases, not what happens when it matters.

---

Given that instruction files are context and not enforcement, the question becomes how to write them so they are as reliable as context can be. The first principle is to write precedence explicitly. Agents do not have an intuitive sense of which instruction file is authoritative when they conflict. Global settings, project files, inline user instructions, and tool defaults all compete, and different tools resolve that competition differently.

The simplest intervention is a single sentence at the top of your canonical project file:

```markdown
If these project rules conflict with broader defaults, follow this file unless the user explicitly says otherwise.
```

That sentence will not force compliance. What it does is give every agent the same local hierarchy — a clear statement that this file is authoritative for this project, and that the resolution of any conflict should favor the local over the global. Most agents, reading that sentence in context, will treat it as a meaningful instruction. The alternative — hoping the tool's default precedence model happens to match your intent — is less reliable.

The second principle is to label rules by their source, because not all rules have the same evidentiary status. A rule derived from a vendor's published specification is different from a practitioner convention that has become widespread but is not formally documented, which is different from a reasoned inference about how a tool probably behaves based on observed behavior. Treating all three as equally authoritative produces rules that look identical on the surface but have very different reliability profiles.

A labeling system that has proven useful in practice:

```text
[OFFICIAL]    vendor docs or published spec
[CONVENTION]  common practitioner pattern
[INFERRED]    reasoned recommendation
[TOOL]        tool-specific behavior
[AGNOSTIC]    designed to hold across tools
```

The purpose of these labels is not pedantry. It is to prevent a specific kind of drift: a Claude Code convention gets copied into a project file, the label disappears, and six months later it is being cited as a cross-tool law. The labels keep the epistemic status of each rule visible.

---

The structural problem underneath all of this is duplication. If you are maintaining separate instruction files for Claude Code, Cursor, Gemini, and Aider, you are maintaining four versions of the same rules, and they will diverge. Not intentionally — through ordinary project evolution. You update the rule in `CLAUDE.md` and forget to update the Cursor version. A teammate adds a convention to one file that should apply everywhere. Over time the files become inconsistent, and you have the original problem in a new form: the rule exists, but which version of it?

The portable solution is one canonical file with thin adapters for each tool. The canonical file carries the actual rules. The adapters do nothing except point at it.

For Claude Code:

```markdown
@AGENTS.md
```

For Aider:

```yaml
read: AGENTS.md
```

For Gemini:

```markdown
# GEMINI.md
Read AGENTS.md first. Treat it as the project instruction source.
```

Now the project has one source of truth and several bridges. Updating the canonical file updates every tool's effective rules. The adapters require almost no maintenance because they contain almost no content.

For larger projects, even this can break down — the adapters accumulate tool-specific sections that should be shared, the shared sections drift back into the adapters, and the canonical file loses its status. The stronger pattern at that scale is to generate the adapter files from a shared source rather than maintain them by hand.

| Pipeline element | Role | Generated output |
|---|---|---|
| instructions/source/ | Durable shared rule modules. | AGENTS.md |
| instructions/adapters/ | Tool-specific additions. | CLAUDE.md, GEMINI.md, .cursor/rules |
| Build script | Concatenates and validates modules. | Consistent adapter files |
| Audit step | Detects drift and duplication. | Diff or report |

The structure looks like this:

```text
instructions/
  manifest.yml
  _shared/
    no-delete.md
    completion-report.md
    conformance-gate.md
  project.md
```

From which you generate:

```text
AGENTS.md
CLAUDE.md
GEMINI.md
.github/copilot-instructions.md
.cursor/rules/project.mdc
```

Each generated file begins with a header:

```html
<!-- GENERATED FILE - do not edit by hand.
     Source: instructions/manifest.yml
     Rebuild: scripts/build-instructions.mjs -->
```

This makes shared rules genuinely shared while still allowing tool-specific sections in the generated outputs. The cost is a small build step and the discipline to treat generated adapter files as outputs — never edit them directly, the same way you would not edit a compiled binary.

The principle here is the same one that governs good software design more broadly: single source of truth, thin adapters, generated outputs. The fact that the outputs are instruction files rather than code does not change the logic.

---

What a well-formed canonical rules file actually contains is worth being specific about, because the most common failure is a file that is too long, too vague, or structured in a way that makes the important rules hard to find.

```markdown
# PROJECT_RULES.md

If these rules conflict with broader/global defaults, follow THIS file
unless I explicitly say otherwise in chat.

## Source of truth
- Read `_MANIFEST.md` first.
- Treat Tier 1 files as canonical.
- Do not use `archive/` unless explicitly requested.

## File safety
- Never permanently delete files. Move superseded files to `archive/`.
- Ask before renaming or overwriting canonical files.
- Show a diff before broad changes.

## Output rules
- Save drafts to `staging/`.
- Save final deliverables to `outputs/`.
- Do not treat `outputs/` or `build/` as source.

## Task behavior
- Load only the files needed for the current task.
- State assumptions before broad changes.
- Stop if two Tier 1 files conflict.
```

Notice what this file does not contain: explanations of why each rule exists, long discussions of edge cases, version histories, or aspirational goals. It contains rules. Each rule is a sentence. The file loads quickly, sits near the top of whatever context the agent builds, and remains legible when skimmed.

The length question is a real one. There is a point past which a rules file becomes fixed context waste — material that loads on every task, occupies window space, and does not improve behavior because the relevant rules are buried in surrounding material. The practical ceiling is lower than most people expect. A file that takes three minutes to read carefully is probably too long for an agent to weight properly in a short task. The test is whether you could read the file aloud in under ninety seconds and have someone else summarize its three most important rules from memory. If not, it is probably too long.

---

Three misconceptions come up consistently in this area.

The first is that loading equals enforcement. It does not, and treating it as if it does is how high-risk operations get past instruction files. The rules file is a communication channel, not a gate. For anything irreversible, design an actual gate.

The second is that every tool needs a full rules file. It does not. Every tool needs an adapter. One tool — probably whichever is most actively used in the project — carries the canonical file. The others point at it.

The third is that precedence is obvious. It is not. In a project with global tool settings, a project-level rules file, inline user instructions, and a system prompt, the resolution order is different across tools and often undocumented at the edge cases. Writing precedence explicitly is not defensive paranoia — it is the minimum responsible specification for a system where multiple instruction sources can conflict.

---

Several things remain genuinely uncertain. Whether `AGENTS.md` becomes universal or remains one standard among several depends on adoption decisions by tool vendors that are not yet made. Whether project rules should be generated from structured YAML — which would allow richer tooling, validation, and diff tracking — is an open design question; the overhead may be worth it at scale and not worth it for small projects. And the question of how much rule text is too much does not have a principled answer yet, only practitioner intuitions that vary widely. These are not edge cases — they are the live questions in the space right now, and the answers will change the right approach to everything else in this chapter.

---

## LLM Exercises

1. Paste the contents of every instruction file in one of your projects into a conversation. Ask the model to identify which rules conflict, which are duplicated, and which are tool-specific versus project-generic. Use the output to draft a canonical file and a set of adapters.

2. Take a rule from your current `CLAUDE.md` or equivalent and label it with the confidence system above: `[OFFICIAL]`, `[CONVENTION]`, `[INFERRED]`, `[TOOL]`, or `[AGNOSTIC]`. Ask the model to help you find the primary source — vendor documentation, practitioner write-up, or inference from observed behavior. Notice how the label changes when you trace the source.

3. Write the precedence sentence for your project — the one that tells agents how to resolve conflicts between this file and global defaults — and then ask the model to try to construct a scenario where that sentence would still fail. Design a response to the failure it proposes.

![CLI-agnostic projects separate durable project rules from tool-specific adapter files and per-task instructions.](images/04-project-rules-and-instruction-precedence-fig-01.png)
*Figure 4.1 - Instruction Layers for CLI-Agnostic Work*

![When instructions conflict, the agent should identify the conflict, prefer the project rule, and ask before overriding it.](images/04-project-rules-and-instruction-precedence-fig-02.png)
*Figure 4.2 - Rule Conflict Resolution Path*

## Prompts

### Figure 4.1 - Instruction Layers for CLI-Agnostic Work

Create a standalone D3 v7 HTML figure for "Instruction Layers for CLI-Agnostic Work" using the pinned CDN. Use a process flowchart structure with 4-6 labeled elements inferred from this concept: CLI-agnostic projects separate durable project rules from tool-specific adapter files and per-task instructions. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 4.2 - Rule Conflict Resolution Path

Create a standalone D3 v7 HTML figure for "Rule Conflict Resolution Path" using the pinned CDN. Use a statistical / quantitative structure with 4-6 labeled elements inferred from this concept: When instructions conflict, the agent should identify the conflict, prefer the project rule, and ask before overriding it. Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
