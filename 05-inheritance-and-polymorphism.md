# Module 5: Inheritance and Polymorphism

*The catalog was there before you walked in.*

---


## EDGE Course Outline Alignment

**Module 5: Inheritance and Polymorphism**

### Learning Objectives (LOs)

- Understand the definitions of inheritance and polymorphism.
- Distinguish between a subclass and a superclass through inheritance.
- Apply object-oriented concepts, such as inheritance, polymorphism, dynamic binding, casting, and ArrayLists, in practical scenarios.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Superclasses
- Subclasses
- The super keyword
- Inheritance
- Constructor Chaining
- Overriding vs. Overloading
- The Object Class
- Polymorphism
- Dynamic Binding
- Casting Objects
- The instanceof operator
- The equals method
- The protected modifier
- Visibility Modifiers
- ArrayList

Here is a design mistake you can make without realizing you made it.

You are building a library system. You have a patron. The patron wants to check out a book. So you write a checkout method, and inside the checkout method you figure out what books are available, and the whole thing works — it runs, it compiles, it produces output. You move on.

What you have just done is let the transaction create the resource. The checkout knows what books exist. Without a checkout, there are no books. The library catalog only exists in the moment someone is trying to use it.

That is backwards. A real library catalog exists on Tuesday morning before the building opens. It exists when no patron is logged in, when no transaction is running, when no screen is showing. The catalog is a durable fact about the world. The checkout is a temporary event that touches that fact. If your design cannot tell the difference, you have not modeled the supply side. You have hidden it inside the demand side, and it will cause you problems the moment you need to search the catalog without checking anything out, or preload resources at startup, or support two simultaneous transactions without one overwriting the other's assumptions about what exists.

That is the moment this module is for.

---

## Two Kinds of Objects

Not all objects are the same kind of thing.

Some objects represent resources that exist independently of any user action. A book in a library. A product in a warehouse. A physician's available appointment slot. These objects exist before the session starts and after it ends. Their state can change — a book can be checked out, a slot can be booked — but the object itself is durable. It is part of the model of the world, not a record of something a user did.

Other objects represent events: a checkout transaction, a purchase order, a scheduled appointment. These objects exist because a user did something. They record that a resource changed state, who requested it, when, under what terms. They depend on the resource objects — a checkout without a book is not a transaction, it is an error.

There is a third kind: objects that represent relationships between resources. An author who writes multiple books. A product category that contains multiple products. A physician who works at multiple clinics. These are not transactions and not pure resources — they are structural facts about how the supply side is organized.

The vocabulary for these three kinds is entity, relationship, and transaction. Entities are the durable resources. Relationships are the structural connections between entities. Transactions are the events that use entities.

Supply-side modeling is the work of getting the entities and relationships right before you write a single transaction.


---

## Why Order Matters

The failure mode at the start of this module — the catalog that only exists inside a checkout — is not a rare mistake. It is the natural consequence of building a system by starting from what the user does.

If you design by following the user's path — the patron arrives, the patron searches, the patron checks out — you will build objects that match the user's path. Search lives inside Patron. Checkout creates Book objects on demand. The whole supply side is a side effect of user actions, not a precondition for them.

This design can run for a long time without visibly breaking. A single patron, a single session, a simple test case: it works. The failure appears when:

- You need to preload the catalog at application startup, before any patron exists.
- You need to search the catalog from two different places in the code — once from a patron's perspective, once from a librarian's — and the search logic is buried inside one of those flows.
- You need to change a book's availability status and have that change reflected across every place the book appears.

At each of these moments, you discover that the entity does not really exist in your design. The data is there, but it is scattered across transaction objects, or recreated on demand, or duplicated across methods. The supply side was never modeled. It was just implied.

![Supply-coupled vs. preloaded rendered as a comparison panels for the chapter concept.](images/05-inheritance-and-polymorphism-fig-01.png)
*Figure 5.2 — Supply-coupled vs. preloaded*


The fix is to model it first, explicitly, as its own layer — before writing any transaction code.

---

## The Catalog as a First-Class Object

Take the library example. The supply side has two obvious entities: `Book` and `Catalog`. A `Book` knows its title, author, ISBN, availability status. A `Catalog` holds a collection of books and can answer questions: is this book available, what books match this search term, how many copies of this title exist.

