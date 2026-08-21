# Module 3: Objects and Classes
*The screen changes. The object does not. Here is why that is the bug.*

There is a failure that looks like a UI problem but is actually an object problem. The user searches for a book, selects it, and proceeds to checkout. The checkout screen displays the right title. They confirm. The confirmation screen shows a different book — or no book at all, or a book whose fields are partially populated in ways that do not match what was selected.

The screen changed correctly. The layout switched. The panels rendered. Java did exactly what the code said. The bug is not in the navigation machinery. The bug is that the `Book` object did not travel with the user. Each screen created its own version of the state, or assumed state that was never passed, or read from a field that was reset when the panel became visible. The program has a flow. The object does not.

I want to start here because this confusion is architectural, not syntactic. You will not find it by reading the code for a typo. You find it by asking a question most beginners do not think to ask: which object is the user actually holding as they move through the application, and what is each screen allowed to do to it?

That question is what this module is about.

---


## EDGE Course Outline Alignment

**Module 3: Objects and Classes**

### Learning Objectives (LOs)

- Describe objects and classes.
- Analyze how objects are initialized using constructors.
- Differentiate between instance and static variables/methods.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Object-oriented programming
- UML Diagrams
- Constructors
- Instance variables and methods
- Static Variables, Constants, and Methods
- Visibility Modifiers and Accessor/Mutator Methods

## What a User Flow Actually Is

The temptation is to think about user flows as screen sequences. The designer draws boxes — Search, Result, Checkout, Confirmation — and arrows between them. The programmer implements the arrows as button handlers that swap visible panels. The flow works in the demo. It fails in the edge case where the user navigates backward, or where the Result screen was reached by two different paths, or where the Confirmation screen needs a field that the Checkout screen never explicitly set.

These failures share a root. The engineer thought about screens. They should have thought about state.

A user flow is not a sequence of screens. It is a path through the state space of the application. The screens are how the user sees and changes that state. But the state — the objects, their fields, their relationships — exists independently of what is displayed. When the Checkout screen shows a book title, it is reading a field on a `Book` object. When the Confirmation screen shows an expected return date, it is reading a field on a `Loan` object. Those objects have to exist, have to carry the right values, and have to be the *same* objects — not copies, not fresh instances — at every step of the flow.

The engineer's job is to decide which objects travel through the flow and what each step is permitted to change.

![Screen-centric vs. state-centric mental model rendered as a comparison panels for the chapter concept.](images/03-objects-and-classes-fig-01.png)
*Figure 3.1 — Screen-centric vs. state-centric mental model*


This is the shift in perspective that makes user interaction design teachable. Once you see the flow as a state path, the questions become concrete: what does this screen read? What does it write? What must already be true when the user arrives here? What must be true when they leave?

---

## The Machinery: CardLayout and the Panel Model

Java Swing gives you a layout manager called `CardLayout` that is designed for exactly this problem. Let me explain what it actually does, because the name is a little misleading.

`CardLayout` manages a container — typically a `JPanel` — that holds multiple child panels, one for each "screen" in your flow. At any moment, only one child panel is visible. Switching screens means telling the `CardLayout` to show a different child. The invisible panels still exist. Their objects still hold their values. Nothing is destroyed when you navigate away. This is the mechanical property that makes `CardLayout` the right tool: the hidden panels are dormant, not dead.

Here is the basic structure. You create a container panel and give it a `CardLayout`:

```java
JPanel container = new JPanel();
CardLayout layout = new CardLayout();
container.setLayout(layout);
```

Then you add your screen panels to the container, each with a string name:

```java
container.add(searchPanel, "search");
container.add(resultPanel, "result");
container.add(checkoutPanel, "checkout");
container.add(confirmationPanel, "confirmation");
```

To navigate, you call `show` on the layout:

```java
layout.show(container, "checkout");
```

That is the mechanical part. What `CardLayout` does not do — what it cannot do, because it has no knowledge of your domain — is move the `Book` object from one panel to the next. That is your responsibility.

