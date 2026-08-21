# Module 7: Midterm Exam

*The if-else chain is not a bug. It is a warning. The system is trying to tell you that transaction type belongs in the object model.*

---


## EDGE Course Outline Alignment

**Module 7: Midterm Exam**

### Learning Objectives (LOs)

- Demonstrate proficiency in using Java programming language.
- Identify and use appropriate data structures, such as arrays, lists, stacks, and queues, for problem-solving.
- Apply basic programming constructs like loops, conditionals, and data types to solve problems.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Midterm exam

Here is the situation. The library has three kinds of transactions: checkout, reserve, and renew. You write a processor that handles all three. The processor looks at the transaction, decides which kind it is, and does the right thing.

The code works. But it looks like this:

```java
if (transaction.getType().equals("checkout")) {
    // checkout logic
} else if (transaction.getType().equals("reserve")) {
    // reserve logic
} else if (transaction.getType().equals("renew")) {
    // renew logic
}
```

Then the library adds holds. You open the processor. You find the chain. You add another branch. You run it. It works.

Then the library adds interlibrary loans. You open the processor again. You find the chain again. You add another branch.

Then a second processor is written — one that generates receipts. It has its own chain. Then a third that sends notifications. Its own chain. By the time the library handles eight transaction types, the type-checking logic is scattered across a dozen places in the code. Every new transaction type requires finding all of them and updating each one. Miss one, and the system silently does the wrong thing for the new type.

This is not a coincidence. It is a structural signal. When you find yourself writing the same type-checking question in multiple places, the design is telling you that the type should be making its own decisions — not being interrogated from outside.

Polymorphism is the mechanism by which that insight becomes executable.

---

## What Runtime Dispatch Actually Does

Let me explain the machinery before naming the principle.

In Java, every object has two types simultaneously. It has its *actual type* — the class that was used to construct it, the thing `new` was called on. And it has its *declared type* — the type of the variable holding the reference, which may be the same class or any superclass or interface it implements.

When you call a method on a reference, Java does not look at the declared type to decide which method to run. It looks at the actual type. This is called dynamic dispatch: the dispatch — the decision about which method to invoke — happens dynamically, at runtime, based on the actual object, not the declared type of the variable holding it.

Here is why this matters. Suppose `CheckoutTransaction`, `ReserveTransaction`, and `RenewTransaction` are all subclasses of `Transaction`. Suppose each overrides a method called `process()`. Now suppose you have a variable declared as `Transaction t`. That variable might hold any of the three subtypes — you do not know which at the point where you call `t.process()`.

Java does know. When `t.process()` executes, Java looks at the actual object `t` refers to — is it a `CheckoutTransaction`? A `ReserveTransaction`? — and calls that class's `process()` method. The caller does not need to ask. The caller does not need a chain of if-else branches. The caller just calls `process()`, and the right behavior happens because the right object is there.

This is the mechanism. The processor that handles transactions does not need to know which kind it has. It calls `process()`. The transaction handles itself.

![Dynamic dispatch rendered as a systems diagram for the chapter concept.](images/07-midterm-exam-fig-01.png)
*Figure 7.1 — Dynamic dispatch*


---

## The Problem With If-Else Chains

Now I want to show why the if-else chain is the wrong answer to a real problem, rather than just a stylistically inferior one.

The if-else chain concentrates knowledge in the wrong place. When the processor asks `transaction.getType().equals("checkout")`, it is making a decision that the `CheckoutTransaction` object is already capable of making itself. The processor knows about checkout transactions. It knows about reserve transactions. It knows about renew transactions. It has absorbed the logic that should belong to each type.

This is a design violation sometimes called the Open/Closed Principle, though the name matters less than the observation behind it. The processor is not closed to modification — every new transaction type forces it to change. And it is not open to extension in any meaningful sense — "extension" here means editing the processor, which is not extension, it is modification.

The cost of this becomes clear when you trace what has to change when a new transaction type arrives. You find every processor. You find every if-else chain. You add a branch to each. You verify that the new branch behaves correctly. You verify that the existing branches were not disturbed. The work grows with every new type and scales with the number of places the type-checking logic has spread.

