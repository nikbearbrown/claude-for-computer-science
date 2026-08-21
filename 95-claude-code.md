# Appendix: Claude Code for Java
*The tool that can change your project can also change it wrong. The question is whether you will notice.*

There is a specific kind of mistake that fluent AI output makes possible. The code compiles. It runs in the demo. It looks like the code you would have written if you understood the problem. But you did not write it — you accepted it — and you do not understand it in the way that matters: you cannot predict what it does in a case you have not tested, you cannot explain why it is designed this way rather than another way, and you cannot defend it when someone asks.

That mistake is not unique to AI. Students have always been able to copy code they do not understand. What AI changes is the fluency of the output and the invisibility of the gap. Copied code from Stack Overflow at least forces you to adapt it. AI-generated code arrives shaped exactly for your problem, in your naming conventions, with comments that sound like your reasoning. The gap between "I understand this" and "I accepted this" has never been harder to see from the outside — or from the inside.

Claude Code is powerful in this course because it can work inside your project. It can read your files, trace your object structure, find inconsistencies across modules, suggest test cases, and generate scaffolding. Those capabilities are genuinely useful. They are also exactly why this appendix exists: a tool that can change your project is a tool that requires you to understand your project well enough to evaluate what comes back.

The rule is simple. Claude Code may help only after you have enough understanding to verify what comes back. Not after you have finished — after you have enough. That threshold moves across the semester. This appendix describes where it is at each stage.

---

## What Claude Code Is For, and What It Is Not

Think of Claude Code in this course as having four legitimate roles: tutor, reviewer, scaffold builder, and second reader. It is not the engineer of record. The engineer of record is you.

As a tutor, Claude Code explains things. It can explain a Java error message if you paste the exact text. It can explain a piece of code line by line. It can quiz you on a class you already wrote, checking whether you understand fields, methods, and object instances. Explanation is safe because the output is text you evaluate, not code you run.

As a reviewer, Claude Code reads what you have already built and looks for problems. It can identify responsibility leaks in a class — places where one object is doing work that belongs to another. It can find inconsistencies between your FXML `fx:id` values and your controller field names. It can suggest places where your authentication logic might not match your stated threat model. Reviewing is useful because the judgment is still yours: Claude identifies candidates, you decide whether they are real problems.

As a scaffold builder, Claude Code generates structural code from specifications you supply. Given a precise ordering rule, it writes a `Comparator`. Given an FXML file, it generates a controller stub with the right `@FXML` fields and empty handler signatures. Given a written class specification, it produces a class skeleton with empty method bodies. Scaffolding is appropriate when the design is yours and you need the mechanical translation into Java syntax.

As a second reader, Claude Code checks your work against itself. Before the final project defense, you can ask it to read your entire codebase and identify inconsistencies — places where a class name in one module does not match the same class referenced in another, or where a field declared private is accessed in a way that assumes it is public. This is the kind of consistency check that is tedious for humans and straightforward for a tool with access to all your files simultaneously.

What Claude Code is not: the tool that decides what your application does, what your objects model, how your data is stored, or what your security properties are. Those are engineering decisions. They require understanding your domain, your requirements, and the tradeoffs between possible designs. Claude Code does not have that understanding. You do, or you are in the process of building it.

---

## The Three Things You Write Before Asking

There is a concrete discipline that separates productive Claude Code use from delegation. Before you ask Claude Code for help on any implementation task, write three short statements:

**Requirement:** What should the program do? Not "implement checkout" — that is a task. "A patron can check out an available book, the book becomes unavailable, and the patron's borrowed list is updated" — that is a requirement.

**Artifact:** Where should that behavior live? Name the class, method, handler, field, test, or FXML element. If you cannot name it, you do not yet know where the code goes. Claude Code cannot know either.

**Evidence:** What would prove the answer is acceptable? "It runs" is not evidence. "After checkout, `book.isAvailable()` returns false, `patron.getBorrowedBooks()` contains the book, and a second checkout attempt returns a failure message" — that is evidence.

When these three statements exist, Claude Code has something to work against. It can evaluate whether its output satisfies the requirement, whether the artifact is the right place for the behavior, and whether the evidence conditions can be met. Without them, it guesses. The guess will be fluent. That is the danger.

| item | question to answer | example answer | what happens if missing |
| --- | --- | --- | --- |
| requirement, artifact, evidence, current module phase gate, forbidden AI action, verification method. | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |

Here is what a complete pre-ask statement looks like for the library domain:

```text
Requirement: A patron can check out an available book.
Artifact: CheckoutTransaction.process() changes Book.available
          and Patron.borrowedBooks.
Evidence: After checkout, book.isAvailable() returns false,
          patron.getBorrowedBooks() contains the book,
          and a second checkout attempt fails with a clear message.
```

That is three sentences. Writing them takes three minutes. What they protect is the difference between code you understand and code you accepted.

---

## The Phase Gates

The course relaxes AI assistance as your verification skill grows. What Claude Code may do in Module 0 is much narrower than what it may do in Module 12, because in Module 12 you have built enough understanding to evaluate more complex outputs.

