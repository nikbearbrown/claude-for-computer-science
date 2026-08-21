# Module 11: Generics

*The button worked. Nobody knew why. Nobody knew where.*

---


## EDGE Course Outline Alignment

**Module 11: Generics**

### Learning Objectives (LOs)

- Explore the application of generics to create classes, interfaces, and methods with a single definition that can be utilized with different data types.
- Describe the benefits of using generics.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Generics
- Wildcards
- Overview of Generics
- Declaring Generic Classes and Interfaces

Here is a button handler. It is real code from a real student project:

```java
checkoutButton.setOnAction(e -> {
    String selectedIsbn = catalogListView.getSelectionModel()
                                         .getSelectedItem();
    if (selectedIsbn == null) {
        statusLabel.setText("No book selected.");
        return;
    }
    Book book = catalog.findByIsbn(selectedIsbn);
    if (book == null || !book.isAvailable()) {
        statusLabel.setText("Book not available.");
        return;
    }
    book.setAvailable(false);
    currentPatron.addCheckout(book);
    catalogListView.getItems().remove(selectedIsbn);
    checkedOutListView.getItems().add(selectedIsbn);
    statusLabel.setText("Checked out: " + book.getTitle());
    try {
        catalog.saveToFile("catalog.csv");
    } catch (IOException ex) {
        statusLabel.setText("Save failed: " + ex.getMessage());
    }
});
```

This handler validates input, queries the model, updates the model, updates two view controls, sets a status message, writes to disk, and handles a file I/O exception. It runs. It passes a first test. It will cause problems the moment anything in this list changes — and things will change.

The problem is not that the handler is long. Long code can be well-organized. The problem is that the handler is a closet. When you did not know where to put something, you put it here. Now the checkout behavior lives in the event handler, the model lives in the event handler, the view update lives in the event handler, and the persistence lives in the event handler. Change any one of those things and you are editing the button handler. Add a second checkout path — a keyboard shortcut, say, or a programmatic checkout — and you write the same logic again, because it is embedded in the button.

That is the moment this module is for.

---

## How Event-Driven Programs Actually Work

Before JavaFX, every program you wrote ran top to bottom. Main called a method. The method ran. Something was printed. The program ended, or looped back and ran again. The program controlled the sequence.

Event-driven programs invert this. The program starts, builds an interface, and then waits. The user does something — clicks a button, types in a field, selects an item — and the system calls code you registered in advance to handle that action. You do not call the handler. The framework calls it for you, when the event occurs.

This is a fundamental shift in how you think about program structure. In a sequential program, the flow is in your control. In an event-driven program, the flow is in the user's control, and your job is to register responses to the actions you care about and do nothing about the ones you don't.

The three-part structure is always the same:

**The event source** — a UI control that can produce events. A `Button`, a `ListView`, a `TextField`. The source does not know what will happen when it fires. It just fires.

**The event** — the object that carries information about what happened. For a button click, it is an `ActionEvent`. For a key press, it is a `KeyEvent`. The event knows which source fired, when, and what the user did.

**The handler** — the code that runs when the event fires. You register it on the source in advance. When the event occurs, the framework finds the registered handler and calls it.

In JavaFX, registration looks like this:

```java
checkoutButton.setOnAction(handler);
```

The handler can be anything that implements `EventHandler<ActionEvent>` — which means anything with a single method `handle(ActionEvent event)`. Java gives you three ways to supply that:

An anonymous inner class, which is the explicit form:

```java
checkoutButton.setOnAction(new EventHandler<ActionEvent>() {
    @Override
    public void handle(ActionEvent event) {
        // handler body
    }
});
```

A named inner class, which is useful when the same handler is reused:

```java
class CheckoutHandler implements EventHandler<ActionEvent> {
    @Override
    public void handle(ActionEvent event) {
        // handler body
    }
}
checkoutButton.setOnAction(new CheckoutHandler());
```

A lambda expression, which is the shortest form and the one you will use most:

```java
checkoutButton.setOnAction(event -> {
    // handler body
});
```

The lambda is syntactic shorthand for the anonymous inner class. It works because `EventHandler<ActionEvent>` is a functional interface — it has exactly one abstract method. Java can infer what the lambda is implementing. When you read `event -> { ... }`, read it as "when this event fires, do this."

All three forms produce the same result. The lambda form is clearest for simple handlers. The named inner class is clearest when the handler is complex enough to deserve its own class, its own name, and its own tests.

![Event structure + three registration forms rendered as a annotated example for the chapter concept.](images/11-generics-fig-01.png)
*Figure 11.1 — Event structure + three registration forms*


---

## What a Handler Should Do

The handler at the top of this module does too much. Not because there are too many lines — because there are too many kinds of things.

Here is a cleaner version of the same behavior:

```java
checkoutButton.setOnAction(event -> {
    String isbn = getSelectedIsbn();
    if (isbn == null) return;
    boolean success = controller.performCheckout(isbn);
    refreshView(success);
});
```

Four lines. Each line calls a method that owns one responsibility:

`getSelectedIsbn()` reads the current selection from the ListView and returns the ISBN string, or null if nothing is selected. It knows about the view. It does not know about the model.

`controller.performCheckout(isbn)` finds the book, checks availability, updates the book's status, records the checkout for the patron. It knows about the model. It does not know about the view.

`refreshView(success)` updates the ListView, the checked-out list, and the status label based on the result. It knows about the view. It does not know how the checkout was performed.

The handler itself knows about the sequence: get the selection, attempt the operation, reflect the result. It does not know the details of any of those three steps.

This is not decoration. It is what makes the behavior traceable. When a bug is reported — the checkout succeeds but the ListView doesn't update — you know the bug is in `refreshView`, not somewhere in the middle of a 40-line handler. When you want to add a keyboard shortcut that also triggers checkout, you call `controller.performCheckout(isbn)` directly without duplicating the logic. When you want to test the checkout logic in isolation, you call `controller.performCheckout(isbn)` in a test without needing a button, a ListView, or a running application.

The handler is the translator between the event and the system. Its job is to call the right methods in the right order. The methods it calls do the actual work.

![Handler responsibility decomposition rendered as a systems diagram for the chapter concept.](images/11-generics-fig-02.png)
*Figure 11.2 — Handler responsibility decomposition*


![Closet vs. clean handler code comparison rendered as a annotated example (paired) for the chapter concept.](images/11-generics-fig-03.png)
*Figure 11.3 — Closet vs. clean handler code comparison*


---

## Tracing the Full Chain

The Feynman test for event handling: can you trace a single user action — one click — from the button through the model to the updated view, naming the Java artifact at each step?

The user clicks the Checkout button. JavaFX detects the click and creates an `ActionEvent` object.

JavaFX looks up the handler registered on `checkoutButton` via `setOnAction`. It finds the lambda.

The lambda calls `getSelectedIsbn()`. This method asks the `catalogListView` for its currently selected item. It returns `"978-0-131-87248-6"` or `null`.

If null, the lambda returns. The click is consumed. Nothing changes in the model.

If not null, the lambda calls `controller.performCheckout("978-0-131-87248-6")`. The controller finds the book in the catalog, confirms it is available, sets `available = false`, adds it to the patron's checkout list. It returns `true`.

The lambda calls `refreshView(true)`. This method removes the ISBN from `catalogListView.getItems()`, adds it to `checkedOutListView.getItems()`, and sets the status label to a success message.

The event is fully processed. The model has changed. The view reflects the model. The handler is finished.

Each step is a specific Java artifact: the `ActionEvent`, the lambda, `getSelectedIsbn()`, `controller.performCheckout()`, `refreshView()`. If the view does not update, the bug is in `refreshView()`. If the model does not update, the bug is in `controller.performCheckout()`. If the wrong book is selected, the bug is in `getSelectedIsbn()`. The trace makes bugs locatable.