Contrast this with the polymorphic design. When a new transaction type arrives, you write a new subclass. You implement `process()`. You write the logic for that type in one place. The existing processors do not change. They already call `process()` on whatever transaction they receive. The new type provides its own answer. Nothing else needs to be touched.

This is what "adding a type without changing existing code" actually means in practice. It is not an aesthetic preference. It is a structural property of the design that makes the system cheaper to extend as requirements accumulate.

![Change-cost trajectory (line chart) rendered as a statistical / quantitative for the chapter concept.](images/07-midterm-exam-fig-02.png)
*Figure 7.2 — Change-cost trajectory (line chart)*


---

## The Inheritance Hierarchy As A Claim

Building the hierarchy is not a Java exercise. It is a claim about the business domain.

When you write `CheckoutTransaction extends Transaction`, you are claiming that a checkout transaction *is a* transaction. Not that it *has* transaction-like properties. Not that it *behaves similarly* to a transaction. That it *is* one — that every obligation a `Transaction` carries, a `CheckoutTransaction` also carries, and that anywhere a `Transaction` is expected, a `CheckoutTransaction` can be substituted without the program noticing the difference.

This is the Liskov Substitution Principle, and it is worth stating plainly before using the acronym: if you claim that B is a subtype of A, then every place in the program that works correctly with an A must also work correctly with a B. If a processor correctly processes a `Transaction`, it must correctly process a `CheckoutTransaction`. If it does not — if the subclass has different preconditions, or violates a postcondition the superclass guarantees, or throws exceptions the superclass never throws — then the inheritance relationship is a lie. The hierarchy says "is-a" but the behavior says "is-sometimes-a" or "is-a-when-used-carefully." That is not a hierarchy. That is a trap.

The practical test is the is-a sentence. Before writing any Java, write the sentence: "A `CheckoutTransaction` is a `Transaction` because ..." and complete it with a business reason, not a Java reason. "Because it extends the class" is circular — it tells you nothing about whether the relationship is valid. "Because every checkout is a kind of library transaction, it shares the same required fields and satisfies the same postconditions on `process()`" is a business reason. It makes a claim you can verify.

If the sentence cannot be completed, the hierarchy is wrong. Full stop. No amount of Java can make a bad inheritance relationship correct.

---

## Tracing The Dispatch

Let me make the dynamic dispatch concrete with the worked example so the mechanism has something specific to run against.

The `TransactionProcessor` class has a method that takes a `Transaction` parameter. Inside that method, it calls `transaction.process()`. The declared type of `transaction` is `Transaction`. At the point the call happens, the processor does not know — and does not need to know — whether the actual object is a `CheckoutTransaction`, a `ReserveTransaction`, or a `RenewTransaction`.

At runtime, suppose a `CheckoutTransaction` was passed in. The JVM looks up the actual type of the object — `CheckoutTransaction` — and searches for a `process()` method defined on that class. It finds the override. It calls it. The checkout-specific logic runs.

Now suppose a `ReserveTransaction` was passed in instead. Same call site. Same processor code. Same line: `transaction.process()`. But now the JVM finds the `ReserveTransaction` override. The reservation-specific logic runs.

The processor did not change. The call site did not change. The behavior changed because the object changed. That is runtime dispatch.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

The student's verification task is to trace this explicitly. Not to confirm that it compiles and runs and produces output. To confirm the dispatch chain: what is the declared type of the parameter? What is the actual type of the object passed in? Which `process()` method was selected? What output or state change does that method produce? And does that output satisfy the requirement for that transaction type?

Running the program once and seeing output is not that evidence. Output tells you the final state. It does not tell you which dispatch path was taken or whether the correct method ran for the correct reason. The evidence requires inspection of the dispatch itself — through the debugger, through a trace, through a test that passes a specific subtype and verifies specific behavior.

---

## Where The If-Else Chain Goes

There is a question the student always asks at this point: if the processor no longer needs an if-else chain, where does the type-specific logic go?

The answer is: it goes into the subclass. The logic that was previously inside the `if (type.equals("checkout"))` branch moves into `CheckoutTransaction.process()`. The logic inside the renew branch moves into `RenewTransaction.process()`. Each transaction type owns its own behavior.