![CardLayout container structure rendered as a structural schematic for the chapter concept.](images/03-objects-and-classes-fig-02.png)
*Figure 3.2 — CardLayout container structure*


The standard pattern is to give each panel a setter method that accepts the object it needs before it becomes visible:

```java
checkoutPanel.setBook(selectedBook);
layout.show(container, "checkout");
```

Notice the order. You set the state first. Then you show the panel. If you show the panel first, the panel's display methods may run before the object is set — and you get the failure from the opening: the right screen, the wrong state.

This order matters. It is not a style preference. It is the contract between your navigation logic and your panels.

---

## The Object That Travels

In the library flow, the central object is a `Book`. The user searches, selects a `Book` from the results, proceeds to checkout, and confirms. At every step after selection, the same `Book` reference should be in scope. Not a copy. The reference.

Why not a copy? Because the application may need to update the `Book`'s state — marking it as checked out, recording the borrower, setting the due date — and those updates need to persist. If each panel holds a copy, updates in Checkout are invisible to Confirmation. The Confirmation screen shows the pre-checkout state. The object in memory shows the post-checkout state. They are different objects. This is the bug.

Java passes object references, not object values. When you write:

```java
checkoutPanel.setBook(selectedBook);
```

you are not copying the `Book`. You are giving `checkoutPanel` a reference to the same `Book` object that the result panel is already pointing at. Any change `checkoutPanel` makes to the book's fields is visible to anyone else holding a reference to the same object. This is the behavior you want. It is also the behavior that can surprise you if you are not thinking about it, because two panels can now interfere with each other through shared state.

The rule that follows from this: decide explicitly which screen owns each piece of state and which screens only read it. The search screen owns nothing — it produces a selected `Book`. The result panel holds a `Book` reference and passes it forward. The checkout panel reads from the `Book` and writes to a new `Loan` object. The confirmation panel reads from both. Each screen has a defined role in the state lifecycle.

When the roles are explicit, bugs become visible. When they are implicit, any screen can modify any field at any time, and tracing the failure requires reading every panel's code simultaneously.

| screen | objects it reads | objects it may modify | state that must be true on entry | state that must be true on exit |
| --- | --- | --- | --- | --- |
| SearchPanel, ResultPanel, CheckoutPanel, ConfirmationPanel | the student can use this as a template for their own domain | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Tracing the Lifecycle

I find it useful to trace a complete user action from click to visible result, because doing it once makes the invisible machinery concrete.

The user is on the search screen. They type "Dune" and click Search. Here is what happens:

The search button's `ActionListener` fires. Inside the listener, the application queries the book collection for titles matching "Dune" and gets back a list of `Book` objects. The result panel's `setResults(List<Book>)` method is called with that list. The result panel populates its display — a list or table showing the results — by reading fields from each `Book`. Then `layout.show(container, "result")` makes the result panel visible.

The user selects "Dune" from the list and clicks Select. The result panel's selection listener fires. It identifies the selected `Book` — not the title string, the `Book` object — and calls `checkoutPanel.setBook(selectedBook)`. Then it shows the checkout panel.

The checkout panel's `setBook` method stores the reference and populates its display fields from the book's current state: title, author, available copies. The user confirms the loan. The checkout panel's confirm handler creates a new `Loan` object, sets its `book` field to the same `Book` reference, records the borrower and due date, and calls `confirmationPanel.setLoan(newLoan)`. Then it shows the confirmation panel.

The confirmation panel reads from the `Loan` object — which carries its own `Book` reference — and displays the summary.

Trace that sequence once with your own domain object and you will notice something: the application never needs to re-query the database for the book after the initial search. The reference carries forward. Every subsequent screen is reading from the same in-memory object. This is efficient, but it also means that if you mutate the object incorrectly at any step, every subsequent screen shows the corrupted state.

---

## What the AI Boundary Protects

AI can generate a `CardLayout` skeleton in seconds. The container, the panels, the `show` calls — that scaffolding is not where the design work is. Hand it to AI and verify the output. The structure is mechanical and the verification is straightforward: does the container have a `CardLayout`? Are the panels named consistently? Does the show call happen after the setter?

