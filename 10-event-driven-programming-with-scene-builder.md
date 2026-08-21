# Module 10: Event-Driven Programming with Scene Builder

*The books did not change. The view did. That distinction is the entire subject of this module.*

---


## EDGE Course Outline Alignment

**Module 10: Event-Driven Programming with Scene Builder**

### Learning Objectives (LOs)

- Analyze the role and implementation of events within JavaFX Scene Builder.
- Understand how JavaFX Scene Builder operates.
- Apply knowledge of JavaFX by incorporating a broader range of UI controls into practical software development projects, demonstrating enhanced skills in GUI creation and design.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Event-driven programming with Scene Builder
- Advanced GUI Programming

Here is the observation. You have been running the library program for nine modules. It prints to the console. Lines of text appear: book titles, patron names, checkout records. The model is working. The data is real. The behavior is correct.

Now the same catalog appears in a window. Rows and columns. A scrollbar. A search bar at the top. Click a row and the book's details appear in a panel on the right. Everything looks different.

But here is the question: what actually changed?

The `Book` objects are the same objects. The availability field is still a boolean. The patron's borrowed-books list is still a list. The checkout logic is still in the same method. None of that moved. What changed is how the state of those objects is presented — which fields are shown, in what layout, to what kind of user. The presentation layer arrived on top of the model layer. The two layers are different things.

This is the distinction the module is built on. And the reason it matters is not aesthetic. It is structural. When you blur the line between the view and the model — when display logic and business logic live in the same class, when a button's click handler modifies the book's state directly without going through the model — you build a system that is difficult to test, difficult to change, and easy to break. The GUI becomes load-bearing structure it was never designed to carry.

The module's job is to show you where the line is and how to hold it.

---

## What MVC Actually Is

Model-View-Controller is a name for an idea that is older than the name. The idea is simple: the thing that knows the truth about the business domain should be separate from the thing that displays it, and both should be separate from the thing that handles user input.

The model is where truth lives. In the library, the model is the collection of `Book` objects, the collection of `Patron` objects, and the methods that operate on them: checkout, return, reserve, search. The model does not know whether it is being displayed in a console or a GUI or a web browser. It does not know what color the table rows are. It knows about books and patrons and the rules governing them.

The view is where display lives. The `TableView<Book>` in JavaFX is a view. It knows how to render a collection of objects as rows in a table. It knows about column widths and fonts and scroll behavior. It does not know the checkout rule. It does not know what availability means to the business. It knows how to show things.

The controller is the bridge. It listens for user input — a button click, a search query typed, a row selected — and translates that input into model operations. The controller knows that when a user clicks "Check Out," the appropriate model method should be called with the appropriate arguments. The controller is the only layer that talks to both.

Three layers. Three responsibilities. The discipline is keeping each one from absorbing the other's work.

![MVC responsibility map rendered as a systems diagram for the chapter concept.](images/10-event-driven-programming-with-scene-builder-fig-01.png)
*Figure 10.1 — MVC responsibility map*


---

## The Scene Graph

Before writing a single event handler or model call, you need to understand how JavaFX represents what is on screen. It uses a structure called the scene graph.

The scene graph is a tree of nodes. Every visible element — a button, a label, a table, an image — is a node. Nodes are arranged in a parent-child hierarchy. A pane is a node that contains other nodes and determines how they are positioned. A control is a node the user can interact with. The root of the tree is the scene, and the scene lives inside a stage — JavaFX's word for a window.

When JavaFX renders your application, it walks this tree and draws each node. When a node's state changes — a label's text is updated, a button's disabled property changes, a table's row data changes — JavaFX re-renders that node. The tree is not just a description of what is on screen right now. It is a live structure that JavaFX monitors. Changes to node properties propagate through the rendering system automatically.

This matters for the model-view separation because it defines the contract between the view and the model. The view displays node properties. The model holds business state. The controller's job is to keep those two things synchronized: when the model changes, the relevant node properties must be updated; when the user acts on a node, the relevant model methods must be called.

The scene graph does not enforce this contract. JavaFX does not know or care whether your business logic is inside a controller, inside an event handler, or embedded directly in the construction of the scene. The discipline is yours to maintain.