This seems obvious after the fact. Before the polymorphic design, the logic had to live somewhere external, because the transaction objects had no behavior — they were data containers, holding fields and exposing getters, but not making decisions. The if-else chain in the processor was compensating for that absence. Polymorphism is not a new feature laid on top of the old design. It is a reorganization: behavior that belonged to the transaction objects all along moves back to them.

The result is that the processor becomes simpler. It no longer knows about specific transaction types. It knows only that it holds a `Transaction`, and that `Transaction` can process itself. The processor is insulated from the details of each type. Adding a new type does not require opening the processor — the processor was already written to handle whatever `Transaction` it receives.

This is the open/closed property made concrete. The processor is closed to modification for new transaction types. The system is open to extension by adding new subclasses. The door to extension is a new class, not a new branch in existing code.

---

## Where AI Fits Into This Picture

The AI boundary for this module has a specific shape, and the shape matters: AI may propose a hierarchy after the student writes is-a sentences. It must flag questionable inheritance relationships.

The ordering is the point. AI is good at generating plausible hierarchies. Given a description of a business domain, AI will propose superclasses, subclasses, method signatures, and override patterns that compile, look reasonable, and may even run correctly. What AI cannot do is verify the is-a claim. It does not know your business domain well enough to know whether a `HoldTransaction` satisfies the same postconditions as a `CheckoutTransaction`, or whether it introduces obligations the superclass never anticipated.

The student who hands AI the problem without the is-a sentences first will receive a hierarchy. The hierarchy will look correct. It may even be correct. But the student will have no way to evaluate it, because they have not done the prior work of making the claim explicit. They will accept the hierarchy because it compiles and produces output, which is exactly the failure mode this course is teaching against.

The student who writes the is-a sentences first — "A `HoldTransaction` is a `Transaction` because..." — arrives at AI with a claim that can be checked. The AI can confirm the claim, challenge it, or identify a case where the inheritance relationship fails the substitution test. That is AI as a verification partner. The student is not asking AI to do the design. The student has done the design and is asking AI to stress-test it.

The failure mode this boundary protects against is subtler than wrong code. It is a hierarchy that is valid in the test cases the student ran and invalid in the cases they did not think to test. An is-a relationship that holds for three of four subtypes and quietly breaks on the fourth. That failure does not show up until the system encounters the edge case — which may be weeks after the code was written and accepted.

---

## Reading The Hierarchy As A Design Decision

The inheritance hierarchy is not a Java artifact. It is an argument about the domain.

When you look at a hierarchy and ask "is this correct?", you are not asking a Java question. You are asking whether the claimed relationships hold in the business domain. Whether every subclass satisfies the contract of the superclass. Whether the override in each subclass is a legitimate specialization of the superclass behavior or a replacement that violates the contract's spirit.

The most common failure here is inheritance for reuse rather than inheritance for substitution. A developer notices that `CheckoutTransaction` and `RenewTransaction` share several fields and writes one as a subclass of the other to avoid duplicating code. The subclass compiles. The fields are reused. But the is-a relationship is false — a renew is not a kind of checkout, and code that processes a checkout will not necessarily process a renew correctly when one is substituted for the other.

This is the distinction between inheritance and composition. Shared fields and shared behavior can be factored out through composition — a `TransactionBase` object that is held by both classes — without claiming an is-a relationship that the domain does not support. The Java compiler does not enforce the distinction. The LSP test does.

The student's job is to hold both things in view: the Java hierarchy that makes dynamic dispatch work, and the business claim that the hierarchy asserts. The Java works if the syntax is correct. The design works only if the claim is true.

| Item | Meaning |
| --- | --- |
| relationship claim, when to use, what the Java looks like, what breaks if the claim is wrong, library example | Use the chapter example as the concrete test case. |
| helps student see that both are legitimate tools and the choice depends on whether the is-a sentence holds in the domain | A concrete checkpoint for applying the chapter concept. |

---

## The Module's Single Capability