What AI cannot generate is the state design. The question of which object travels through the flow, who owns it, which screen may modify which fields, and what must be true at each transition — that design is yours. It requires knowing the business process, the domain objects from the previous modules, and the requirements that your specific project must satisfy. AI does not know those things. It can guess at them, and a plausible guess in a `CardLayout` skeleton will look correct until the edge case where the user navigates backward, or searches for a book that is already checked out, or confirms a loan without completing the checkout step.

The phase gate for this module follows from this. Before you ask AI for anything, write the flow in a table: screen, user action, object involved, state change. Four columns. One row per step. When that table exists and you have verified that it reflects your actual business requirement, AI can scaffold the Java. The table is the spec. The AI produces a candidate implementation of the spec. You verify the implementation against the spec.

Without the table, AI produces a candidate implementation of its best guess at your requirement. The guess will be reasonable. It will not be yours.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

Here is the prompt that keeps the boundary in place:

```text
I am working on Module 3: Objects and Classes in INFO 5100.
My domain is [library / inventory / scheduling].
My current specification is: [paste your written flow table].
Help only within this boundary: AI may generate CardLayout skeleton code
and panel switching. It may not generate business logic or decide what
state moves between screens.
Do not produce work that violates the phase gate: the flow table above
must already exist before you generate any code.
```

The prompt works because it gives AI the spec — your flow table — and closes off the decisions that are not AI's to make. AI sees the table and generates code to implement it. You verify that the code implements the table correctly. The design judgment happened before AI was involved.

---

## The Separation of Concerns Behind CardLayout

There is a design philosophy embedded in the `CardLayout` pattern that is worth naming explicitly, because it recurs across the whole course.

The layout manager is responsible for visibility. The panels are responsible for display. The objects are responsible for state. These three responsibilities are deliberately separated. `CardLayout` does not know anything about `Book` objects. A `JPanel` does not know anything about which other panels are visible. The `Book` object does not know anything about how it is displayed or where in the navigation flow it currently is.

This separation is called *separation of concerns*, and it is the design move that makes large applications maintainable. When concerns are separated, you can change one without affecting the others. You can swap the confirmation panel's layout without touching the `Loan` object. You can add a new field to `Book` without changing the navigation logic. You can change the flow — add a step between Checkout and Confirmation — without rewriting the domain objects.

When concerns are mixed — when a panel directly modifies the object that another panel is reading, or when the navigation logic is embedded inside a domain object — changes in one place produce unexpected effects elsewhere. The application still works, in the sense that it compiles and runs. It becomes harder to understand, modify, and debug with every feature added.

The `CardLayout` pattern enforces some of this separation structurally: the layout manager cannot modify your objects because it does not know they exist. But the separation between panels and objects is enforced only by the engineer's discipline. Java will not stop you from writing business logic inside a button handler. The module's design constraint — the flow table before the code — is a tool for maintaining that discipline before the application is complex enough to make violations painful.

![Three-layer responsibility model rendered as a hierarchy / layered stack for the chapter concept.](images/03-objects-and-classes-fig-03.png)
*Figure 3.3 — Three-layer responsibility model*


---

## The Inventory and Scheduling Shape

The library flow is the main path through this module's examples, but the same design shape applies to any domain.

In an inventory system, the traveling object is likely an `Item` or a `StockEntry`. The flow might be: Search for item, View item details, Edit quantity, Confirm update. The `Item` reference travels from search to detail to edit. The edit screen is the only one with write permission on the quantity field. The confirmation screen reads the updated value and displays the change summary.

In a healthcare scheduling application, the traveling object is likely an `Appointment` being created or an existing `Appointment` being modified. The flow might be: Select patient, Choose provider, Select time slot, Confirm booking. Here the object may be assembled across multiple screens — the `Appointment` starts partially constructed and gains fields at each step — rather than retrieved fully formed from search. That pattern requires a staging object: an `Appointment` with some fields populated, passed forward, and progressively completed.