![Click-to-view chain with bug-location callouts rendered as a process flowchart for the chapter concept.](images/11-generics-fig-04.png)
*Figure 11.4 — Click-to-view chain with bug-location callouts*


---

## The Responsibility Rule

There is a simpler way to state what the handler should and should not contain: each method should have one reason to change.

A method that validates input, updates the model, and refreshes the view has three reasons to change. If the validation logic changes, you edit the method. If the model's checkout API changes, you edit the same method. If the view gains a new control that needs refreshing, you edit the same method again. Each change risks breaking the other behaviors, because they all live together.

A method that only validates input changes only when the validation logic changes. A method that only updates the model changes only when the model's checkout API changes. A method that only refreshes the view changes only when the view changes. They are independent. They can be read, tested, and debugged independently.

This is not a theoretical principle. It is a practical observation about how software breaks. Code breaks at change boundaries. If you put three things in one method, you get three opportunities to break in one place. If you put three things in three methods, you get three independent, locatable, testable units.

Apply this test to the handler you wrote: count the number of reasons it could need to change. If the answer is more than one, the handler has too many responsibilities.

| code element | what it does | which layer owns it | reason it would change |
| --- | --- | --- | --- |
| Core idea | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| Practical use | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| Common failure | The pattern becomes easy to misuse or overlook. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Inner Classes, Lambdas, and When to Use Each

The choice between a lambda and a named inner class is not purely syntactic. It is a design decision about how complex the handler is and whether it needs to be understood, tested, or reused independently.

A lambda is appropriate when:

- The handler body fits in a few lines.
- The handler delegates immediately to named methods.
- The handler is used in exactly one place.
- Reading the lambda inline makes the registration code clearer.

```java
searchButton.setOnAction(e -> performSearch());
addButton.setOnAction(e -> showAddDialog());
clearButton.setOnAction(e -> clearForm());
```

These are clear. Each lambda calls one method. The registration and the delegation are visible together. There is no reason to give these handlers their own classes.

A named inner class is appropriate when:

- The handler has enough logic to deserve a name.
- The handler is reused across multiple sources.
- The handler needs to be tested independently.
- The handler's behavior is complex enough that it benefits from being read in isolation.

```java
class CheckoutHandler implements EventHandler<ActionEvent> {
    private final Catalog catalog;
    private final PatronSession session;
    private final CatalogView view;

    CheckoutHandler(Catalog catalog, PatronSession session, CatalogView view) {
        this.catalog = catalog;
        this.session = session;
        this.view = view;
    }

    @Override
    public void handle(ActionEvent event) {
        String isbn = view.getSelectedIsbn();
        if (isbn == null) {
            view.showError("No book selected.");
            return;
        }
        boolean success = session.checkout(catalog, isbn);
        view.refresh(success);
    }
}
```

This handler has a name, a constructor, dependencies, and a method body that you can read without context. You can write a test for it by constructing it with mock dependencies. You can register it on multiple buttons if needed. You can change it without touching the code that registers it.

The choice is not lambda versus inner class as a style preference. It is "does this handler deserve its own class?" If yes, give it one. If no, a lambda is cleaner.

