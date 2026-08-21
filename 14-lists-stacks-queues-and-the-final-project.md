# Module 14: Lists, Stacks, Queues, and the Final Project

*Running is not done. Done is running, explained, defended, and handed off.*

---


## EDGE Course Outline Alignment

**Module 14: Lists, Stacks, Queues, and the Final Project**

### Learning Objectives (LOs)

- Apply learned knowledge to create a GUI-based application to address an identified problem.
- Recall Lists, Stacks, and Queues.
- Use Lists, Stacks, and Queues to solve programming problems.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- The List Interface
- The Comparator Interface
- Queues and Priority Queues
- The List and ListIterator Interfaces
- ArrayList and LinkedList Classes
- Vector and Stack

Here is a question the examiner will ask. Not a guess — it will be asked, in some form, at every professional code review for the rest of your career:

"What does this component do, why did you design it this way, and what would you change?"

Three parts. The first is factual. The second is architectural. The third is the honest one — the one that separates people who built something from people who assembled something and hoped it worked.

If your answer to the first part is "I'm not sure, the AI wrote it," the review is over. If your answer to the second part is "I don't know, it was in a tutorial," the review is over. If your answer to the third part is "nothing, it's fine," the review is also over, because nothing is ever fine — there are always trade-offs, and someone who cannot name them has not thought about what they built.

The final project is practice for that moment. Not the code — the explanation. The code is necessary but insufficient. Done means running code, source code, tests, AI Use Disclosures, and a defense. The examiner can ask about any component. Your job is to be able to answer.

This module is about what it takes to reach that standard.

---

## What a Professional Handoff Requires

The word "done" has a specific meaning in professional software development. It does not mean the program runs. It means the program runs, can be understood by someone who did not write it, can be maintained by someone who was not in the room, and includes enough evidence that the behavior matches the requirements.

For this course, done means five things.

**Running code.** The application launches, demonstrates the primary use cases, and does not crash on the inputs the examiner will use. A demo that requires you to avoid certain buttons is not done.

**Source code.** The code is readable, organized, and accompanied by enough structure that someone else could find the class that handles checkout, the method that loads the catalog, the handler that responds to the search button. Spaghetti that works is not done.

**Tests.** There is evidence that the critical behaviors were verified. Not a complete test suite — this is a semester project, not a production system — but tests for the behaviors most likely to break: the persistence round-trip, the checkout state change, the edge case that produced a bug during development. Code with no tests is code with no evidence.

**AI Use Disclosures.** Every component where AI generated more than boilerplate has a disclosure: what was asked, what was received, what was wrong or incomplete, and what the student decided that AI could not. Disclosures in commit comments are sufficient. Undisclosed AI assistance is academic dishonesty in the same way that undisclosed collaboration is.

**A defense.** Three design decisions, explained. The reasoning behind each. What alternatives were considered. What would be changed given more time. The defense is not a performance — it is evidence that the student built the system, not just submitted it.

These five things together are a professional handoff. Any one missing and the handoff is incomplete.

