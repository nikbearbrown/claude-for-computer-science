# Module 9: Event-Driven Programming
*A collection is not a container. It is a decision about how you will access what is inside.*

Five hundred books. Sort them by title.

The first solution works. You write a loop, compare strings, swap elements. The books come out alphabetical. You move on.

Three weeks later, the requirement changes: sort by author last name, then by title within each author. You go back to the sorting code. It is a tangle of string comparisons with no names attached to anything. You cannot tell which comparison is the title sort and which is something else. You rewrite it. You probably introduce a bug. The new sort works in the demo. You are not confident it handles edge cases — authors with no last name, titles that start with "The," books with multiple authors — because the logic is buried in comparisons rather than expressed as rules.

The second solution, the one this module teaches, names the ordering rule before implementing it. The rule is an object. The object has a method. The method says: given two books, which comes first? When the requirement changes, you change the rule object. You do not excavate a comparison tangle. You replace a named thing with another named thing.

That is the difference between code that sorts and code that sorts *for a reason*.

---


## EDGE Course Outline Alignment

**Module 9: Event-Driven Programming**

### Learning Objectives (LOs)

- Describe the concept of handler classes in event-driven programming, including how handler objects are registered to respond to specific events.
- Describe events, event sources, and event classes.
- Apply knowledge of event-driven programming to write and analyze code that effectively manages and responds to various events.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Event-driven programming
- Handlers and handling user events
- Event handling
- Inner classes
- The KeyEvent Class
- KeyCode constants
- Procedural vs. Event-Driven Programming

## What Collections Are Actually Doing

Most beginners think of a collection as a place to put things. A `List` is a box. You put books in it. You get books out. You sort them when you need to.

That framing is not wrong, but it is incomplete in a way that creates real problems.

Collections do not just store things. They encode assumptions about how you will access what is inside. Those assumptions affect what operations are fast, what operations are slow, and what operations are even possible. When you choose a collection, you are making a performance and capability contract with the rest of the code.

`List` is the right choice when order matters and you will iterate: display the books in the order the librarian specified, show results in the order they were returned by a search. `List` gives you indexed access and a defined iteration order. The cost is that lookup by a key — find the book with ISBN 978-0-7432-7356-5 — requires scanning the whole list in the worst case.

`Map` is the right choice when you need to look up by key: find the book with this ISBN, find the account for this username, find the appointment for this patient ID. `Map` gives you O(1) average-case lookup. The cost is that you have no defined iteration order over the entries, and you cannot express "give me the second book."

`Set` is the right choice when membership matters and duplicates do not: which ISBNs are currently checked out, which patron IDs have overdue fines. `Set` gives you fast membership testing and automatic duplicate elimination. The cost is that elements have no order (unless you use `TreeSet`) and no index.

The mistake most beginners make is using `List` for everything. `List` is flexible and familiar. It compiles. It runs. The performance problem only shows up when the collection grows: a `List`-based lookup that scans five hundred books is imperceptibly fast; the same scan over fifty thousand books is not.