![Scene graph tree rendered as a hierarchy / taxonomy for the chapter concept.](images/10-event-driven-programming-with-scene-builder-fig-02.png)
*Figure 10.2 — Scene graph tree*


---

## TableView and the Question It Forces

The `TableView` is the most important JavaFX control for the semester project, and it forces a question immediately: where does the data come from?

A `TableView<Book>` displays `Book` objects. Each column is defined by a `TableColumn<Book, String>` or similar, and each column has a cell value factory — a function that, given a `Book` object, returns the value to display in that column's cell. For the title column, the factory returns `book.getTitle()`. For the availability column, it returns `book.isAvailable() ? "Available" : "Checked Out"`.

Look at what this means. The `TableView` does not hold a copy of the book data. It holds a reference to the `Book` objects, and it calls the model's own methods to extract what it needs to display. The displayed value is computed from the model at render time, not stored in the table. If the model changes — if a book is checked out and its availability flips from true to false — the table re-renders with the new value the next time it draws that row.

This is the correct design. The table is a view onto the model. It is not a parallel copy of the model data. The moment you start copying model state into table cells manually — extracting strings and storing them in the table rather than letting the cell value factory do the extraction at render time — you have created two sources of truth. The model says one thing. The table says another. They will diverge.

The cell value factory is the seam between the view and the model. It is the one place in the table's architecture where the view reaches into the model. Everything else in the `TableView` — the layout, the sorting, the selection, the scrolling — is the view's business. The data is the model's business.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## The Search Bar Problem

The search bar is where most students first violate the model-view separation, and it is worth examining why.

The user types in the search bar. They want to see only the books matching their query. The easiest thing to do — the thing that feels obvious — is to filter the table's rows directly: remove the rows that do not match, keep the rows that do. The table now shows the filtered results.

The problem is that this approach uses the table as the authoritative list of books. When you remove rows from the table to filter, you have modified what the table considers to be the full catalog. When the user clears the search, you have to restore the removed rows — which means you were storing them somewhere else, which means you now have two lists of books: the one in the model and the one that was removed from the table.

The correct approach is to filter the model and let the view reflect the model. The controller receives the search query. It calls the model's search method — the same method that existed before there was a GUI, the method you wrote in Module 9. The model returns a filtered list. The controller updates the table's data to show that filtered list. When the search is cleared, the controller asks the model for the full catalog and updates the table again.

The model is the single source of truth. The view displays whatever the model provides. The controller mediates. The table never makes decisions about which books exist.

This is not a subtle distinction in theory. In practice, it is exactly the line between a system that is testable and a system that is not. You can test the model's search method without a GUI. You cannot test search behavior that lives inside an event handler embedded in the table construction. When the search breaks — and search always breaks on edge cases — you need to be able to isolate where the logic failed. If the logic is in the model, you can test it. If it is in the view, you cannot.

---

## The Event Handler and Where It Belongs

The controller's primary artifact is the event handler. A button's `setOnAction` method takes a handler — a function that runs when the button is clicked. That function is where the controller lives.

Here is what the handler should do: receive the event, extract the relevant input from the view (what the user typed, what row is selected, what state a checkbox is in), call the appropriate model method with that input, and update the view to reflect the model's new state. Four things. No more.

Here is what the handler should not do: compute business logic. Validate a patron's eligibility directly in the handler instead of calling the model's validation method. Calculate a due date in the handler instead of calling the model's date service. Update book state directly from the handler instead of calling `book.checkOut()`. Every time business logic migrates into an event handler, that logic becomes untestable, non-reusable, and invisible to the rest of the system.

The handler is a translator, not a decision-maker. It converts user intent into model operations and model results into view updates. The decisions belong in the model.

The practical discipline is simple to state. Look at every line in your event handler and ask: is this a display decision or a business decision? Display decisions belong in the handler. Business decisions belong in the model. If the handler is more than a dozen lines, suspect that it has absorbed model logic it should not hold.