| component | what it is | what "done" means for this component | the failure mode if it is missing |
| --- | --- | --- | --- |
| running code (no crash on examiner's inputs | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| source code (navigable by someone else | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| tests (evidence of critical behaviors | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| AI Use Disclosures (specific task | prompt | verification | judgment |
| defense (three decisions in strong format) | student uses this as a pre-submission gate | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |

---

## The Architecture You Built

Over the course of this semester, you built an architecture in layers. It is worth naming the layers explicitly now, because the defense will ask you to explain them.

The supply side — entities, relationships, and the catalog — came first. A `Book` is an entity. An `Author` is an entity. The `Catalog` holds entities and answers queries about them. None of these know about users, sessions, or screens. They model the world the application operates in.

The transaction layer came second. A `CheckoutTransaction` records that a patron borrowed a book on a date with a due date. A `PatronSession` tracks the currently authenticated user. Transactions use entities — they cannot exist without them — but they do not own entities. The catalog is not recreated by the checkout.

Persistence arrived in Module 8. The catalog loads from CSV at startup and saves at defined checkpoints. The transaction log persists separately. Both survive program restart. Neither depends on the UI to exist.

The view layer — JavaFX, `ListView`, `Button`, `Label` — came after the model was working. The view displays model state. It does not own model state. When the model changes, the view reflects the change through a refresh method called by the handler.

The event layer connected user actions to the model through handlers that translate, not execute. The handler calls `controller.performCheckout()`. The handler does not implement checkout.

What you have built is a layered system where each layer has a defined responsibility, a defined boundary, and a defined interface to the layers above and below it. The examiner's question — "why did you design it this way?" — is answered by pointing to the layers and explaining what each one is for.

![Five-layer project architecture rendered as a hierarchy / layered stack for the chapter concept.](images/14-lists-stacks-queues-and-the-final-project-fig-01.png)
*Figure 14.1 — Five-layer project architecture*


---

## The Three-Question AI Audit

Every AI-assisted component in the final project must pass three questions before submission. Not as a formality — as evidence that you understood what AI did and what you decided.

**Question one: What did you ask AI to do, and what did it produce?**

The answer is specific. Not "I used AI for the persistence layer" but "I asked AI to generate a `saveToFile` method given this signature and this CSV format, and it produced a method that did not handle the case where the file is missing on first run."

Specificity matters because vague disclosures are not disclosures. "AI helped me write code" describes every student in the course. The disclosure that matters names the task, the prompt, and the output — including where the output was wrong.

**Question two: What did you verify, and how?**

The answer names a specific artifact and a specific verification method. Not "I tested it" but "I ran the application with an empty catalog directory, confirmed the load method returned without error, and confirmed the save method produced a file with the correct CSV structure."

This question is the core of the book's thesis. AI can generate code. AI cannot run your application against your requirements. AI cannot observe that the edge case you did not think of produced the wrong behavior. Verification is human work, and it requires naming what was verified and how.

**Question three: What did you decide that AI could not?**

The answer names a judgment. Not a line of code — a decision. "AI could not decide whether a missing file at startup should be treated as an error or as a normal first-run condition, because that depends on the application's deployment context. I decided it should be silent on first run and log a warning in subsequent runs."

This is the question that separates engineering from transcription. The code may look identical regardless of which decision was made. The decision determines the behavior in the edge case. The edge case determines whether the application is correct. AI does not know the deployment context. You do.

Three questions. Every AI-assisted component. No exceptions.

![AI audit form rendered as a annotated example for the chapter concept.](images/14-lists-stacks-queues-and-the-final-project-fig-02.png)
*Figure 14.3 — AI audit form*


---

## Defending a Design Decision

The defense asks you to explain three design decisions. Here is what a good defense looks like, and what a weak one looks like, for the same decision.

The decision: ISBN lookup uses a `HashMap<String, Book>` instead of a linear search through an array.

**Weak defense:** "I used a HashMap because it's faster."

This tells the examiner that you know HashMaps are faster than linear search. It does not tell them that you know why, or when that matters, or what the HashMap costs, or whether the trade-off was appropriate for this application.

**Strong defense:** "The catalog has up to several hundred books. Checkout requires an ISBN lookup every time a patron scans a barcode. Linear search through an array takes time proportional to the number of books — on average, half the catalog per lookup. A HashMap lookup takes constant time regardless of catalog size. The cost is memory: the HashMap stores keys and values plus the internal structure. For a catalog this size, that cost is negligible. I chose the HashMap because the lookup frequency justifies it. If the catalog were ten books, I would have kept the array — the overhead of maintaining a HashMap for ten entries is not worth it."

This tells the examiner three things: you understand how both data structures work, you understand the trade-off, and you applied judgment to the specific requirements of your application. That is what a design decision is.

The same structure applies to every decision you defend:

- What alternatives existed?
- What does each alternative cost and gain?
- What was true about your specific requirements that made one alternative better than the other?
- If the requirements changed, which decision would change?

Three decisions, prepared in this structure, before the defense.


| question | weak answer (what it reveals) | strong answer (what it demonstrates) |
| --- | --- | --- |
| "What does this component do?" (weak: restates the code | strong: names the requirement it satisfies | A specific, evidence-linked version that readers can verify. |
| "What alternatives existed?" (weak: "I didn't think of any" | strong: names at least one with its cost | A specific, evidence-linked version that readers can verify. |
| "Why this choice for your requirements?" (weak: "it's standard practice" | strong: names the specific requirement that made this better | A specific, evidence-linked version that readers can verify. |
| "What would you change?" (weak: "nothing" | strong: names a trade-off that a different requirement would flip) — | A specific, evidence-linked version that readers can verify. |

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Where Requirements Prohibited AI Delegation

The final project requires identifying three places where requirements prohibited AI delegation. This is not a gotcha — it is the most important thing the course has been teaching.

AI can generate correct Java. AI can generate correct Java that does the wrong thing, because the wrong thing and the right thing often look identical in code. The places where requirements prohibited delegation are the places where "looking correct" was not enough — where the student had to know what the requirement actually was and verify that the code actually met it.

Here are the three most common answers students give, and what they reveal.

**"I had to write the validation logic myself because AI didn't know my requirements."**

This is correct, and it names the right reason. Validation logic depends on what counts as valid input in your domain. A library application might require ISBNs to be 13 digits. An inventory application might require quantities to be positive integers. A healthcare scheduling application might require that appointment times do not overlap for the same provider. AI does not know these rules unless you state them explicitly, and even when you state them, AI may implement them in a way that does not match your model's data types or your error-handling strategy.

**"I had to write the checkout state machine myself because AI generated a version that allowed double-checkout."**

This is a better answer. It names a specific failure in AI output and a specific requirement it violated. A book that is already checked out cannot be checked out again. That rule is not inferrable from the class structure alone — it depends on the semantics of the availability flag and the business rule that exactly one patron holds a book at a time. The student who caught this failure understood both the code and the requirement.

**"I had to write the authentication logic myself because AI generated a version that stored passwords in plain text."**

This is the best kind of answer. It names a security requirement, identifies AI's failure to satisfy it, and names the decision the student made instead. Even if plain-text storage would have passed the functional tests, the requirement prohibits it — and recognizing that prohibition, and acting on it, is engineering judgment.

The three places you identify should be specific. Not "I couldn't use AI for the model" but "I couldn't use AI to decide what the Catalog's initial state should be on first run, because that depends on whether the application is expected to fail loudly or recover silently, and that is a deployment decision, not a coding decision."

| answer quality | what it names | what it reveals to the examiner |
| --- | --- | --- |
| level 1 "validation logic because AI didn't know my requirements" (names reason, generic | It makes the underlying reasoning visible instead of implied. | A concrete checkpoint for applying the chapter concept. |
| level 2 "checkout state machine because AI allowed double-checkout" (names specific AI failure and violated requirement, better | The pattern becomes easy to misuse or overlook. | A concrete checkpoint for applying the chapter concept. |
| level 3 "authentication because AI stored passwords in plain text" (names security requirement + failure + human decision, best) — | The pattern becomes easy to misuse or overlook. | A concrete checkpoint for applying the chapter concept. |

---

## Testing as Evidence

Tests are not an optional addition to the final project. They are evidence.

The examiner can ask: "How do you know the persistence round-trip works?" There are two possible answers. One is "I ran it a few times and it seemed to work." The other is "I have a test that adds three books, saves to a temporary file, creates a new catalog instance, loads from that file, and asserts that the catalog contains the same three books with the same field values." The second answer is evidence. The first is an impression.

For the final project, three categories of tests matter most.

**Persistence tests.** Create objects, save them, load them into a new instance, and verify that the loaded state matches the saved state. This test is worth writing because persistence bugs are invisible to casual observation — the application can appear to work while silently losing data on every restart.

**State transition tests.** Verify that the model enforces its rules. A book cannot be checked out if it is already checked out. A patron cannot have more checkouts than the limit allows. A deleted record does not reappear after reload. These rules are the business logic of your domain, and a test that fails on them is a test that found a real bug.

**Edge case tests.** The cases you know are wrong. An empty catalog. An ISBN that does not exist. A CSV file with a malformed line. A patron with no checkouts attempting a return. These tests verify that the error handling you wrote in earlier modules actually works for the cases that will occur in practice.

Tests are not proof of correctness. They are evidence of reasoning. A student who wrote tests for specific behaviors has demonstrated that they thought about what the correct behavior should be and checked whether the code produces it. That is the verification habit the course has been building toward.

![Impression vs. evidence rendered as a comparison panels for the chapter concept.](images/14-lists-stacks-queues-and-the-final-project-fig-03.png)
*Figure 14.4 — Impression vs. evidence*


| category | what it tests | example test description | what it proves to the examiner |
| --- | --- | --- | --- |
| persistence tests (round-trip survival, "save three books, reload, assert field equality", that the data layer works across restarts | A specific, evidence-linked version that readers can verify. | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |
| state transition tests (business rule enforcement, "attempt double-checkout, assert rejection", that the model enforces domain rules | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |
| edge case tests (error handling under abnormal input, "load malformed CSV, assert no crash", that the error handling from earlier modules actually fires) | student uses this to confirm all three categories are covered before the demo | Use the chapter example as the concrete test case. | A concrete checkpoint for applying the chapter concept. |

---

## The Verification Habit

Every module in this course ended with the same question in a new form: can you trace the behavior, explain the design choice, and identify what AI did and did not decide?

The answer was always the same structure. Here is the behavior. Here is the artifact that carries it. Here is what AI generated. Here is what I verified. Here is what I decided.

The final project is not a new question. It is the same question, asked about the whole system.

The verification habit is not specific to Java or to JavaFX or to this course. It applies to any system where code is generated by a tool that does not know your requirements — which, at this moment in the history of software, includes AI tools, code generators, library implementations, and any code copied from documentation or Stack Overflow. The tool generates code. The engineer verifies that the code satisfies the requirement.

The habit is this: before accepting any piece of code as correct, name the requirement it must satisfy and name the observation that confirms it satisfies that requirement. The requirement can be stated in one sentence. The observation can be a test, a trace, a run against a known input, or an inspection of the code path for the case in question. If you cannot name both, you have not yet verified the code.

This is not a high bar. It is the minimum bar. It is what separates code that happens to work from code you can defend.

---

## What the Course Built

You arrived at this course with the ability to write sequential programs in Java. Variables, loops, conditionals, methods. You leave it with something different.

You can design a layered system: supply side before transaction, model before view, interface before implementation. You understand that the order matters because each layer depends on the ones below it, and building the layers in the wrong order produces a system that works in the happy path and breaks at the first requirement change.

You can implement persistence: the round-trip from object to file to object, with exception handling that reflects a deliberate decision about failure modes rather than a default that swallows errors silently.

You can build a GUI application: a view that displays model state without owning it, handlers that translate events without implementing business logic, a controller layer that keeps the two separate.

You can collaborate with AI in a way that preserves your engineering judgment: using AI for the things it is good at — boilerplate, syntax, scaffolding — while retaining responsibility for the things it cannot do — knowing the requirement, deciding the failure mode, verifying the behavior.

And you can defend what you built. Not just show it — explain it. Name the trade-offs. Name the alternatives. Name what you would change.

That is the whole course, compressed into the final defense. Not the running program — the explanation of the running program. The running program is evidence that the code works. The explanation is evidence that you built it.

The bridge question this course leaves you with is this: the verification habit should travel to the next system. Every tool will change. The requirement to verify will not.

![Course-capability arc rendered as a comparison panels for the chapter concept.](images/14-lists-stacks-queues-and-the-final-project-fig-04.png)
*Figure 14.5 — Course-capability arc*


---

## Exercises

**Warm-up**

1. Write the three-question AI audit for one component you built using AI assistance. Be specific: name the task, the prompt summary, the output, what was wrong or incomplete, and what you decided that AI could not. If you cannot name something that was wrong or incomplete, inspect the component more carefully — there is always something. *(Tests: AI disclosure practice, critical inspection of AI output, judgment identification.)*

2. State one design decision you made in any earlier module in the strong-defense format: alternatives considered, cost and gain of each, what was true about your requirements that made your choice better, and what would change if the requirements changed. Do this in prose, not code. *(Tests: design decision articulation, trade-off reasoning, requirements connection.)*

**Application**

3. Write a persistence round-trip test: instantiate entities, save to a temporary file, create a new collection instance, load from the file, assert field equality between the original and loaded objects. Run it. If it fails, fix the persistence layer until it passes. Document the failure and the fix. *(Tests: persistence verification, test writing, debugging discipline.)*

4. Write a state transition test for one business rule in your domain: a rule that the model must enforce and that AI output might have violated. Name the rule in plain English before writing the test. The test should fail if the rule is not enforced and pass only if it is. *(Tests: business rule identification, state machine testing, requirements verification.)*

**Synthesis**

5. Identify three places in your project where requirements prohibited AI delegation. For each one: name the requirement, name the specific thing AI generated or would have generated that violated it, and name the judgment you applied instead. The three places should be from different layers of the system — not three things from the same module. *(Tests: cross-system audit, delegation boundary identification, judgment articulation.)*

6. Prepare the defense. Write out three design decisions in the strong format: alternatives, costs and gains, requirements context, what would change. Then read them aloud — not to anyone, just aloud — and notice where you hesitate. A hesitation is a place where the understanding is incomplete. Return to the module that introduced that concept and close the gap before the demo. *(Tests: defense preparation, self-assessment, knowledge gap identification.)*

**Challenge**

7. Write a short reflection — one page, prose — on the question: what would you build differently if you started the project today, knowing what you know now? The reflection should name at least one architectural decision, at least one AI collaboration decision, and at least one testing decision. It should not be a list of regrets — it should be a statement of what you learned, expressed as what you would do differently. This reflection is the truest evidence of the course's central goal: engineers who can explain their systems, including the parts they would revise. *(Tests: synthesis of course learning, honest self-assessment, professional reflection.)*

---

## Key Terms

- **ecosystem:** The complete set of interconnected components — supply side, transaction layer, persistence, view, event handlers, tests — that together constitute a working application. Define this term in the context of your semester project before using it in lab work.
- **code review:** A systematic examination of source code by someone other than the author, with the goal of finding defects, verifying design decisions, and confirming that the code satisfies its requirements. Define this term in the context of your semester project before using it in lab work.
- **defense:** An oral or written explanation of design decisions, trade-offs, and AI use, presented to an examiner who can ask follow-up questions. Define this term in the context of your semester project before using it in lab work.
- **AI audit:** The three-question process applied to every AI-assisted component: what was asked and received, what was verified and how, what was decided that AI could not. Define this term in the context of your semester project before using it in lab work.
- **handoff:** The act of transferring a completed system to another person, team, or context in a form that allows them to understand, maintain, and extend it without access to the original author. Define this term in the context of your semester project before using it in lab work.
- **deployment:** The process of making a completed application available for its intended use — which includes not just running the code but confirming that it behaves correctly in the environment where it will be used. Define this term in the context of your semester project before using it in lab work.
- **reflection:** A deliberate examination of what was built, why, and what would be done differently — the evidence that understanding has moved beyond execution to judgment. Define this term in the context of your semester project before using it in lab work.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Exercise: Programming with ArrayList and LinkedList Classes | Formative | For-Credit | Individual | Choose and use list implementations for a programming problem. |
| Discussion - Vector and Stack | Community | For-Credit | Individual | Compare legacy and current collection choices. |
| Optional Exercise: Queue Interface and LinkedList Class | Practice | For-Credit | Individual | Use queue operations in a Java program. |
| Quiz - Lists, Stacks, and Queues | Summative | For-Credit | Individual | Assess list, stack, queue, and comparator concepts. |
| Final GUI Application Project | Summative | For-Credit | Individual or Group as assigned | Apply the course concepts to a GUI-based real-world application. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation: authoritative source for language and library facts.
- OpenJFX documentation: authoritative source for JavaFX application structure, event handling, and deployment.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": on common novice difficulties and what instruction actually changes.
- Peng et al. and Vaithilingam et al.: on cautious claims about AI coding assistance, verification risk, and what delegation costs.

---

## References

<!-- Fact-check pass references. -->
1. Oracle. Class Stack. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Stack.html