| criterion | lambda | named inner class |
| --- | --- | --- |
| handler body length (few lines vs complex | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| reuse (one place vs multiple sources | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| testability (not independently tested vs independently testable | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| dependency clarity (captures from enclosing scope vs explicit constructor params | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| readability (inline with registration vs readable in isolation) — | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## The Model-View Boundary Under Pressure

Here is something that will happen on a deadline: you need to add a feature quickly, and the fastest path is to put the code in the button handler. The handler already has access to the model, the view, and the file system. One method call in the handler body, and the feature works.

This is how the closet fills up. Each addition was fast. The cumulative result is a handler that no one can reason about, because every behavior lives inside it and the behaviors are entangled.

The discipline is not complicated. It just requires a decision at the moment of fastest temptation:

Before writing a line in a handler body, ask: whose responsibility is this?

If the answer is "the model's," write it as a method on the model and call that method from the handler.

If the answer is "the view's," write it as a method on the view and call that method from the handler.

If the answer is "the handler's," which means it is genuinely about translating the event into a sequence of calls, write it in the handler.

The handler should contain coordination. Everything else should be delegated.

Here is the checkout handler after a round of deadline additions that violated this rule:

```java
checkoutButton.setOnAction(event -> {
    String isbn = catalogListView.getSelectionModel().getSelectedItem();
    if (isbn == null) { statusLabel.setText("Nothing selected."); return; }
    Book book = catalog.findByIsbn(isbn);
    if (book == null) { statusLabel.setText("Not found."); return; }
    if (!book.isAvailable()) { statusLabel.setText("Not available."); return; }
    book.setAvailable(false);
    patron.addCheckout(book);
    dueDate = LocalDate.now().plusWeeks(3);  // new feature: due dates
    book.setDueDate(dueDate);
    catalogListView.getItems().remove(isbn);
    checkedOutListView.getItems().add(isbn + " (due " + dueDate + ")");
    statusLabel.setText("Checked out. Due: " + dueDate);
    try { catalog.saveToFile("catalog.csv"); }
    catch (IOException ex) { statusLabel.setText("Save failed."); }
    if (patron.getCheckoutCount() >= 5) {
        warningLabel.setVisible(true);  // new feature: checkout limit warning
        warningLabel.setText("Checkout limit approaching.");
    }
});
```

Two features were added directly to the handler. Both work. Now the handler also sets due dates, formats due date strings for the view, checks the patron's checkout count, and controls label visibility. None of those belong to the handler. The handler's responsibility was to translate the click into a sequence of calls. It now contains business logic, view formatting logic, and persistence logic.

The refactored version:

```java
checkoutButton.setOnAction(event -> {
    String isbn = getSelectedIsbn();
    if (isbn == null) return;
    CheckoutResult result = controller.performCheckout(isbn);
    refreshView(result);
});
```

`controller.performCheckout(isbn)` now handles due dates, availability checks, patron record updates, and saving. It returns a `CheckoutResult` object that carries the outcome — success or failure, due date, whether the limit warning applies. `refreshView(result)` reads the result and updates every view element that needs updating. The handler sees neither the business logic nor the view logic. It sees a sequence.

The discipline holds whether or not you are on a deadline. The handler on a deadline is exactly the same size as the handler without deadline pressure. The complexity was moved into `controller.performCheckout()` and `refreshView()`, where it belongs.

![Before/after deadline refactor rendered as a comparison panels for the chapter concept.](images/11-generics-fig-05.png)
*Figure 11.5 — Before/after deadline refactor*


---

## The Java Artifacts That Carry This

For event handling, the artifacts are:

**The event source with registered handler** — the UI control with `setOnAction` or the equivalent registration call:

```java
checkoutButton.setOnAction(event -> handleCheckout());
```

**The handler body** — the lambda or inner class that translates the event into a sequence of method calls:

```java
private void handleCheckout() {
    String isbn = getSelectedIsbn();
    if (isbn == null) return;
    CheckoutResult result = controller.performCheckout(isbn);
    refreshView(result);
}
```

**The delegated methods** — the model methods and view methods that the handler calls, each owning one responsibility:

```java
// Model method — knows nothing about the view
public CheckoutResult performCheckout(String isbn) { ... }

// View method — knows nothing about business logic
private void refreshView(CheckoutResult result) { ... }
```

**The responsibility boundary** — not a line of code, but a verifiable property: the handler contains only coordination, the model methods contain only business logic, and the view methods contain only display logic. Verified by reading: can you change the checkout business logic without touching the handler? Can you change the view layout without touching the controller?

Four artifacts. The registration, the handler body, the delegated methods, and the boundary between them. If the boundary is maintained, the chain is traceable. If the boundary is violated, the trace will eventually break.

| artifact | what it is | the specific code pattern | the failure mode if it is missing or violated |
| --- | --- | --- | --- |
| event source with registration (setOnAction call | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| handler body (lambda or inner class with only coordination | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| delegated model method (contains business logic, no view references | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| delegated view method (contains display logic, no business logic) | student uses this as a self-check before submitting the lab | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |

---

## The AI Boundary

AI may generate event registration boilerplate and method-call skeletons. It may not fill business logic bodies.

The reason is the same reason the AI boundary exists in every module, expressed here in a new form. JavaFX event registration syntax is the same for every button in every application. AI knows it. The registration call, the lambda form, the inner class form — these are generic. AI can generate them correctly without knowing anything about your project.

The business logic inside the handler is not generic. What `performCheckout()` must do depends on your model design, your data structures, your validation rules, and your instructor's requirements. AI will generate plausible checkout logic that may not match your model's API, may not validate what your requirements require, and may not handle the edge cases your domain involves.

The distinction the phase gate enforces: write the action sequence in plain English before generating a handler.

For the checkout button: "When the user clicks Checkout, get the selected ISBN from the catalog list. If nothing is selected, show an error and stop. Find the book in the catalog. If the book is not available, show an error and stop. Mark the book as unavailable. Record the checkout for the current patron. Update the catalog list to remove the book. Update the checked-out list to add the book with its due date. Show a success message."

That sequence is the specification. Give it to AI and ask for a handler structure — the registration call, the method names, the sequence of calls. Do not ask AI to fill in `performCheckout()` or `refreshView()`. Those are yours, because they depend on the model you built in earlier modules and the view you built in Module 10.

The skeleton AI generates is a scaffold. Your job is to fill the business logic bodies, verify that the sequence matches your specification, and confirm that the handler does not contain logic that belongs in the model or view.

![AI scaffolds / you fill bodies rendered as a comparison panels for the chapter concept.](images/11-generics-fig-06.png)
*Figure 11.6 — AI scaffolds / you fill bodies*


---

## What Event-Driven Design Reveals

Return to the opening handler. The one that validates, updates, formats, refreshes, and writes to disk in a single lambda body.

That handler had behavior. It had a sequence of steps that, when traced, produced the right output. What it lacked was a boundary: a decision about which step belonged to which layer, and a commitment to put each step where it belonged.

The event-driven model teaches that boundary by making it consequential. When the boundary holds, you can change the checkout logic without touching the view. You can add a keyboard shortcut that triggers checkout without duplicating the logic. You can read the handler and understand the sequence without reading the model or the view.

When the boundary breaks — when the handler becomes a closet — the sequence is still there, but it is no longer findable. "Where does the checkout update the catalog?" The answer is somewhere in the handler, between the validation and the persistence call, but to find it you have to read the whole thing, because the whole thing is one undivided block.

The discipline the module teaches is not a style rule. It is the minimum required to keep the trace intact. Click → handler → model update → view refresh. Each step visible. Each step independent. Each step testable.

When you add a feature under deadline pressure and it goes in the handler because the handler is already there and already has access to everything, you have broken the trace for everyone who comes after you — including yourself, two weeks later, when the feature needs to change and you cannot find where it lives.

The handler is the translator. Not the application. Not the model. Not the persistence layer. The translator. Its job is to hand off quickly to the things that know how to do the work, and then to reflect the result in the view.

When it does that, and only that, the application responds. Not just to this click, but to every change that follows.

The bridge question is this: handlers work. How do you build more complex interfaces without tangling layout and logic?

---

## Exercises

**Warm-up**

1. Register a handler on a single button in your project using all three forms: anonymous inner class, named inner class, and lambda. Confirm all three produce the same behavior. Then choose the lambda form and delete the other two. Write one sentence explaining why the lambda is appropriate for this specific handler. *(Tests: event registration syntax, form equivalence, design choice reasoning.)*

2. Take the opening handler from this module — the one with seven responsibilities — and count the number of reasons it could need to change. List each reason as one sentence. Then identify which of the three layers (handler, model, view) should own each responsibility. *(Tests: responsibility analysis, layer identification.)*

**Application**

3. Write the action sequence for your Checkout (or equivalent primary action) in plain English before writing any code: each step on a new line, each step named as a responsibility. Then implement one handler method and the delegated model and view methods it calls. The handler body should contain only coordination: `getSelectedItem()`, one controller call, one view refresh. *(Tests: sequence-before-code discipline, responsibility decomposition, handler delegation.)*

4. Register handlers for three distinct user actions in your project. For each handler, write the registration call, the handler body, and the delegated methods. Verify the responsibility boundary: the handler contains no business logic, the model method contains no view logic, the view method contains no business logic. *(Tests: full handler chain, three-layer separation, boundary verification.)*

**Synthesis**

5. Add a new feature to an existing handler by placing the code where it belongs — not in the handler body, but in the appropriate model or view method. Trace the full chain before and after adding the feature: does the handler body change? Why or why not? Write the answer in prose. *(Tests: boundary discipline under extension, trace maintenance, separation under change.)*

6. Take a handler in your project that violates the responsibility rule — has more than one reason to change — and refactor it. Extract each responsibility into a named method. Show the before and after. Write one sentence for each extracted method explaining what it owns and why it belongs where you put it. *(Tests: refactoring discipline, method naming, responsibility ownership.)*

**Challenge**

7. Write a named inner class handler for a complex user action — one where the handler needs its own state, multiple dependencies, or reuse across more than one source. Give the handler a constructor that takes its dependencies explicitly. Explain why the named inner class is more appropriate than a lambda for this specific case, and what you gain from making the dependencies explicit in the constructor rather than capturing them from the enclosing scope. *(Tests: named inner class design, dependency injection into handlers, trade-off reasoning.)*

---

## Key Terms

- **event:** A notification object created by a UI control when a user action occurs, carrying information about the source, the type of action, and when it happened. Define this term in the context of your semester project before using it in lab work.
- **handler:** The code registered to execute when a specific event fires on a specific source. Define this term in the context of your semester project before using it in lab work.
- **registration:** The act of connecting a handler to an event source using `setOnAction` or equivalent, so the framework knows which code to call when the event occurs. Define this term in the context of your semester project before using it in lab work.
- **lambda:** A concise syntax for supplying a single-method interface implementation inline, used here as a shorthand for `EventHandler<ActionEvent>`. Define this term in the context of your semester project before using it in lab work.
- **inner class:** A class defined inside another class, used here to encapsulate handler logic that is too complex for a lambda or needs independent testability. Define this term in the context of your semester project before using it in lab work.
- **responsibility:** The single reason a method or class should need to change — the one concern it owns. Define this term in the context of your semester project before using it in lab work.
- **model update:** The step in the handler chain where the model's state is changed to reflect the user's action, performed by a model method called from the handler. Define this term in the context of your semester project before using it in lab work.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Optional Exercise: Generics | Practice | For-Credit | Individual | Write and use a generic class or interface. |
| Optional Exercise: Generic Methods | Practice | For-Credit | Individual | Write a method parameterized by type. |
| Optional Exercise: Generic Array | Practice | For-Credit | Individual | Reason about generic arrays and Java type restrictions. |
| Quiz - Generics | Summative | For-Credit | Individual | Assess generic types, generic methods, and wildcards. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation: authoritative source for lambda syntax, functional interfaces, and the event model.
- OpenJFX documentation: authoritative source for JavaFX event handling, `EventHandler`, `setOnAction`, and the full event type hierarchy.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": on common novice difficulties and what instruction actually changes.

---

## References

<!-- Fact-check pass references. -->
1. OpenJFX. Interface EventHandler. JavaFX 21 API Specification, 2023. https://openjfx.io/javadoc/21/javafx.base/javafx/event/EventHandler.html
2. Oracle. Lambda Expressions. Java Language Specification, Java SE 21, 2023. https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.27