The staging object pattern is a small extension of what we have covered. The key constraint is the same: the engineer decides which screen is allowed to set which field. The confirmation screen must not set any field. It reads only. If the confirmation screen is allowed to write to the appointment, a display bug becomes a data corruption bug.

These three domains have different objects and different flows, but the questions are identical: what travels, who owns it, what is each step allowed to change?

---

## What Breaks When This Goes Wrong

Let me describe the failure specifically, because naming it precisely is most of the protection.

The most common failure is the lost reference. The search panel constructs a new `Book` object to display results. The checkout panel constructs another new `Book` object to display the selection. They look identical — same title, same author, same fields — because they were both built from the same database row. But they are different objects. An update made to the checkout panel's book is not visible on the confirmation panel, which holds a reference to a third `Book` object created when the confirmation panel loaded.

This failure does not appear in a demo where the flow runs forward once without edge cases. It appears when the application is tested against the actual requirement: that a loan modification made in checkout persists to confirmation.

The second common failure is premature display. A panel's `setVisible(true)` call runs before its `setBook` call. The panel's constructor or initialization code tries to read from a null reference. The application throws a `NullPointerException` on navigation, or silently displays empty fields, or displays the previous user's data if the reference was set in a prior session.

Both failures are caught by the same verification practice: after implementing the flow, trace the object reference manually from the moment of selection through every screen. Not the visual output — the reference. Confirm that each panel's setter is called before the panel becomes visible. Confirm that the same reference, not a copy, is passed forward. Confirm that no screen writes to a field it is not supposed to own.

That trace is the evidence the lab asks for. Not "it ran." Not "it compiled." The trace.

| failure name | what the user sees | what the code shows | which layer the bug lives in | which verification step catches it |
| --- | --- | --- | --- | --- |
| Lost Reference, Premature Display | students should be able to map both failures to the three-layer model and to the setter-before-show contract | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## What This Module Does Not Settle

We have connected screens to objects and established the discipline of deciding who owns what state. We have not addressed what happens when the flow needs to handle errors: the book is unavailable when the user reaches checkout, or the database write fails at confirmation, or the user navigates backward and the object is in a partially modified state.

Error handling in flows is the question this module raises without answering. It appears fully in a later module, when the project is complex enough for the failure modes to be concrete rather than hypothetical.

The bridge question to carry forward is this: the screen flow works. How do you diagnose it when it does not?

---

## Lab: Two-Screen Flow

Design and implement a two-screen flow for your semester project domain. Before writing any code, submit the flow table: screen, user action, object involved, state change. One row per step.

The implementation must include a `CardLayout` container, at least two panels with setter methods, and navigation code that calls the setter before showing the panel. Identify in writing: the object that travels between screens, which panel is permitted to modify which fields, and the evidence that the correct reference — not a copy — is passed forward.

If you used AI to generate the CardLayout skeleton, include the AI Use Disclosure: what you asked, what the AI produced, what you verified, and what the AI did not and could not decide.

| screen | user action | object involved | field(s) read | field(s) written |
| --- | --- | --- | --- | --- |
| Core idea | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| Practical use | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| Common failure | The pattern becomes easy to misuse or overlook. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Exercises

**Warm-up**

1. Draw the state path — not the screen sequence — for your semester project's two-screen flow. For each step, name the object involved, the fields it reads, and the fields it writes. If a step has no object, name what is missing and where it would come from. *(Tests: screen-vs-state distinction; flow-as-state-path.)*

2. Write the setter method signature for one panel in your flow. Then write the two lines of navigation code — setter call and `layout.show()` — in the correct order. Explain in one sentence why the order is a contract, not a preference. *(Tests: setter-before-show contract; CardLayout mechanics.)*

**Application**

3. A classmate's checkout panel constructs a new `Book` object by reading the title string from a label on the result panel, then calling the database to fetch a matching record. The confirmation panel shows the correct title. Explain why this implementation may still be wrong, using the concept of reference identity. Which of the two common failures does this most resemble, and what test would expose it? *(Tests: reference vs. copy distinction; lost-reference failure mode.)*