Every module in this course adds exactly one concrete capability to the semester project. This module's capability is: extensible transaction behavior through an inheritance hierarchy, so that new transaction types can be added by writing a new class rather than editing existing code.

In practical terms: the `TransactionProcessor` holds a supertype reference. When it calls `process()`, the actual object decides what happens. You have removed at least one type-checking chain from the processor. The removal is justified by the hierarchy you designed and the is-a sentences you wrote before writing the Java.

By the end of this module, you should be able to look at the processor code and confirm three things: that the declared type of the parameter is the supertype; that the call `transaction.process()` dispatches correctly to each subtype; and that adding a new subtype — sketched on paper, not necessarily implemented — would require writing one new class and no changes to the processor.

If the third condition fails — if adding a new type requires touching the processor — the polymorphism is incomplete. The type-checking logic has merely moved, not been eliminated. Find it. Move it back into the subclass where it belongs.

When you can do this for the library transaction hierarchy, you can do it for inventory and scheduling. The objects get different names. The transaction types are different. The hierarchy must be justified by different is-a sentences. But the structural move — supertype reference, overridden method, dispatch without interrogation — is identical.

![If-else vs. polymorphic before/after rendered as a comparison panels for the chapter concept.](images/07-midterm-exam-fig-03.png)
*Figure 7.3 — If-else vs. polymorphic before/after*


---

## What You Are Building Toward

Modules 1 through 6 gave you objects with state and behavior, collections, relationships, screen flow, and persistence. This module gives you the mechanism for building a system that can grow — where adding capability means adding types, not editing existing logic.

The next question is: where does the persistent record of each transaction live? You have transactions that process themselves. That processing produces a result — a book attached to a patron, a reservation confirmed, a renewal extended. That result has to survive program restart. The record of what happened has to exist somewhere.

That is Module 8's subject. But the shape of the question depends on what you built here. If transactions are just data containers with a type string, the persistence question is simple: save the fields. If transactions are objects with behavior, the persistence question is richer: what state does each type persist, and does the persistence mechanism need to know which type it is saving?

The bridge question is: transactions work. How do their results survive program restart? Hold that question. The polymorphic design you built here shapes the answer in ways that will become clear when Module 8 arrives.

---

## Key Terms

- **superclass** — the parent class in an inheritance hierarchy; defines the shared contract — fields, methods, and their expected behaviors — that all subclasses must satisfy.
- **subclass** — a class that extends a superclass; inherits the superclass contract and may override methods to provide type-specific behavior, provided the override does not violate the contract.
- **override** — a subclass method with the same signature as a superclass method; when the method is called through a supertype reference, the subclass version runs.
- **polymorphism** — the property by which a single call site can invoke different method implementations depending on the actual type of the object at runtime; in Java, enabled by inheritance and method overriding.
- **dynamic dispatch** — the runtime mechanism by which Java selects which method implementation to invoke; it looks at the actual type of the object, not the declared type of the variable holding the reference.
- **LSP** — the Liskov Substitution Principle: if B is a subtype of A, then everywhere an A is expected, a B must be substitutable without breaking the program's correctness; the is-a sentence is the plain-language test.
- **open/closed** — a design property: a class is open for extension (new types can be added) and closed for modification (existing code does not need to change when new types arrive); a polymorphic hierarchy achieves this for the processor that calls the overridden method.

---

## Exercises

**Warm-up**

1. For each of the following, write the is-a sentence and evaluate whether it holds: (a) `CheckoutTransaction` is a `Transaction`; (b) `RenewTransaction` is a `CheckoutTransaction`; (c) `LibraryCard` is a `Transaction`. For each sentence that fails, write one sentence explaining what breaks when the substitution is attempted. *(Tests: applying the LSP is-a test in plain language before writing any Java.)*

2. Trace the dispatch for the following scenario: `TransactionProcessor.handle(Transaction t)` is called with a `ReserveTransaction` object. Write out four things explicitly: the declared type of `t`, the actual type of the object, the method that Java selects, and the class where that method is defined. Then repeat the trace for a `RenewTransaction` passed to the same call site. *(Tests: distinguishing declared type from actual type; tracing runtime dispatch step by step.)*