The gates are not arbitrary restrictions. They are calibrated to what you can verify at each stage. A gate that prevents Claude Code from generating authentication logic before Module 6 is not bureaucracy — it is the recognition that you cannot evaluate authentication logic before you understand threat modeling, hashing, and the separation between `Person` and `UserAccount`. If the gate were not there, Claude Code would generate a plausible login system, you would accept it, and you would have credentials stored in a way you cannot defend.

![The phase-gate ladder shows four widening permission bars for Claude Code use, all anchored to one constant verification threshold.](images/95-claude-code-fig-01.png)
*Figure 95.1 — Phase-gate ladder*


In the early modules (0–2), Claude Code is diagnostic only. It explains errors you paste, quizzes you on code you already wrote, and clarifies terminology. It does not write submitted code. The first object you build in this course must be built by you, because building it is how you learn what building it involves.

In the middle modules (3–7), Claude Code becomes a scaffold and coach. It can generate layout skeletons after you provide the screen flow. It can generate class stubs after you write the specification. It can review an inheritance hierarchy after you write the is-a statements. In every case, the design work precedes the generation. Claude Code implements what you have already decided.

In the later modules (8–12), Claude Code assists with implementation. It can generate persistence boilerplate after you have answered the data-loss questions. It can generate comparators from precise ordering rules. It can generate FXML controller stubs from the FXML you designed. The scope of what it generates expands, but the principle does not change: your specification precedes its output.

In the final modules (13–14), Claude Code becomes a synthesis assistant. It suggests test cases from behavior descriptions you provide. It reviews the project for inconsistencies. You write the final project's `CLAUDE.md` from scratch, specifying what Claude may and may not do for your specific application. That document is assessed. It must be specific. A `CLAUDE.md` that says "help me with Java" is not a design document; it is evidence that the process was not visible.

---

## The CLAUDE.md File

The project `CLAUDE.md` is an instruction file. It tells Claude Code how to behave in this project. A weak file produces weak assistance: Claude makes plausible guesses about what you need, generates code shaped by generic Java conventions rather than your specific requirements, and marks no design decisions because it was not asked to.

A useful `CLAUDE.md` states what Claude may do, what it may not do, and what evidence you must produce before accepting generated code. For the course project, start with this:

```text
You are assisting a graduate Java student in INFO 5100.

The goal is not to finish the code as quickly as possible.
The goal is to help the student understand, build, debug, test,
and defend their own object-oriented Java application.

Rules:
- Explain before generating.
- Ask for missing requirements instead of guessing.
- Mark every design decision you make.
- Do not silently choose architecture, data structures, security behavior,
  persistence behavior, GUI workflow, or test meaning.
- If you generate code, include what I must verify before accepting it.
- Never let plausible Java substitute for verified Java.
```

Then add the current module's phase gate. The phase gate tells Claude what the module requires you to produce before asking for help — the screen flow table before navigation scaffolding, the threat model before authentication code, the is-a statements before inheritance review.

The `CLAUDE.md` serves two purposes. The first is instrumental: it shapes Claude's behavior in the project. The second is epistemic: writing it forces you to articulate what you own and what you are asking for help with. A student who cannot write a `CLAUDE.md` specific to their application does not yet understand their application well enough to defend it.

---

## Module-Level Boundaries

Each module's boundary follows the same logic: Claude may help with the mechanical part of the module's skill after you have done the design part. The design part is what you own. The mechanical part is where AI assistance is appropriate.

**Module 3 (User Interaction).** Claude may generate CardLayout or FXML navigation scaffolding only after you provide the screen flow table: screen, user action, object involved, state before, state after, next screen. Without the table, Claude generates a plausible navigation structure for a plausible application. With the table, Claude implements your flow.

**Module 4 (Debugging).** This is the strictest gate in the course, for a reason. The skill this module teaches is forming hypotheses about failure and testing them. AI assistance before you have formed a hypothesis teaches you nothing and robs you of the practice. You must provide symptom, suspected cause, isolation evidence, and the next thing you plan to check. Only then may Claude assess whether the hypothesis is plausible and suggest what to inspect. Claude does not identify the bug. You do.

**Module 6 (Authentication).** Claude must not generate authentication logic before you state the threat model. The threat model question is: if someone reads my user data file, what can they do? Write that answer before asking for code. Claude must not generate plaintext password storage under any framing. If it does, reject it. If you are not sure whether a storage design is plaintext or hashed, that uncertainty is the signal to ask for an explanation, not to accept the output.

**Module 8 (Persistence).** Before Claude generates file I/O code, you answer four questions: What happens if the file does not exist? What happens if a read fails? What happens if a write fails halfway through? How much data loss is acceptable? You own the data-loss policy. Claude writes the boilerplate that implements it.

**Module 12 (Scene Builder).** Claude may generate controller stubs from FXML you provide. It may not invent FXML for an unspecified workflow. The prompt structure is: here is my FXML, generate `@FXML` fields and empty handler stubs, note every `fx:id` connection, do not implement handler logic. After receiving the stub, verify it against your mapping table before adding implementation.

