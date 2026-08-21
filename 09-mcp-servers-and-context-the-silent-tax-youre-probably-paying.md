# Chapter 9 — MCP Servers and Context: The Silent Tax You're Probably Paying
*Every tool you connect is a toll you pay before the work begins.*

You added the GitHub MCP server because you needed it. Then Slack, because that made sense too. Then Drive. Then a database connector. Then web search. Each one solved a real problem on the day you added it. None of them felt like a cost.

Now simple sessions feel slow. The agent hedges more than it used to. You ask for a quick edit and it seems to consider twelve possible actions before picking one. Nothing is broken. But something has changed.

What changed is the tool surface — the total area of capability the agent has to consider before it does anything at all. You did not add one tool. You added a tool surface. And that surface has a tax.

## What the Tax Actually Is

The Model Context Protocol is an open standard for connecting AI hosts to external tools and data sources. The core idea is straightforwardly useful: give the agent access to your file system, your calendar, your codebase, your database, and it can work with real material instead of asking you to paste things in. The protocol handles discovery, invocation, and response — the agent calls a tool, the tool returns data, the agent uses it.

The tax shows up in two places most people do not think to look.

The first is schema cost. Before a session begins, the client typically loads the schemas for every active MCP server — the descriptions of what each tool does, what arguments it accepts, what it returns. These descriptions exist so the agent knows which tool to reach for. But they occupy context whether the agent uses the tool or not. A session with eight connected MCP servers starts with eight schemas already loaded, before a single message has been exchanged.

The second is payload cost. When a tool is called and returns data, that data enters the context window. A database query that returns a full table of raw records can consume thousands of tokens in one call. The agent now has to process that table — to read it, extract what it needs, and continue. The rest of the table sits in context for the remainder of the session, weighting every subsequent response.

![MCP schema cost before task start](images/09-mcp-servers-and-context-the-silent-tax-youre-probably-paying-fig-01.png)
*Figure 9.1 - MCP schema cost before task start*

<!-- → [DIAGRAM: Two-panel comparison — left: session with five active MCP servers showing schema cost before first message; right: same session with two task-scoped servers; annotate token consumption at session start; caption: "Schema cost is paid before the first message. The agent is already working with a crowded window."] -->

Anthropic's guidance on code execution with MCP names both problems directly: tool schemas and intermediate payloads can consume substantial context, and moving intermediate processing into code execution — rather than returning raw results — can significantly reduce what the model must hold. The practical implication is not to use fewer tools. It is to be deliberate about which tools are active when, and what those tools return.

## The Minimal Surface Principle

The rule is simple enough to state in two lines:

```text
Load the smallest MCP surface that can do the task.
Return the smallest payload that preserves evidence.
```

The first line is about configuration. Most MCP clients allow per-project or per-task configuration — a project working in a Python codebase does not need the Slack connector or the Drive connector active by default. The servers that are always needed belong in a global config. The servers that are project-specific belong in a project config. The servers that were added for a one-off task should be disabled when that task is done.

The second line is about what the tool returns. This is where the biggest gains are, and where most implementations leave the most on the table.

## What a Tool Should Return

Here is the kind of response a naive database tool might produce:

```json
[
  {"id": "...", "created_at": "...", "updated_at": "...", "metadata": {...}, "amount": 42},
  {"id": "...", "created_at": "...", "updated_at": "...", "metadata": {...}, "amount": 107},
  ...
]
```

This is a raw dump. Every record, every field, including fields the agent will never use. If the query returned 1,842 records, that is 1,842 objects in the context window.

Here is what the same tool should return if the task is summarizing monthly spend:

```json
{
  "query": "spend_by_month",
  "date_range": "2026-01 through 2026-06",
  "records_count": 1842,
  "monthly_totals": [
    {"month": "2026-01", "total": 1204.55},
    {"month": "2026-02", "total": 983.20},
    {"month": "2026-03", "total": 1411.80},
    {"month": "2026-04", "total": 1098.45},
    {"month": "2026-05", "total": 1356.70},
    {"month": "2026-06", "total": 872.30}
  ],
  "source": "transactions table"
}
```