Notice what the `Catalog` does not know. It does not know which patron is logged in. It does not know whether a transaction is in progress. It does not have a `checkout()` method. It does not know what screen the user is looking at. All of that is the demand side — the transaction layer — and it belongs somewhere else.

The `Catalog` is a durable resource. Here is what that looks like in code:

```java
public class Catalog {
    private Book[] books;
    private int count;

    public Catalog(int capacity) {
        books = new Book[capacity];
        count = 0;
    }

    public void addBook(Book b) {
        books[count] = b;
        count++;
    }

    public Book findByIsbn(String isbn) {
        for (int i = 0; i < count; i++) {
            if (books[i].getIsbn().equals(isbn)) {
                return books[i];
            }
        }
        return null;
    }

    public Book[] searchByTitle(String keyword) {
        Book[] results = new Book[count];
        int found = 0;
        for (int i = 0; i < count; i++) {
            if (books[i].getTitle().toLowerCase()
                        .contains(keyword.toLowerCase())) {
                results[found] = books[i];
                found++;
            }
        }
        // trim to actual size
        Book[] trimmed = new Book[found];
        for (int i = 0; i < found; i++) trimmed[i] = results[i];
        return trimmed;
    }
}
```

This class exists before any patron is created. In a main method or an initialization block, you build the catalog and populate it:

```java
Catalog catalog = new Catalog(100);
catalog.addBook(new Book("The Design of Everyday Things",
                         "Don Norman", "978-0-465-06710-7", true));
catalog.addBook(new Book("A Pattern Language",
                         "Christopher Alexander", "978-0-195-01919-3", true));
catalog.addBook(new Book("The Timeless Way of Building",
                         "Christopher Alexander", "978-0-195-02402-9", false));
```

The catalog now exists. No patron was needed to create it. Any transaction that later needs to find a book will look here — not construct its own view of what books exist.

![Catalog class boundary rendered as a structural schematic for the chapter concept.](images/05-inheritance-and-polymorphism-fig-02.png)
*Figure 5.3 — Catalog class boundary*


---

## Relationships Between Entities

A real supply side rarely has just entities. It has relationships: structural connections between entities that exist independent of any transaction.

In the library domain, the interesting relationship is between authors and books. An author can write multiple books. A book can have multiple authors. This is a many-to-many relationship, and your design needs to decide how to represent it.

The simplest representation: store an array of author names inside each `Book`. This works until you need to search by author across the whole catalog, or display an author's complete bibliography, or update an author's name in one place. At that point, names-in-a-book is not a relationship — it is duplicated data.

The cleaner representation: an `Author` entity, with its own identity, that a `Book` references.

```java
public class Author {
    private String name;
    private String biography;

    public Author(String name, String biography) {
        this.name = name;
        this.biography = biography;
    }

    public String getName() { return name; }
}
```

```java
public class Book {
    private String title;
    private Author[] authors;
    private String isbn;
    private boolean available;

    public Book(String title, Author[] authors,
                String isbn, boolean available) {
        this.title = title;
        this.authors = authors;
        this.isbn = isbn;
        this.available = available;
    }
    // getters omitted for space
}
```

Now the author exists as an entity. You can update the biography in one place. You can find all books by a given author by scanning the catalog for books whose author array contains that author reference. The relationship is modeled, not implied.

![Author shared across Books rendered as a structural schematic for the chapter concept.](images/05-inheritance-and-polymorphism-fig-03.png)
*Figure 5.4 — Author shared across Books*


| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## The Same Design Shape Across Domains

The supply-side pattern generalizes. Every business system has resources that exist before users arrive, and the modeling job is always the same: identify the entities, identify the relationships, implement them as objects that can be preloaded and queried.

In an inventory system, the entities are products, suppliers, and warehouses. The relationships are which supplier provides which products, which warehouse stocks which products. A `ProductCatalog` is structurally identical to a library `Catalog` — it holds products, it answers queries, it has no idea which customer is currently browsing. A `SupplierRelationship` object records that Supplier A provides Product B at a given price and lead time. That relationship exists independent of any purchase order.

