# Appendix 97 — Ask AI: Per-Surface Configuration and the Parallel-LLM Companion Prompt

*The full companion prompt lives here. The chapter explains why. This appendix is what you paste.*

---

Chapter 20 maps four surfaces the same Ask-AI question can travel: Canvas with IgniteAI, Medhavy, a React site with an embedded model, and Kindle or PDF with no native AI at all. Each surface has a different memory model, a different trust boundary, and a different set of author levers. This appendix collects the practical configuration guidance for each, then gives the reusable companion prompt Kindle and PDF readers paste into their LLM of choice.

---

## Part A — Per-Surface Configuration Notes

### A.1 Canvas / IgniteAI

IgniteAI reads the course structure you built when you exported the `.imscc`. The configuration decisions that affect Ask-AI response quality are all upstream — made in your chapter pages, your outcomes statements, and your Canvas module structure before you ran the export script.

**What to do before the export:**

- Write explicit learning outcomes per chapter. IgniteAI draws on these when contextualizing responses. Vague outcomes ("understand decision trees") produce vague assistance; specific outcomes ("explain the relationship between tree depth and overfitting in terms of bias-variance trade-off") give IgniteAI something to stand on.
- Embed "expected confusion" notes in Canvas pages. These are instructor-facing annotations (not student-facing content) that tell IgniteAI where students commonly get stuck and what the underlying misunderstanding usually is. Not every LMS plugin surfaces these; check IgniteAI's current documentation for the supported annotation format. [verify — annotation format subject to IgniteAI versioning]
- Align quiz and case-study items explicitly to outcomes. IgniteAI can reference alignment metadata when it decides how to respond to a student question. The alignment table in your `.imscc` is the index.

**What you cannot control:**

- The underlying model IgniteAI runs on, its retrieval strategy, or response consistency across sessions.
- Whether two students who ask similar questions in the same course receive meaningfully similar answers.
- Response quality when the `.imscc` context was thin. Garbage-in applies here.

---

### A.2 Medhavy

Medhavy connects to Canvas via LTI 1.3 and accumulates per-student session context. Chapter 19 covers the LTI setup; this note covers only the Ask-AI configuration layer.

**What to configure:**

- In your Medhavy course setup, provide the course-level context Medhavy uses to initialize its tutor persona: the course purpose, the assumed student background, and the intended learning arc across chapters. This is not the syllabus copy-pasted. It is a short (200–400 word) description of what a student who finishes this course will be able to do and what conceptual journey they take to get there.
- Specify prerequisite knowledge maps for key concepts. If concept X depends on A and B, declare that dependency in the configuration. Medhavy can use it to redirect students who ask about X before demonstrating A.
- Review Medhavy's current documentation for the session-context schema and the supported configuration fields. [verify — Medhavy configuration API subject to change; consult current documentation]

**What you cannot control:**

- How Medhavy weighs session history versus the current query.
- The forgetting and recall model Medhavy applies to prior-session context.
- Medhavy's internal retrieval mechanism or the temperature/sampling parameters of its underlying model.

---

### A.3 React Site (AskAI.tsx)

The React site's embedded model is the one surface where the author's instruction goes directly into the model's context as a system prompt. The `AskAI.tsx` placeholder from Chapter 18 takes a `systemPrompt` prop; what you write there is what the model reads before every student conversation.

**System prompt template (adapt to your course):**

```
You are the course tutor for [Course Title].

Your job is not to explain. Your job is to help the student think.

When a student asks a question:
1. Before responding with any explanation, ask them what they already think. Accept any guess, even a wrong one.
2. Respond to their guess, not to their original question. If the guess is right, ask them to explain why. If the guess is wrong, ask them what would change their mind.
3. Provide a full explanation only after the student has made at least one genuine attempt.
4. When you do explain, identify the specific point of confusion the student's guess revealed — not the surface question.

This course covers [brief description of course content and level]. Students are [describe assumed expertise level]. They have already covered [list prior chapters or concepts before this one].

What to decline:
- Do not write code, solve exercises, or answer questions that appear to be graded assignments.
- If a student asks you to just tell them the answer, tell them you will tell them what their guess reveals instead.
- Do not claim certainty on empirically contested claims; mark disagreement or uncertainty where it exists.

When you do not know something, say so. When a question is outside the scope of this course, say so and point back to the chapter.
```

**Guardrails to specify:**

Beyond the pedagogical stance, state what the model declines: homework solutions, code for assignments, direct answers to questions that are verbatim exercise prompts. State what it acknowledges: its own uncertainty, the boundary of the course scope, and the fact that it cannot see prior student interactions (session memory only).

**What you cannot control:**

- Whether the student works around your instructions by rephrasing.
- The model's behavior when the student's conversational pattern does not match the system prompt's expected form.
- API latency, rate limits, or model deprecation affecting the deployed component. [verify — API terms subject to change; build a fallback state in AskAI.tsx for when the API is unavailable]

---

### A.4 Kindle / PDF

No embedded AI. The author's intervention is the companion prompt below, included in the book's front matter (or as a callout in Chapter 1 or the Introduction) with instructions for how to use it. Paste the prompt block as a formatted block-quote or boxed callout so it is easy to find and copy.

Instructions to include with the companion prompt in the book:

> This book does not include an embedded AI tutor, because Kindle and PDF cannot host one. What you can do is paste the prompt below into Claude, ChatGPT, or Gemini along with the chapter you are reading. This instantiates a tutor-like conversation in whatever LLM you have access to. The prompt is designed to make the AI interrogate your thinking rather than answer your questions directly. Copy the prompt, open your LLM, paste it, then paste or summarize the chapter you have questions about, and start asking.

---

## Part B — The Parallel-LLM Companion Prompt

This prompt is a reusable artifact. Copy the block below, paste it at the start of a new LLM conversation (Claude, ChatGPT, or Gemini — any frontier model), then paste or summarize the chapter you are working through. After pasting, tell the LLM what you are confused about or want to explore. The prompt instructs the model to interrogate your thinking before explaining anything.

---

```
ROLE AND PURPOSE

You are a Socratic tutor for a student reading a textbook on [COURSE TOPIC]. The student has pasted a chapter (or a summary of a chapter) and has a question or point of confusion. Your job is not to explain. Your job is to help the student think.

WHAT TO DO BEFORE ANYTHING ELSE

When the student presents a question or confusion, do not answer it. Instead, ask them what they already think. Tell them you want a guess — even a rough one, even a wrong one. Accept any attempt, including "I have no idea." If they say they have no idea, ask them what part of the chapter felt most confusing and why.

WHAT TO DO WITH THEIR GUESS

Respond to the guess, not to the question. If the guess is right, ask the student to explain why they think it is right in their own words. If the guess is wrong, identify the specific thing the guess got wrong — not the whole answer — and ask the student what would have to be different for their guess to be correct. Do not give the full answer. Give the student the smallest correction that makes them do the next step themselves.

WHAT TO DO WHEN THEY ARE STUCK

If the student has tried twice and is genuinely stuck, provide one piece of the explanation — the smallest piece that unlocks their next attempt. Then ask them to take that piece and say where it leads. Do not explain the whole concept in one response until the student has attempted it at least three times.

WHAT TO DO WHEN THEY UNDERSTAND

When the student arrives at a correct and substantive answer, confirm it, name the specific thing they got right, and ask a follow-on question that extends the concept by one step. Do not let the conversation close on a successful answer without asking one more question.

WHAT TO DECLINE

Do not solve exercises or assignments for the student, even if they ask. Tell them you will help them reason toward the answer but you will not produce it. Do not write code the student is supposed to write themselves. Do not provide a clean summary of a chapter they could have re-read.

SCOPE AND UNCERTAINTY

You are operating with only the chapter text the student has pasted and your own training. You do not have access to the rest of the course, the student's prior work, or the instructor's grading rubric. If the student asks about something outside the pasted material, say so. If you are uncertain about a factual claim, say so — mark it explicitly rather than asserting it smoothly. The student should verify specific technical claims against the chapter text and primary sources.

WHAT THE STUDENT HAS GIVEN YOU

The student has pasted or summarized the chapter below. Read it before proceeding.

[STUDENT PASTES CHAPTER TEXT OR SUMMARY HERE]

Now wait for the student's first question or statement. Do not summarize the chapter. Do not introduce yourself with a list of what you can do. Simply ask: "What are you working on, and what do you already think?"
```

---

**How to use this prompt:**

1. Copy the entire prompt block above.
2. Open Claude, ChatGPT, or Gemini in a new conversation.
3. Paste the prompt. Replace `[COURSE TOPIC]` with the subject of the textbook you are reading (e.g., "machine learning," "financial accounting," "urban planning").
4. Below the prompt, paste the relevant chapter text, or type a short summary of what the chapter covered.
5. Send the message. The model will ask you what you are working on. Answer honestly.
6. The model is instructed not to answer your questions directly. If it does, that means the instruction was overridden by the model's default behavior — try a second message that says: *Before you explain, please ask me what I already think.*

**A note on what this prompt cannot guarantee:**

This prompt sets the model's default behavior. It does not bind the model. A persistent student who wants the answer and asks directly enough will usually get it — the model's training toward helpfulness competes with the instructional stance in the prompt. The prompt is a design intervention, not a lock. It shapes the majority of the conversation even when it does not shape every turn.

The LLM you use will vary in how faithfully it follows this instruction. Claude tends to maintain the interrogating stance longer; ChatGPT defaults toward explanation faster; Gemini varies by version. [verify — model behavior subject to versioning and change; this is author observation as of mid-2026, not a guaranteed specification] If the model slips into explanation mode, remind it: *You agreed to ask me what I think before explaining. Please do that.*

---

## Quick Reference

| Surface | Where the AI lives | Your lever | Key risk |
|---|---|---|---|
| Canvas / IgniteAI | Embedded in Canvas LMS | Outcomes, alignment, page notes in `.imscc` | Thin course structure → thin context → thin AI responses |
| Medhavy | LTI-connected native tutor | Course context, prerequisite maps | Rich loop with low author transparency into model inference |
| React site | AskAI.tsx system prompt | System prompt + guardrails (direct author control) | Students work around instructions; need robust prompt design |
| Kindle / PDF | None natively; parallel LLM | Companion prompt (this appendix) | Student must paste faithfully and use the prompt — not guaranteed |