Six numbers instead of 1,842 objects. The provenance is preserved — you know what was queried, over what range, against how many records, from which table. The agent can reason about the result without seeing the raw data. And if you need to verify a specific month, the tool can be called again with a narrower query.

| Property | Raw dump | Aggregated response |
|---|---|---|
| Token count estimate | High and unpredictable. | Lower and bounded. |
| Provenance preserved | Often buried in payload. | Preserved as selected citations or IDs. |
| Verifiable | Hard to inspect quickly. | Easier to re-query by source. |
| Useful to agent | Mixed signal and noise. | Task-shaped summary. |
| Re-queryable | Requires parsing the dump. | Keeps the query path explicit. |

*Aggregation shifts work into the tool without discarding provenance.*

This is the design principle behind the second line: the tool should do work, not just fetch data. Aggregation, filtering, and summarization belong in the tool layer, not in the context window. The agent's job is to reason about results, not to process raw database output.

## Auditing Your Tool Configuration

The fastest way to find the tax you are paying is to inventory what you have and categorize it honestly:

```text
[ ] List every active MCP server.
[ ] Mark each: always-needed, project-needed, or one-off.
[ ] Disable one-off servers by default.
[ ] Prefer project-specific config over global config.
[ ] Filter fields before returning data.
[ ] Aggregate in the tool when possible.
[ ] Log which connector supplied which claim.
```

The last item is underrated. When an agent makes a claim based on tool output, you want to know which tool produced the evidence. A response that says "according to the transactions table, monthly spend peaked in March" is auditable. A response that synthesizes three tool outputs without attribution is not. Logging provenance at the tool layer costs almost nothing and makes verification possible.

![MCP server scope audit](images/09-mcp-servers-and-context-the-silent-tax-youre-probably-paying-fig-02.png)
*Figure 9.2 - MCP server scope audit*

<!-- → [INFOGRAPHIC: MCP audit checklist as a visual flow — each item shown as a decision gate: "Is this server always needed?" → yes: keep in global config, no: move to project config or disable; caption: "The audit is not about using fewer tools — it is about using the right tools at the right scope."] -->

The goal of the audit is not minimalism for its own sake. Some tasks genuinely need a wide tool surface. The goal is to make the surface match the task, session by session, rather than accumulating every connector you have ever added and leaving it active forever.

## Three Misconceptions Worth Naming

**"More MCP servers means a better agent."** More servers can mean more ambiguity. When the agent has twelve tools that could plausibly address a request, it has to decide which one is right before it can act. That decision consumes reasoning that could have gone toward the actual task. A focused tool surface is not a limitation — it is a reduction in overhead.

**"Only tool results cost context."** Tool schemas and descriptions load before the first message. In a client that exposes large schemas by default, a session with five connected servers may start with a context window that is already meaningfully occupied before any work begins.

**"Raw data is more honest."** Raw data is sometimes necessary — when the task genuinely requires record-level inspection, when you need to verify an edge case, when the aggregate would hide the signal. But for most tasks, a computed aggregate plus provenance is more useful than a raw dump and far less expensive. Returning everything is not rigorousness. It is delegation of processing to the wrong layer.

## What Would Change My Mind

I would revise this chapter if MCP clients broadly adopted dynamic tool retrieval — loading schemas only when a tool is actually needed, rather than at session start — and stopped exposing large unused schemas by default. Current practice still requires active pruning. When the protocol and its clients make just-in-time tool loading the default behavior rather than an advanced configuration option, the schema cost problem largely disappears. We are not there yet.

## Still Puzzling

- Which clients expose tool schema cost clearly?
- Can MCP servers declare lightweight and heavyweight modes?
- What is the right default for personal productivity connectors?

---

## LLM Exercises

> These exercises are designed to be completed with a live agent session and real tool configurations. The point is to observe actual behavior, not to reason about it abstractly.

**Exercise 1: Measure the surface.** List every MCP server currently active in your primary agent environment. For each one, record: when you added it, how often you have used it in the last two weeks, and whether it is active globally or scoped to a specific project. Classify each as always-needed, project-needed, or one-off. How many belong in each category?

**Exercise 2: Run the minimal surface test.** Disable all but one MCP server and run a task you would normally perform with the full configuration. What, if anything, is harder? What is the same or easier? Re-enable servers one at a time until the task works as expected. Note which servers were actually needed.