4. Your flow needs a backward navigation: the user can return from Checkout to Result and change their selection. Describe what must happen to the object state when they navigate back. Which panel, if any, needs its state reset? What is the risk if you do not reset it? *(Tests: object lifecycle across bidirectional navigation; state ownership rules.)*

5. Apply the AI boundary test to this suggestion: "Use a shared static `Book` field in a utility class so every panel can access the current selection without passing references." Is this AI-appropriate scaffolding or a design decision that belongs to the engineer? Explain what the suggestion optimizes for and what it sacrifices. *(Tests: AI boundary rule; separation of concerns; shared-state tradeoffs.)*

**Synthesis**

6. Translate the library flow's state table — screen, objects read, objects modified, entry state, exit state — into your own domain (inventory or scheduling). For each screen, name the equivalent domain object and specify which fields are read-only and which are writeable. Identify any screen where your domain's flow differs structurally from the library flow and explain why. *(Tests: domain transfer; state ownership across domains; staging object pattern if applicable.)*

7. The three-layer responsibility model assigns visibility to `CardLayout`, display to panels, and state to domain objects. Describe a real implementation choice that would violate the separation between the display layer and the state layer — something Java would allow but the module's design constraint forbids. Explain what downstream effect that violation would produce when a new feature is added in a later module. *(Tests: separation of concerns; design constraint as forward error prevention.)*

**Challenge**

8. The staging object pattern (used in the scheduling domain) requires an `Appointment` object to be partially constructed across multiple screens. Design the ownership rules for a three-screen flow — Patient Selection, Provider Selection, Time Slot Selection — where each screen adds fields to the same `Appointment` object. Specify: which screen creates the object, which fields each screen is permitted to set, what the object's state looks like at the end of each transition, and what happens if the user skips the middle screen. *(Open-ended; anticipates error-handling patterns introduced in a later module.)*

---

- **user flow** — the path a user takes through an application, described not as screen sequence but as state transitions: which objects change, which fields are read, which fields are written, at each step.
- **screen** — in this module's context, a `JPanel` managed by `CardLayout`. A screen is a view into part of the application's object state. It does not own that state; it displays and optionally modifies it.
- **CardLayout** — a Swing layout manager that holds multiple panels in a container and shows exactly one at a time. The hidden panels remain in memory with their state intact. Navigation is a `show()` call on the layout, preceded by setter calls on the destination panel.
- **state** — the current values of an object's fields. In the context of a user flow, state is what the objects carry between screens. The screens display state; they do not replace it.
- **transition** — a navigation step in the flow: the moment when one screen becomes invisible and another becomes visible. A correct transition sets the destination panel's state before changing visibility.
- **separation of concerns** — the design principle that assigns each responsibility — visibility, display, state — to a distinct layer, so that changes in one layer do not require changes in another.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Exercise: Objects and Classes | Formative | For-Credit | Individual | Identify classes, objects, fields, and methods in a small design. |
| Exercise: Data Field | Formative | For-Credit | Individual | Practice instance data and access control. |
| Discussion - Constructors | Community | For-Credit | Individual | Explain how constructors initialize valid objects. |
| Graded Lab - Fundamentals of OOP | Formative | For-Credit | Individual | Build and test classes with fields, constructors, and methods. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and the Java SE API documentation: authoritative sources on `CardLayout`, `JPanel`, and layout management.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": research on the object-reference confusion this module addresses directly.
- Peng et al. and Vaithilingam et al.: empirical work on AI coding assistance; the phase gate structure in this module is grounded in their findings on verification risk.

*Current tool instructions, version-specific setup, and AI platform behavior require pre-offering verification.* [verify]
<!-- FACT-CHECK FLAG: UNVERIFIED — see factchecks/03-objects-and-classes-assertions.md -->

---

---

## References

<!-- Fact-check pass references. -->
1. Oracle. The java Command. Java SE 21 / JDK 21 Tool Guides, 2023. https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html