**Module 14 (Final Project).** You write the `CLAUDE.md` for your final project. It is specific to your application, your domain objects, your workflows, and your data-loss and security policies. It names what Claude may do, what Claude may not do, which design decisions you own, and how you will verify every AI-assisted component. A generic template is not acceptable. Generic templates describe generic applications, and your semester project is not generic.

---

## The Three-Question Audit

Before accepting any AI-assisted component — code, test, design suggestion, or documentation — answer three questions:

First: can you explain what this component does without using Claude's explanation? Not "Claude said it checks the hash" but "this method hashes the supplied password with SHA-256 and compares the result to the stored hash using `.equals()`." If your explanation requires borrowing Claude's words, you have accepted but not understood.

Second: can you explain why it is designed this way, including the alternative you rejected? Not "Claude designed it this way" but "I stored `patronId` as a string rather than a `Patron` reference because holding a reference would give authentication code access to borrowing history, which violates separation of concerns." The alternative matters. Understanding a design decision means knowing what it costs.

Third: can you trace the non-trivial behavior to a test, a debugger trace, an inspection step, or an explicit reason it was not tested? If a method has a null path you have not exercised, you need a reason — the requirement rules out null input, or the field is always set in the constructor — not just an absence of testing. Untested does not mean correct.

If any answer is no, the component is not ready. Put it back. Ask Claude for an explanation, not a replacement.

---

## The AI Use Disclosure

Every lab requires an AI use disclosure. The disclosure is not a formality. It is evidence that a process happened. A disclosure that says "I used Claude to help with the code" is a confession that the process is not visible. It proves nothing about what you understood or what you verified.

A useful disclosure names the specific task, the prompt or command used, what Claude contributed that was worth keeping, what you changed or rejected and why, and how you verified the part you accepted. Here is an example for Module 9:

```text
I used Claude Code to suggest boundary test cases for Catalog.searchByTitle.
Prompt: "List test cases for a title search that returns a List<Book>.
        Categories: normal, boundary, error, null."
Claude suggested: empty catalog, no match, one match, multiple matches, null query.
I rejected the null-query case — our requirement treats null as an empty string,
not as an exception, so the expected output is an empty list, not a throw.
I wrote the JUnit tests myself. Two failed before I fixed the method.
I own the search requirement and the definition of what counts as a title match.
```

That disclosure proves something. It shows the gate held: Claude suggested, you evaluated, you rejected one case on substantive grounds, you wrote the code, you ran the tests, two failed. The process is visible. The understanding is defensible.

The disclosure also serves a function beyond the lab grade. Writing it makes visible any place where you accepted without verifying. If you cannot fill in "what I changed or rejected," you probably accepted everything. That is the signal to go back.

---

## Permissions and the Project Boundary

Claude Code can be configured with project and user settings that control which files it can read, which commands it can run, and what tools it can invoke. In this course, the configuration concern is simple: do not give any AI tool access to secrets or files you would not paste into a public classroom chat.

This course project should not require real secrets. But the habit matters beyond this course. API keys, database credentials, private keys, and `.env` files should be excluded from AI tool access by default. The student rule is: if you would not paste the file into a public classroom chat, do not expose it to Claude Code without explicit instructor guidance.

The exact setup details — how to configure settings, which tool version to use, which commands are available — change semester to semester and should be verified against current course instructions. This appendix describes the principles. The course setup guide describes the mechanics.

---

## What Ownership Actually Means

The closing rule of this appendix is: Claude Code can make the work faster. It cannot make the work yours.

The work becomes yours through a specific process. You state what you asked for. You describe what came back. You name what you checked and how. You name what you rejected and why. You explain why the final design is defensible — not just "it works" but "it works because it implements this requirement, and I verified that by doing this, and the alternative I rejected would have had this problem."

That process is visible. It produces evidence. It is the standard this course holds you to, because it is the standard a professional engineer is held to. The professional engineer does not say "the AI generated it and it runs." They say "this design choice was deliberate, these are the tradeoffs, this is how I know it satisfies the requirement."

The course moves from diagnostic-only AI use to full synthesis assistance not because the rules become less strict but because your ability to verify expands. In Module 14, Claude Code may do more than in Module 0 because you can evaluate more. The threshold is always the same: you must be able to verify what comes back. What changes is the scope of what you can verify.

By the end of the semester, when you write the final project's `CLAUDE.md`, you are not describing constraints on a tool. You are describing your own engineering judgment — what you own, what you are willing to delegate, and what evidence you require before accepting anything back. That document is a statement of professional practice. Write it as one.

---

## References

<!-- Fact-check pass references. -->
1. Anthropic. Claude Code overview. Anthropic Docs, accessed 2026-05-25. https://docs.anthropic.com/en/docs/claude-code/overview
2. Anthropic. Claude Code settings. Anthropic Docs, accessed 2026-05-25. https://docs.anthropic.com/en/docs/claude-code/settings
3. Anthropic. Security. Anthropic Docs, accessed 2026-05-25. https://docs.anthropic.com/en/docs/claude-code/security