In a healthcare scheduling system, the entities are providers, rooms, and service types. The relationships are which providers are qualified to deliver which service types, which rooms are equipped for which services. A `ProviderSchedule` is the supply-side model of time slots — it says what is available, without recording who has booked it. The booking is a transaction. The schedule is an entity.

The pattern is always: entity, relationship, transaction. Model them in that order. Build the supply side before the demand side touches it.

| domain | entity classes | relationship objects | transaction classes |
| --- | --- | --- | --- |
| library (Book | Author | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| inventory (Product | Supplier | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| healthcare scheduling (Provider | Room | ServiceType, ProviderQualification, Booking) | student should see the structural identity across domains and recognize the pattern they are learning is not library-specific |

---

## The Java Artifact That Carries This

Abstract patterns are useful. The book's standard is higher: you should be able to point to a specific line of code that carries the concept.

For supply-side modeling, the artifacts are:

**The entity class** — an object that represents a durable resource, initialized before any transaction runs:

```java
public class Book {
    private String title;
    private String isbn;
    private boolean available;
    // constructor and getters
}
```

**The collection class** — an object that holds multiple entities and answers queries about them:

```java
public class Catalog {
    private Book[] books;
    private int count;
    // addBook, findByIsbn, searchByTitle
}
```

**The preload block** — the code in `main` or an initialization method that builds the supply side before transactions begin:

```java
Catalog catalog = new Catalog(100);
catalog.addBook(new Book(...));
catalog.addBook(new Book(...));
```

**The query method** — the method that the transaction layer calls to find resources without owning them:

```java
Book found = catalog.findByIsbn(isbn);
if (found != null && found.isAvailable()) {
    // proceed with checkout
}
```

Four artifacts. The entity, the collection, the preload, the query. If you can find all four in your project and explain what each one does, you have modeled the supply side. If the query method is also the place where resources are created, you haven't — you've hidden the supply side inside the transaction, and the original failure is still there.

| artifact name | what it is | the specific code pattern that carries it | the failure mode if it is missing |
| --- | --- | --- | --- |
| entity class, collection class, preload block, query method | student should use this as a self-check before submitting the lab | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |

---

## Preloading and the Separation Test

There is a simple test for whether you have actually separated the supply side from the demand side.

Comment out every transaction in your program — every checkout, every purchase, every booking. Then run main. Does the catalog still exist? Does it still contain books? Can you still call `searchByTitle` and get results?

If the answer is yes, the supply side is independent. The catalog was built before any transaction ran and survives the removal of all transactions.

If the answer is no — if the catalog is empty, or if main throws an error, or if the books only appear when the checkout code runs — the supply side was never really modeled. It was constructed on demand by the transaction layer, which means it is not durable. It is ephemeral.

This test costs thirty seconds. Run it before you submit.

The preload block is what makes the test pass:

```java
public static void main(String[] args) {
    // Supply side built first — no transactions yet
    Catalog catalog = new Catalog(100);
    catalog.addBook(new Book("Thinking in Java",
                             "Bruce Eckel", "978-0-131-87248-6", true));
    catalog.addBook(new Book("Effective Java",
                             "Joshua Bloch", "978-0-134-68599-1", true));
    catalog.addBook(new Book("Clean Code",
                             "Robert Martin", "978-0-132-35088-4", false));

    // Separation test: this should work before any patron exists
    Book[] results = catalog.searchByTitle("Java");
    for (Book b : results) {
        System.out.println(b.getTitle() + " — available: " + b.isAvailable());
    }

    // Demand side comes after — it uses the catalog, does not create it
    Patron patron = new Patron("Alice", "LIB-0042");
    // patron.checkout(catalog, isbn) ...
}
```

The separation is visible in the structure of main. Supply side first. Demand side after. The catalog does not know the patron exists. The patron knows the catalog exists and asks it for resources.

![main() with supply/demand divider rendered as a annotated example for the chapter concept.](images/05-inheritance-and-polymorphism-fig-04.png)
*Figure 5.5 — main() with supply/demand divider*


---

## The AI Boundary

AI may generate class stubs from a written specification. It may not invent the specification.

The distinction matters here more than in earlier modules, because supply-side design is domain-specific in a way that syntax is not. Java's rules for writing a class are the same regardless of whether you are modeling a library or a warehouse. But what fields a `Book` needs, what relationships an `Author` has to a `Book`, what queries a `Catalog` must answer — those are decisions that depend on your project's requirements, your domain's actual structure, and what the transaction layer will eventually need. AI does not have that information unless you give it explicitly.

If you ask AI to design your supply-side model without writing the specification first, you will get a plausible Java class hierarchy that may compile and run and still be wrong — wrong because it solves a generic library problem, not your specific project's problem. The fields will be reasonable for some library, not necessarily yours. The relationships will reflect common assumptions, not your instructor's requirements. The queries will answer questions someone might ask, not the questions your transaction layer will actually ask.

The phase gate: for each class, write what it knows, what it does, and how it relates to other classes before asking AI for stubs.

What it knows means the fields: not just the names, but what each field represents in your domain and what constraints it has. What it does means the methods: what questions it can answer, what state changes it supports. How it relates means the object graph: which classes hold references to which other classes, and in which direction.

Write that. Three sentences per class is enough. Then give it to AI and ask for stubs.

What you get back is a starting point, not a design. Your job is to evaluate the stubs against the specification you wrote, not against your intuition that the code looks right. AI will generate a `Book` class with fields. Your job is to check whether those fields are the fields your specification required — not whether the fields seem reasonable in the abstract.

![AI three-stage workflow rendered as a process flowchart for the chapter concept.](images/05-inheritance-and-polymorphism-fig-05.png)
*Figure 5.6 — AI three-stage workflow*


---

## What the Separation Reveals

Return to the failure at the start of this module. The catalog that only existed inside the checkout method.

That design had entities — books were represented as data somewhere. What it lacked was entity objects: durable Java objects that existed before the transaction ran, could be queried independently of the transaction, and maintained state across multiple transaction calls.

The separation the module teaches is not a stylistic preference. It is what makes the following things possible:

A librarian can search the catalog without being a patron. If the catalog only exists inside the patron's checkout flow, this is impossible without duplicating the search logic.

An administrator can add books to the catalog before the library opens. If the catalog is constructed by transactions, there is no catalog to add to.

Two patrons can check out different books simultaneously without interfering. If each checkout constructs its own view of available books, the two transactions do not share a catalog — they share a fiction of one.

Each of these is a real requirement in a real library system. The supply-side model is what enables all three. Not just makes them cleaner — enables them. The transaction-creates-resource design is not a worse version of the same thing. It is a different thing that cannot do the job.

| Item | Meaning |
| --- | --- |
| can a librarian search without a patron? | can an admin add books before opening? |

The model is: entities exist. Relationships between entities exist. Transactions use entities. That order is not negotiable by the nature of the domain. Your design's job is to reflect that order in code.

When it does, the supply side is a foundation. When it doesn't, every transaction is also secretly a resource manager, and the design will fight you every time you try to extend it.

---

## The Lesson and Its Limits

The lesson from this module is specific: you can now design a supply-side object model with at least three classes, preload entities before any transaction runs, implement queries that the transaction layer can call without owning the resource data, and verify the separation by removing transactions and confirming the supply side still exists.

The limit is equally specific. This module models a static supply side — a catalog preloaded at startup, unchanged until explicitly modified. It does not address persistence: what happens to the catalog when the application closes and restarts. It does not address concurrent modification: what happens when two transactions try to change the same entity simultaneously. It does not address the full complexity of many-to-many relationships, which in a production system would be managed by a relational database rather than an array of references.

Those are real problems. They are the problems the next several modules build toward. What you have now is the foundation they require: a supply side that exists as its own layer, independent of any transaction, queryable from anywhere in the system.

A foundation that does not exist cannot be built on.

The bridge question is this: the system has resources. Who is allowed to use them?

---

## Exercises

**Warm-up**

1. Build the library supply side from scratch: a `Book` class with at least four fields, an `Author` class with at least two fields, and a `Catalog` class that holds books and supports search by title and lookup by ISBN. Preload at least five books in `main`. Run the separation test: comment out any patron or transaction code and confirm the catalog is still queryable. *(Tests: entity class design, collection class, preload block, separation.)*

2. Draw — on paper or in a text file — the object graph for your supply-side model. Show which classes hold references to which other classes, and in which direction. Label each arrow with the field name that carries the reference. Then check: does the direction of any arrow point from a supply-side object toward a transaction object? If so, name why that is a problem. *(Tests: relationship modeling, separation verification.)*

**Application**

3. Choose either an inventory domain (products, suppliers, warehouses) or a healthcare scheduling domain (providers, rooms, service types). Identify three entity classes and two relationship objects. Write a one-paragraph specification for each class — what it knows, what it does, how it relates — before writing any code. Then implement the classes and preload five objects in each. *(Tests: transfer of entity/relationship/transaction model to a new domain.)*

4. Add a second query method to your collection class — something other than a simple lookup by ID. A `searchByAuthor`, a `findAvailable`, a `filterByCategory`. The query must use a loop and return a subset of the collection. Write one sentence explaining what requirement in your domain this query satisfies. *(Tests: collection design, query method implementation, requirements connection.)*

**Synthesis**

5. Write a `main` that builds a supply side and a demand side, in that order. The demand side must call at least two query methods on the supply-side collection. Annotate every line that belongs to the supply side with `// supply` and every line that belongs to the demand side with `// demand`. Then count: how many supply-side lines come after the first demand-side line? That number should be zero. If it is not, explain what that reveals about your current design. *(Tests: layer separation, preload order, supply/demand boundary.)*

6. Take an existing class from your project where a transaction method also constructs or manages resource data. Identify the specific lines where the transaction is doing supply-side work. Describe, in prose, how you would refactor those lines into a proper entity or collection class — without writing the refactored code yet. *(Tests: design analysis, recognizing the supply-side failure in existing code.)*

**Challenge**

7. The `Catalog.searchByTitle` method returns an array sized to the actual results. This requires two passes through the data: once to count matches, once to populate the result array — or an oversized initial array that gets trimmed. Research Java's `ArrayList` and rewrite `searchByTitle` using `ArrayList<Book>` as the return collection. Explain what the `ArrayList` gives you that a fixed-size array does not, and what it costs. *(Tests: extension beyond module scope, collection choice reasoning, trade-off analysis.)*

---

## Key Terms

- **entity:** A durable object representing a resource that exists independent of user transactions. Define this term in the context of your semester project before using it in lab work.
- **relationship:** A structural connection between entities, represented as a reference or collection of references in one class pointing to another. Define this term in the context of your semester project before using it in lab work.
- **transaction:** An event object that records a user action and its effect on entity state. Define this term in the context of your semester project before using it in lab work.
- **catalog:** A collection object that holds entities and answers queries about them. Define this term in the context of your semester project before using it in lab work.
- **array:** A fixed-size, indexed sequence of values of the same type. Define this term in the context of your semester project before using it in lab work.
- **loop:** A control structure that applies the same operation to a sequence of values. Define this term in the context of your semester project before using it in lab work.
- **abstraction:** The practice of hiding the details of how something works behind the boundary of what it offers. Define this term in the context of your semester project before using it in lab work.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Discussion - Understanding Superclasses and Subclasses | Community | For-Credit | Individual | Explain inheritance relationships and design tradeoffs. |
| Optional Exercise: Constructor Chaining | Practice | For-Credit | Individual | Trace superclass and subclass constructor order. |
| Optional Exercise: Casting | Practice | For-Credit | Individual | Practice safe object casting and instanceof checks. |
| Optional Exercise: Array List | Practice | For-Credit | Individual | Use ArrayList with inherited object types. |
| Graded Lab - Classes & Inheritance | Formative | For-Credit | Individual | Implement inherited classes and verify polymorphic behavior. |
| Quiz - Inheritance & Polymorphism | Summative | For-Credit | Individual | Assess inheritance, overriding, dynamic binding, and casting. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation: authoritative source for language and library facts.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": on common novice difficulties and what instruction actually changes.
- Peng et al. and Vaithilingam et al.: on cautious claims about AI coding assistance, verification risk, and what delegation costs.

---

## References

<!-- Fact-check pass references. -->
1. Oracle. Method Invocation Expressions. Java Language Specification, Java SE 21, 2023. https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.12.4.4