3. The library's notification service has this code: `if (t.getType().equals("checkout")) sendCheckoutEmail(); else if (t.getType().equals("renew")) sendRenewalEmail();`. The library adds a hold transaction. List every place in the codebase where a developer would need to make a change to support the new type, assuming three processors (checkout, receipt, notification) each have their own chain. Then describe what would need to change in a polymorphic design. *(Tests: quantifying the cost of the if-else approach; seeing the structural advantage of polymorphism before implementing it.)*

**Application**

4. Redesign the `TransactionProcessor` to use polymorphism. Write the `Transaction` superclass with an abstract or overridable `process()` method, and at least three subclasses. For each subclass, write the is-a sentence first. Then implement the override. Show that the processor no longer contains any type-checking logic. *(Tests: executing the full polymorphic design move — is-a sentence, hierarchy, override, removal of if-else chain.)*

5. Translate the transaction hierarchy to your chosen domain. In inventory: identify at least three transaction types (receipt, transfer, adjustment, write-off, or others appropriate to your domain) and write is-a sentences for each against a `Transaction` superclass. In scheduling: identify at least three appointment transaction types (new booking, cancellation, reschedule, waitlist promotion, or others) and do the same. Implement the hierarchy in Java. *(Tests: transferring the polymorphic design move to a new domain; writing is-a sentences grounded in a different business context.)*

6. Write an AI prompt that complies with this module's boundary — it presents at least two is-a sentences and asks AI to evaluate whether the hierarchy is valid and flag any questionable relationships. Run the prompt. In one paragraph, describe one thing AI confirmed, one thing it challenged or flagged, and whether the flag was valid. *(Tests: applying the AI boundary in practice; using AI as a stress-test partner rather than a hierarchy generator.)*

**Synthesis**

7. A teammate proposes inheriting `RenewTransaction` from `CheckoutTransaction` on the grounds that renewals and checkouts share several fields. Write a two-paragraph response. The first paragraph evaluates the is-a claim: does the substitution hold? Is a `RenewTransaction` valid everywhere a `CheckoutTransaction` is expected? The second paragraph proposes a design that achieves the shared fields without the invalid inheritance relationship. *(Tests: distinguishing inheritance-for-substitution from inheritance-for-reuse; applying the composition alternative.)*

8. You have implemented the polymorphic hierarchy and removed the type-checking chain from `TransactionProcessor`. A classmate argues that the old if-else design was simpler and easier to read, and that the hierarchy is unnecessary abstraction. Write a response that concedes the strongest version of their point — when is the if-else chain genuinely the better design? — and then explains the condition under which the polymorphic hierarchy earns its complexity. Ground your answer in the library domain, not in general principles. *(Tests: evaluating polymorphism as a design trade-off rather than a universal improvement; applying the MKBHD lens — what did this design optimize for, and at what cost?)*

**Challenge**

9. The bridge question asks: transactions work — how do their results survive program restart? Before reading Module 8, consider this: if `CheckoutTransaction`, `ReserveTransaction`, and `RenewTransaction` each override `process()`, and each produces a different kind of result (a loan record, a hold queue entry, a renewed due date), what does the persistence layer need to know about transaction types in order to save and reload those results? Does your persistence design need its own if-else chain — or can it also be polymorphic? Write your best current answer. There is no single correct response; the goal is to make your thinking explicit before Module 8 either confirms or complicates it. *(Points toward serialization, polymorphic persistence strategies, and whether the open/closed principle can extend to the data layer.)*

---

## Bridge

Transactions work. How do their results survive program restart?

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Midterm | Summative | For-Credit | Individual | Demonstrate cumulative Java programming proficiency. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation — use for language and library facts, including the specification of method overriding and dynamic dispatch.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming" — use for common novice difficulties with inheritance, including the reuse-vs-substitution confusion.
- Peng et al. and Vaithilingam et al. — use for cautious, empirically grounded claims about AI coding assistance and the verification risks that attend generated hierarchies.

---

## CLI Quick Reference

```bash
java --version
javac --version
# Run from your project directory when checking environment and compiled output.
```

---

## References

<!-- Fact-check pass references. -->
No references added by fact-check pass.