| Item | Meaning |
| --- | --- |
| "Which color to show for an available book", "Whether a patron is eligible to borrow", "Updating the table after a checkout", "Calculating the due date", "Disabling the checkout button when no row is selected", "Enforcing the three-book limit" | A concrete checkpoint for applying the chapter concept. |
| student uses this as a checklist when auditing their own event handlers | A concrete checkpoint for applying the chapter concept. |

---

## The Panes and What They Are For

A pane in JavaFX is a node that positions its children. Different panes apply different layout strategies.

`VBox` stacks children vertically. `HBox` stacks them horizontally. `BorderPane` divides the container into five regions — top, bottom, left, right, center — each of which can hold a node. `GridPane` arranges children in rows and columns with explicit coordinates. `StackPane` places children on top of one another.

The choice of pane is a display decision. Which pane you use does not affect the model. A `Book` object does not know whether its title is displayed in a `VBox` or a `GridPane`. The layout is purely the view's concern.

This is worth stating because it is easy to treat pane selection as a coding task when it is actually a design task. The question is not "what are the valid arguments to `BorderPane`?" The question is "what layout serves the user's task?" A catalog view where the user needs to scan many items quickly suggests a table with sortable columns. A detail view where the user needs to see one item thoroughly suggests a form with labeled fields. The pane implements the layout decision; the layout decision comes from understanding the user's task.

For the semester project, the catalog view is a natural `BorderPane`: a search bar at the top, a `TableView` in the center, and a detail panel on the right or bottom for the selected item's full information. This layout serves a common pattern in business applications — browse a collection, select an item, inspect the details. The pane structure makes the pattern explicit in the scene graph.

---

## Keeping Display Out of the Model

There is a failure mode this module is specifically designed to prevent, and it is worth naming directly: model objects that know about the GUI.

It happens like this. The student is building the detail panel. They want to show a book's title in a specific color — green if available, red if checked out. The easiest place to add that logic seems to be the `Book` class: add a method `getTitleColor()` that returns the color string. The `Book` already knows its availability. Why not let it return the display representation of that state?

Because now the `Book` class knows about colors. It knows about the GUI. It has absorbed a display decision that belongs in the view. If the application later runs headlessly — in a batch process, in a web API, in a testing environment — the `Book` class carries GUI concepts that serve no purpose and introduce dependencies on JavaFX that belong nowhere near a domain model.

The rule is: model objects return domain state. View logic decides how to display that state. The `Book` returns `isAvailable()`, a boolean. The cell value factory or the event handler or the controller decides what color to render based on that boolean. The color decision lives in the view. The availability decision lives in the model.

This is the same principle that governs the search bar and the event handler — it is just applied at the class level rather than the method level. Model objects are pure domain objects. They know about business state and business rules. They do not know about screens, colors, fonts, or layouts.

---

## Where AI Fits Into This Picture

The AI boundary for this module reflects the structure of the three layers: AI may generate JavaFX layout and display boilerplate. It may not generate event handlers or model logic.

Layout boilerplate — the construction of a `BorderPane`, the definition of `TableColumn` objects, the scaffolding of a basic scene — is genuinely tedious to write from scratch, and AI is good at generating it correctly. The boilerplate is display logic. It does not touch the model. The risk of AI-generated layout code is low, provided the student understands what they received and can trace each displayed value back to a model field.

That tracing is the verification step, and it is the student's responsibility regardless of where the code came from. Every visible value in the GUI must be traceable to a model object and a model field. If a column displays a string that is not the result of calling a model method, the separation has already been violated — either the display logic computed a value that should have come from the model, or the model holds display concerns it should not. The trace exposes both.

Event handlers are outside the boundary because they are where the controller logic lives, and controller logic requires understanding the model: which method to call, what arguments to pass, what the model's return value means for the view. AI can generate an event handler that looks correct. It cannot verify that the handler calls the right model method for your specific business requirement, or that it handles the model's error states correctly. That requires understanding the model — which is exactly the understanding this course is building.

The pattern is the same as every previous module: AI for scaffolding, human for judgment. Layout is scaffolding. Event handling is judgment.

---

## The Module's Single Capability