**Exercise 3: Rewrite a raw payload.** Take the raw output from a tool you use regularly — a database query, a file listing, a search result — and rewrite it as a compact, evidence-preserving response following the format in this chapter. Include query description, record count, the computed result, and source. What did you have to decide about what to preserve and what to summarize?

---

## Exercises

**Warm-up**

1. *(Basic recall)* What are the two places where MCP servers add context cost, according to this chapter? Describe each in one sentence and explain when each cost is paid. *What this tests: distinguishing schema cost from payload cost.*

2. *(Vocabulary)* What does "tool surface" mean? Give an example of a tool surface that is well-matched to a task and one that is over-provisioned for the same task. *What this tests: understanding the concept beyond its definition.*

3. *(Identification)* Look at the two tool response examples in this chapter. Identify three specific differences between them. For each difference, name what it costs or preserves. *What this tests: close reading of the design principle in action.*

**Application**

4. *(Audit)* Run the MCP audit checklist on a real or hypothetical tool configuration with five to eight servers. For each item on the checklist, state whether it passes or fails and what you would change. Write one sentence explaining the most significant issue you found. *What this tests: applying the audit to a concrete configuration.*

5. *(Rewrite)* A weather MCP server returns the following for a query about tomorrow's forecast: full JSON including hourly temperature, humidity, wind speed at ten altitudes, UV index, air quality index, pollen count, and satellite imagery URLs. The task is to decide whether to hold an outdoor event. Rewrite the response as a compact, evidence-preserving payload. *What this tests: applying the minimal payload principle to a realistic scenario.*

6. *(Configuration)* You are setting up a project that involves editing Python files, checking a Postgres database for usage statistics, and occasionally searching the web for documentation. Design an MCP configuration that follows the minimal surface principle. Which servers go in global config? Which go in project config? Which should be disabled by default? *What this tests: translating the principle into a concrete configuration decision.*

**Synthesis**

7. *(Integration)* Chapter 3 introduced the manifest's Tier 3 — ignore unless requested. How does the minimal surface principle for MCP servers apply the same logic to tools rather than files? Where do the two ideas reinforce each other, and where do they diverge? *What this tests: connecting the tool-scoping principle to the broader context management framework.*

8. *(Design)* The chapter mentions that aggregation and filtering belong in the tool layer, not the context window. But some MCP servers are third-party tools you cannot modify. Propose a strategy for reducing payload cost from a tool you cannot change — without modifying the tool itself. *What this tests: applying the principle under a real constraint.*

**Challenge**

9. *(Open-ended)* The "Still Puzzling" section asks whether MCP servers could declare lightweight and heavyweight modes. Design a lightweight/heavyweight mode convention for a database MCP server of your choice. What would the lightweight mode return by default? What would trigger the heavyweight mode? How would the agent know which mode to use? What failure modes does your design introduce? *What this tests: moving from a puzzling question to a concrete proposal, with honest accounting of tradeoffs.*

## Prompts

### Figure 9.1 - MCP schema cost before task start

Create a standalone D3 v7 HTML figure for "MCP schema cost before task start" using the pinned CDN. Use a comparison panels structure with 4-6 labeled elements inferred from this concept: Two-panel comparison — left: session with five active MCP servers showing schema cost before first message; right: same session with two task-scoped servers; annotate token consumption at session start; caption: "Schema cost is paid before the first message. The agent is already working with a crowded window." Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.

### Figure 9.2 - MCP server scope audit

Create a standalone D3 v7 HTML figure for "MCP server scope audit" using the pinned CDN. Use a process flowchart structure with 4-6 labeled elements inferred from this concept: MCP audit checklist as a visual flow — each item shown as a decision gate: "Is this server always needed?" → yes: keep in global config, no: move to project config or disable; caption: "The audit is not about using fewer tools — it is about using the right tools at the right scope." Use CSS custom properties for the Bear Brown palette, an accessible SVG with title and desc, responsive ResizeObserver redraw, and reduced-motion support. If quantitative marks are used, start the y-axis at zero, use one primary red series plus neutral grays, and add direct labels. Deliver one self-contained HTML file with inline CSS and JavaScript.