| operation | best collection type | why | what you give up |
| --- | --- | --- | --- |
| ordered display | iteration (List | It makes the underlying reasoning visible instead of implied. | A concrete checkpoint for applying the chapter concept. |
| keyed lookup by ID (Map | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. | A concrete checkpoint for applying the chapter concept. |
| membership test | deduplication (Set | It makes the underlying reasoning visible instead of implied. | A concrete checkpoint for applying the chapter concept. |
| sorted traversal (List with sort or TreeSet | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. | A concrete checkpoint for applying the chapter concept. |
| multiple sort orders (List with Comparator) | this is the reference table students consult when choosing a collection for a new feature | It makes the underlying reasoning visible instead of implied. | A concrete checkpoint for applying the chapter concept. |

The phase gate for this module asks you to name three things before writing collection code: the three most common operations your application will perform on this collection, and the expected size. Those answers determine the right collection type. When you have stated them, you have a spec. AI can then scaffold the implementation of that spec.

---

## Comparator: The Ordering Rule as an Object

Sorting in Java is built around a single idea: the comparison rule should be separable from the objects being compared.

There are two mechanisms for expressing that rule. `Comparable` is for natural ordering — the ordering that makes sense for the object itself, in most contexts. A `Book` that implements `Comparable<Book>` is saying: if you ask me to compare myself to another book without any other instructions, I will use my title. `Comparable` is baked into the class.

`Comparator` is for external ordering — an ordering rule that exists independently of the objects being sorted. A `Comparator<Book>` that sorts by author is a separate object. You can have as many `Comparator` objects as you have ordering requirements. You pass the one you want to `Collections.sort` or `List.sort` at the time you need the sort.

The practical difference: `Comparable` handles the default case. `Comparator` handles everything else.

Here is a `Comparator` that sorts books by title:

```java
Comparator<Book> byTitle = Comparator.comparing(Book::getTitle);
```

Here is one that sorts by author last name, then by title within each author:

```java
Comparator<Book> byAuthorThenTitle = Comparator
    .comparing(Book::getAuthorLastName)
    .thenComparing(Book::getTitle);
```

Here is one that sorts by year, most recent first:

```java
Comparator<Book> byYearDescending = Comparator
    .comparingInt(Book::getPublicationYear)
    .reversed();
```

Notice what each of these is: a named rule. `byAuthorThenTitle` is not a tangle of string comparisons. It is an object with a name that describes its purpose. When the requirement changes — sort by author last name, then by title, then by edition — you modify this object. You add `.thenComparingInt(Book::getEdition)`. The sort call does not change. The collection does not change. The rule changes.

![Comparator: tangled inline vs. named object rendered as a comparison panels for the chapter concept.](images/09-event-driven-programming-fig-01.png)
*Figure 9.1 — Comparator: tangled inline vs. named object*


To sort the library's book list by author then title:

```java
List<Book> books = catalog.getAllBooks();
books.sort(byAuthorThenTitle);
```

That is the complete operation. `books.sort` takes the list and the comparator. The comparator defines the order. The list is reordered in place.

---

## Comparable: The Natural Order

When an ordering is genuinely intrinsic to the object — when you would always sort books by title in the absence of other instructions — implementing `Comparable` in the class makes that order the default.

```java
public class Book implements Comparable<Book> {
    private String title;
    private String author;

    @Override
    public int compareTo(Book other) {
        return this.title.compareTo(other.title);
    }
}
```

The `compareTo` method returns a negative number if `this` should come before `other`, zero if they are equal, and a positive number if `this` should come after. `String.compareTo` already does this for alphabetical order, so for a title-based natural ordering, delegating to `String.compareTo` is correct.

Once `Book` implements `Comparable`, you can sort a `List<Book>` without a `Comparator`:

```java
Collections.sort(books); // uses Book's natural order: by title
```

The design decision: use `Comparable` for the ordering you would choose if you had to pick only one. Use `Comparator` for everything else.

| mechanism | where the rule lives | how many orderings | when to use it | how to apply it |
| --- | --- | --- | --- | --- |
| Comparable (inside the class, one natural order, use for the default sort, Collections.sort(list | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| Comparator (outside the class, as many as needed, use for alternate orderings, list.sort(comparator)) | the purpose is to make the design decision mechanical, not mysterious | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Filtering: A Different Kind of Selection

Sorting reorders a collection. Filtering reduces it: return only the elements that satisfy a condition.

The Java stream API makes filtering readable. A stream is a pipeline: you take a collection, pass it through a sequence of operations, and collect the result. The operations do not modify the original collection. They produce a new one.

Here is a filter that returns only available books:

```java
List<Book> available = books.stream()
    .filter(book -> book.isAvailable())
    .collect(Collectors.toList());
```

Here is a filter combined with a sort: available books, sorted by title:

```java
List<Book> availableByTitle = books.stream()
    .filter(Book::isAvailable)
    .sorted(byTitle)
    .collect(Collectors.toList());
```

The pipeline reads like a description of what you want: take all books, keep the available ones, sort them by title, collect the result. Each step is named. Each step does one thing.

Filtering by author name:

```java
String authorQuery = "Ursula K. Le Guin";
List<Book> byAuthor = books.stream()
    .filter(book -> book.getAuthor().equals(authorQuery))
    .collect(Collectors.toList());
```

The `filter` method takes a predicate — a function from an element to a boolean — and keeps only the elements for which the predicate returns `true`. Any condition expressible as a boolean can be a filter predicate.

![Stream pipeline with persistent source rendered as a process flowchart for the chapter concept.](images/09-event-driven-programming-fig-02.png)
*Figure 9.2 — Stream pipeline with persistent source*


There is one thing to watch for. Stream operations produce a new collection; they do not modify the original. If you call `books.stream().filter(...)` and do not assign the result, nothing changes. This is the most common beginner error with streams, and it is invisible: the code compiles, the operation runs, the result is discarded.

---

## The Library in Practice: Three Collections Doing Three Jobs

In the library semester project, collections appear in at least three distinct roles, each requiring a different type.

The catalog — all books in the library — is a `List<Book>`. Order matters for display. The librarian sees books in a specified sequence. Iteration is the primary operation: show me all books, filtered and sorted. ISBN-based lookup is secondary and infrequent enough that scanning the list is acceptable.

The checkout index — a lookup from patron ID to their current loans — is a `Map<String, List<Loan>>`. The primary operation is "given this patron, what do they have checked out?" That is a keyed lookup. A `Map` gives O(1) average-case access. The value is a `List<Loan>` because a patron may have multiple loans and their order may matter for display.

The overdue set — patron IDs with at least one overdue item — is a `Set<String>`. The primary operation is "is this patron overdue?" That is a membership test. `Set.contains` gives O(1) average-case performance. Duplicate patron IDs are meaningless — either a patron is overdue or not — and `Set` enforces that automatically.

Three collections. Three different types. Each type chosen because it matches the operation pattern of its role.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

When students use `List` for all three, the catalog display is fine, the patron lookup is slower than necessary, and the overdue check requires iterating the full overdue list for every lookup instead of a single `contains` call. The application still works. In a demo with ten patrons and fifty books, the performance difference is invisible. With ten thousand patrons and one hundred thousand books, it is not.

The habit this module builds is choosing collection type by operation, not by familiarity.

---

## What AI Can and Cannot Do Here

AI can write a `Comparator` quickly and correctly. Given a precise ordering rule — sort by author last name ascending, then by title ascending, then by publication year descending — AI will produce a correct `Comparator` chain. This is a legitimate use of AI assistance: the rule is yours, the Java expression of the rule is mechanical, AI handles the mechanical part.

What AI cannot do is choose the collection type for you. The choice depends on which operations matter, how frequently they occur, and how large the collection will grow. Those facts live in your domain knowledge and your project requirements. AI does not have them. A plausible AI choice — `ArrayList` for everything, because it is the most common Java collection — will be correct for some roles and wrong for others, and the wrongness will not appear until the collection is large enough to expose the performance difference.

The phase gate follows from this. Before asking AI for collection code, write the operation profile: what are the three most common operations on this collection, and what is the expected size? When that exists, AI can scaffold the implementation. Without it, AI is guessing at your access patterns.

Here is the prompt:

```text
I am working on Module 9: Event-Driven Programming in INFO 5100.
My domain is [library / inventory / scheduling].
My collection: [name and element type].
My three most common operations: [list them].
My expected size: [approximate number of elements].
Help only within this boundary: AI may write a comparator from a precise
ordering rule. It may not choose the collection type before I have named
the common operations.
My ordering rule is: [describe the sort order precisely].
```

The prompt works because the collection type decision is already made before AI is involved. AI receives the ordering rule and writes the `Comparator`. You verify that the comparator sorts a known set of elements in the expected order.

---

## The Inventory and Scheduling Shape

The same three-collection pattern appears in every domain.

In an inventory system, the product catalog is a `List<Product>` for display and filtered browsing. The SKU index is a `Map<String, Product>` for lookup by stock-keeping unit. The low-stock set is a `Set<String>` of product IDs below the reorder threshold. The `Comparator` sorts products by category, then by name within category. The filter returns only in-stock items.

In a healthcare scheduling system, the appointment list for a provider's day is a `List<Appointment>` sorted by start time. The patient index is a `Map<String, Patient>` for lookup by patient ID. The booked slot set is a `Set<TimeSlot>` for availability checking — given a requested time, is it already taken? The `Comparator` sorts appointments by time. The filter returns only appointments of a specific type.

In both domains, the design question is the same: what operation do I perform most often, and which collection type makes that operation fast? The answer determines the type. The type determines the performance contract.

| domain | collection role | collection type | primary operation | why this type |
| --- | --- | --- | --- | --- |
| library catalog (List, iteration | display | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| library checkout index (Map, keyed lookup | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| library overdue set (Set, membership test | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| inventory catalog (List | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| inventory SKU index (Map | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| inventory low-stock set (Set | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| scheduling appointment list (List | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| scheduling patient index (Map | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |
| scheduling booked slots (Set) | this makes the pattern visible across all three domains simultaneously | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | It makes the underlying reasoning visible instead of implied. |

---

## Verifying a Sort

A `Comparator` that compiles is not necessarily a `Comparator` that sorts correctly. The verification step is to construct a small, known set of elements, sort them, and check the result against what you expect.

For the `byAuthorThenTitle` comparator, construct at least these cases:

- Two books with different authors: the earlier author alphabetically should come first.
- Two books with the same author and different titles: the earlier title alphabetically should come first.
- Two books with the same author and the same title: the comparator should return zero or a stable order.
- A book with a null author or null title, if null values are possible in your data: the comparator should handle this without throwing a `NullPointerException`.

That last case is the one most AI-generated comparators miss. `Comparator.comparing(Book::getTitle)` will throw a `NullPointerException` if any book's title is null, because `String.compareTo` does not accept null. If null titles are possible — a book scanned into the system before its metadata was complete, for example — the comparator needs to handle them explicitly.

The verification is not sophisticated. It is four test cases. But four specific test cases against a known expected output are how you know the comparator is correct, not just plausible.

| test case description | input (book A fields) | input (book B fields) | expected compareTo result (negative | zero |
| --- | --- | --- | --- | --- |
| different authors, same author different titles, same author same title, null title present | students fill in the "expected" columns before running the sort, then compare to actual output | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| the null-title row should show what Comparator.comparing() does vs. what Comparator.comparing(key, nullsFirst | nullsLast) does | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## What This Module Does Not Settle

We have connected collection type to access-pattern reasoning, implemented named ordering rules as `Comparator` and `Comparable`, and built filter pipelines using streams. We have not connected the sorted, filtered collection to a GUI component.

The bridge question is: the data is organized. How does it appear in a GUI?

A `List<Book>` sorted by title is not visible to the user until it is bound to a display component — a table, a list, a set of labeled rows in a panel. That binding is the subject of the next module, where the collection becomes the model and the display component becomes the view. The design question there is the same as the design question here: what operations does the display need, and how do you expose them from the collection without letting the view modify the underlying data?

---

## Lab: Three Sort Orders and One Filter

Implement three sort orders for your semester project's primary collection: one that uses `Comparable` (the natural order), two that use named `Comparator` objects. Implement one filter using the stream API.

Before writing any code, submit the operation profile for at least one collection in your project: collection name, element type, three most common operations, expected size. The operation profile is the justification for the collection type. The collection type is the design decision. The comparators and filters implement the spec that the design decision implies.

If you used AI to write a `Comparator`, include the ordering rule you provided — precisely, as you stated it to AI — and your verification: the test cases you ran, the expected output, and the actual output. State whether the AI-generated comparator handled null values correctly.

---

## Exercises

**Warm-up**

1. For each of the three collections in the library project — the catalog `List<Book>`, the checkout index `Map<String, List<Loan>>`, and the overdue `Set<String>` — write the operation profile: three most common operations and expected size. Then confirm that the collection type assigned in the module matches the profile. If you would choose differently for any of them, explain why. *(Tests: collection-type-to-operation reasoning; operation profile as a design spec.)*

2. Write a `Comparator<Book>` named `byYearThenTitle` that sorts books by publication year ascending, then by title alphabetically within each year. Write it using the `Comparator.comparingInt().thenComparing()` chain. Then write the single line that applies it to a `List<Book>` named `books`. *(Tests: Comparator chain construction; list.sort() call.)*

**Application**

3. The `byAuthorThenTitle` comparator works correctly for books where both fields are non-null. A book is added to the catalog before its author metadata is entered, leaving `getAuthorLastName()` returning `null`. Describe exactly what happens when the comparator encounters this book, name the exception if one is thrown, and write the corrected comparator using `Comparator.nullsLast()` or `Comparator.nullsFirst()` as appropriate for your domain. *(Tests: null-handling in comparators; Comparator.nullsFirst/nullsLast; verification test case design.)*

4. A classmate implements a search feature by iterating a `List<Book>` and comparing each book's ISBN to the query. It works in the demo with fifty books. Apply the complexity definition from this module: what is the O-notation for this operation, what collection type would reduce it to O(1), and what change to the project's data structure would be required to support that lookup? *(Tests: O(n) vs. O(1) reasoning; List-to-Map migration; operation-profile thinking.)*

5. You ask AI for a filter that returns appointments scheduled for a specific provider on a specific date. AI returns: `appointments.stream().filter(a -> a.getProvider().equals(provider) && a.getDate().equals(date)).collect(Collectors.toList())`. Apply the AI boundary test: is this AI-appropriate scaffolding or a design decision you should have made first? What would you verify before accepting this output, and what edge case does the filter not handle that you should check? *(Tests: stream filter syntax; AI boundary applied to stream operations; verification of filter correctness.)*

**Synthesis**

6. Translate the library's three-collection pattern to your own domain (inventory or scheduling). Name the three collections, assign a type to each, and write the operation profile for each. For one of the three, write the `Comparator` that implements the primary sort order for that collection. *(Tests: domain transfer; collection-type assignment by operation; Comparator construction in a new domain.)*

7. The library catalog currently uses a `List<Book>`. A new requirement adds: "given an ISBN, return the book immediately, without scanning the list." Two design options: (a) keep the `List` and add a `Map<String, Book>` as a parallel index; (b) replace the `List` with the `Map` as the primary store and derive sorted display order by collecting and sorting. For each option, describe what it gives up, name the operation that becomes slower or more expensive, and recommend which is better for the library's access patterns — then state what evidence would change your recommendation. *(Tests: List-Map tradeoff; parallel index pattern; design reasoning from operation profiles.)*

**Challenge**

8. The stream pipeline `books.stream().filter(Book::isAvailable).sorted(byAuthorThenTitle).collect(Collectors.toList())` is called every time the library's main display panel refreshes — which happens on every user interaction. The catalog contains eighty thousand books. Describe the performance implications: how many elements does `filter` examine, how many does `sorted` sort, and what is the combined cost in O-notation? Then propose a caching strategy — without changing the collection type — that would reduce the cost of repeated identical queries, and identify what invariant the cache must maintain to stay correct when books are checked out or returned. *(Open-ended; anticipates caching, invalidation, and the model-view update patterns introduced in the next module.)*

---

- **List** — a Java collection that preserves insertion order and supports indexed access. The right choice when order matters and iteration is the primary operation. Lookup by value requires scanning.
- **Map** — a Java collection that stores key-value pairs and supports O(1) average-case lookup by key. The right choice when the primary operation is "given this key, give me the value." No defined iteration order over entries.
- **Set** — a Java collection that stores unique elements and supports O(1) average-case membership testing. The right choice when the primary question is "is this element present?" Duplicates are automatically eliminated.
- **Comparator** — an object that defines an ordering rule for elements of a given type. Separate from the elements being sorted; can be named, composed, reversed, and reused. The right choice for all orderings except the natural order.
- **Comparable** — an interface that a class implements to define its natural ordering. The ordering that applies when no `Comparator` is specified. Only one natural order is possible per class.
- **filter** — a stream operation that returns a new collection containing only the elements for which a predicate returns `true`. Does not modify the original collection.
- **stream** — a pipeline over a collection that supports filter, sort, map, and collect operations without modifying the source. Each operation returns a new stream; `collect` materializes the result.
- **complexity** — a characterization of how an operation's cost grows with collection size. O(1) means constant cost regardless of size. O(n) means cost grows linearly with size. Choosing the wrong collection type converts O(1) operations into O(n) operations.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Optional Exercise: Creating a JavaFX application with a response for user action | Practice | For-Credit | Individual | Register an event handler and verify response behavior. |
| Optional Exercise: Creating a JavaFX Program (MyKeyEvent) | Practice | For-Credit | Individual | Use keyboard events and KeyCode constants. |
| Quiz - Event-Driven Programming | Summative | For-Credit | Individual | Assess event sources, event objects, and handlers. |
| Optional Lab - Event-Driven Programming | Practice | For-Credit | Individual | Build and test a small event-driven GUI. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and the Java SE API documentation: authoritative sources on `Comparator`, `Comparable`, `Collections`, `List`, `Map`, `Set`, and the stream API.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": research on the abstraction difficulties that collection-choice reasoning requires.
- Peng et al. and Vaithilingam et al.: empirical work on AI coding assistance; the operation-profile phase gate is grounded in their findings on verification risk when collection-type decisions are delegated.

---

## References

<!-- Fact-check pass references. -->
1. OpenJFX. Interface EventHandler. JavaFX 21 API Specification, 2023. https://openjfx.io/javadoc/21/javafx.base/javafx/event/EventHandler.html