Every module in this course adds exactly one concrete capability to the semester project. This module's capability is: a visual interface that displays real model data, where every visible value is traceable to a model object and no display logic has migrated into the model.

In practical terms: a JavaFX window opens. A `TableView` displays the catalog — actual `Book` objects from the model, not hardcoded strings. A search bar filters by calling the model's search method, not by removing rows from the table. A detail panel shows the selected book's fields.

The verification requirement is specific. You should be able to point to any value visible in the GUI and trace it: this string on screen came from this field on this object by way of this cell value factory. If you cannot trace it — if the value was computed in the view, stored in a local variable, copied from the model into a display string that lives outside the model — the separation has been violated somewhere, and the violation is evidence.

When you can do this for the library catalog, you can do it for the inventory table and the scheduling grid. The domain objects have different fields. The columns show different information. The layout serves different user tasks. But the structural move — view displays model state, controller mediates, model owns no display logic — is identical.

![Three-layer stack with controller seam rendered as a hierarchy / layered stack for the chapter concept.](images/10-event-driven-programming-with-scene-builder-fig-03.png)
*Figure 10.3 — Three-layer stack with controller seam*


---

## What You Are Building Toward

Nine modules built the model: objects, collections, relationships, transactions, persistence, search. This module adds the window onto that model. The user can now see the catalog, search it, select a book, and inspect its state — without the model knowing anything about the window.

The next question is: the GUI displays data, but how does the user make it respond? The search bar filters. The checkout button should check out a book. The return button should return one. These actions require event handlers that call model methods and update the view to reflect the new state. The handlers must deal with the possibility that the model rejects the action — the book is already checked out, the patron has hit their limit — and surface that failure to the user without crashing.

That is Module 11's subject. The event handling system and the feedback loop between model errors and view state. But the foundation for that work is what you built here: the clean separation between the model that enforces rules and the view that displays results. Without that separation, event handling becomes untestable and fragile. With it, each module builds cleanly on the last.

---

## Key Terms

- **JavaFX** — the Java framework for building graphical user interfaces; provides the scene graph, layout panes, controls, and event system used in this course's semester project.
- **scene graph** — the tree of visual nodes that JavaFX renders; every element on screen is a node, nodes are arranged in a parent-child hierarchy, and changes to node properties trigger re-rendering.
- **pane** — a JavaFX node that contains and positions child nodes according to a layout strategy; `VBox`, `HBox`, `BorderPane`, and `GridPane` are common examples, each implementing a different arrangement rule.
- **control** — a JavaFX node the user can interact with; buttons, text fields, checkboxes, and `TableView` are controls; they generate events when the user acts on them.
- **TableView** — a JavaFX control that displays a collection of objects as a table of rows and columns; columns are defined by cell value factories that extract display values from model objects at render time.
- **model** — the layer of the application that holds business state and business logic; in the semester project, the model includes domain objects (`Book`, `Patron`) and the methods that operate on them; the model does not know about the GUI.
- **view** — the layer that displays model state to the user; in JavaFX, the view is the scene graph — the panes, controls, and their layout; the view does not make business decisions.
- **controller** — the layer that mediates between view events and model operations; it receives user input from the view, calls the appropriate model methods, and updates the view to reflect the model's new state.

---

## Exercises

**Warm-up**

1. The following items are either model responsibilities, view responsibilities, or controller responsibilities. Classify each and write one sentence explaining your reasoning: (a) storing the list of `Book` objects; (b) rendering a `Book`'s availability as green or red text; (c) calling `patron.borrow(book)` when the checkout button is clicked; (d) enforcing the three-book borrowing limit; (e) updating the `TableView` after a successful checkout; (f) deciding what column headers appear in the table. *(Tests: applying the three-layer taxonomy before writing any JavaFX code.)*

2. The cell value factory for the availability column currently returns `book.isAvailable() ? "Available" : "Checked Out"`. A student proposes moving this logic into a `Book` method called `getAvailabilityLabel()` and having the factory call that instead. Evaluate the proposal: does it respect the model-view separation? What would you need to be true about `getAvailabilityLabel()` for this to be an acceptable design? What would make it a violation? *(Tests: applying the model-display boundary at the method level; distinguishing domain state from display representation.)*

3. Trace the following value from screen to model: the title string shown in the first row of a `TableView<Book>`. Write out each step — the control, the column, the cell value factory, the method call, and the field on the `Book` object. Then describe what would need to change in the model for the displayed value to change without touching any view code. *(Tests: executing the verification trace the module requires; seeing the cell value factory as the seam between view and model.)*

**Application**

4. Build the library catalog view: a `BorderPane` with a search `TextField` at the top, a `TableView<Book>` in the center displaying title, author, and availability, and a detail panel at the right or bottom showing the selected book's full information. The `TableView` must display real `Book` objects from the model. The search field must filter by calling the model's search method, not by removing rows from the table. Submit a screenshot and a trace of one displayed value back to its model field. *(Tests: implementing the full module capability — real model data, correct MVC structure, no rows deleted from the table on search.)*

5. Translate the catalog view to your chosen domain. In inventory: a table of products with name, quantity on hand, and status. In scheduling: a table of appointments with patient name, provider, time, and status. The domain objects are different; the structural move is the same. For each column, write the cell value factory in plain English before writing it in Java. *(Tests: transferring the view-onto-model pattern to a different domain; forcing articulation of the factory before implementation.)*

6. Write an AI prompt that complies with this module's boundary — it requests JavaFX layout boilerplate for the catalog view, without asking AI to write event handlers or model calls. Run the prompt. Then audit the output: for each column, trace the displayed value to a model method. Mark any column whose value is not traceable. Revise any violation. *(Tests: applying the AI boundary for layout vs. logic; executing the verification trace on AI-generated code.)*

**Synthesis**

7. A classmate implements the search bar by removing non-matching rows from the `TableView`'s items list and storing the removed rows in a local variable. The search appears to work correctly on first use. Write a two-paragraph response: the first paragraph describes a scenario where the implementation produces wrong behavior (hint: consider what happens when the user searches twice without clearing, or when a book is added to the catalog while filtered results are showing); the second paragraph describes the correct implementation and explains why it avoids the failure. *(Tests: identifying the two-sources-of-truth failure mode; articulating the correct design.)*

8. You need to show available books in green and overdue books in red. You have three options: (a) add a `getColor()` method to `Book`; (b) compute the color in the cell value factory based on `book.isAvailable()` and `book.isOverdue()`; (c) add a custom `TableCell` subclass that reads the model's availability and due-date state and sets its own text fill. Evaluate each option against the model-view separation principle. Which is acceptable? Which is a violation? For the acceptable options, is there a reason to prefer one over the other? *(Tests: applying the model-display boundary across three concrete design choices; reasoning about where display logic legitimately lives within the view layer.)*

**Challenge**

9. The bridge question asks: the GUI displays data — how does the user make it respond? Before reading Module 11, sketch your best answer for one action: the checkout button. What event handler logic would you write? What model method would you call? What would you do if the model rejects the checkout — because the book is unavailable, or the patron is at their limit? How would you surface that failure to the user without crashing? Write your answer as pseudocode or a numbered sequence of steps. The goal is to make your current thinking explicit so you can compare it to what Module 11 actually teaches about error feedback and view state. *(Points toward model error handling, view state management, and the feedback loop between model exceptions and user-facing messages.)*

---

## Bridge

The GUI displays data. How does the user make it respond?

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Discussion - Event-Driven Programming with Scene Builder | Community | For-Credit | Individual | Explain how Scene Builder events connect to controller code. |
| Optional Exercise: Creating a Java Application With an Interaction | Practice | For-Credit | Individual | Create an interactive JavaFX application using Scene Builder concepts. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation — use for language and library facts.
- OpenJFX documentation — use for JavaFX layout panes, controls, `TableView`, cell value factories, and the event system; this is the authoritative source for JavaFX specifics.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming" — use for common novice difficulties, including the conflation of display and business logic.

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
1. OpenJFX. Class FXMLLoader and Annotation FXML. JavaFX 21 API Specification, 2023. https://openjfx.io/javadoc/21/javafx.fxml/javafx/fxml/FXMLLoader.html
