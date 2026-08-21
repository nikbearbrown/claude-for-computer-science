# Module 13: Collections and Iterators

*A test is not a proof. It is a claim made executable. The difference matters more than it sounds.*

---


## EDGE Course Outline Alignment

**Module 13: Collections and Iterators**

### Learning Objectives (LOs)

- Define the Java Collections Framework.
- Use the common methods defined in the Collection Interface to manage and manipulate groups of objects in Java.
- Implement the iterator interface to traverse the elements in a collection.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Data Structures
- Collections
- The Collection Interface
- Iterators
- Java Collections Hierarchy Framework

Here is what happened. The search method worked on Monday. You ran it. Books appeared. The right books appeared. You moved on.

On Tuesday, a new feature changed the matching rules — not intentionally breaking search, but touching the logic search depends on. Nobody ran search again before the demo. The demo ran search. The wrong books appeared, or no books appeared, or the program threw an exception it had never thrown before.

This is not a story about carelessness. It is a story about the gap between "I ran it once and it worked" and "I have stated what this method is supposed to do and made that statement executable." The search method worked on Monday because the state of the system on Monday happened to produce correct output. It failed on Tuesday because the state changed and nobody had written down what the method was required to do so that the requirement could be checked automatically.

A test is how you write down the requirement in a form the computer can check. A passing test does not prove the whole system is correct. It proves that on this input, with this setup, the output matched the stated requirement — at the time the test was run. That is a limited claim. It is also a far stronger claim than anything you can make by running the program and looking at the output.

---

## What A Test Actually Is

Before any JUnit syntax, before any assertion, there is a conceptual move that determines whether testing is useful or theatrical. The move is this: decide what the method is required to do before you decide whether it does it.

This sounds obvious. In practice it is not. Most developers who run their programs are checking whether the output looks right, not whether the output satisfies a stated requirement. Those are different activities. "Looks right" is a judgment made in the moment, with the current state of the system, by a developer who wrote the code and knows what it is supposed to produce. A stated requirement is a claim that can be checked by anyone, at any time, including after the surrounding code has changed.

A test encodes the requirement as an assertion. The assertion has three parts: a setup that creates the initial conditions the method expects; an execution that calls the method with known inputs; and a check that compares the output to what the requirement says the output should be. If the output matches, the test passes. If it does not, the test fails, and the failure is information: either the implementation is wrong, or the requirement has changed, or the setup is wrong. All three are worth knowing.

The library search method gets five tests in the worked example. Empty catalog: search on any query should return an empty list, not throw an exception. One result: search on a query that matches exactly one book should return a list of one. Multiple results: search on a query that matches several should return all of them. No result: search on a query that matches none should return an empty list. Null input: search called with a null query should not crash the program.

One of these fails. The null input test fails because the search implementation assumes the query is a non-null string and calls `.toLowerCase()` on it without checking. The assumption was never stated. It lived invisibly in the implementation. The test made it visible.

This is what tests do. They do not generate correctness. They surface assumptions.


---

## The Anatomy of a JUnit Test

A JUnit test is a Java method annotated with `@Test`. When the test runner executes, it calls every `@Test` method in the test class and reports which assertions passed and which failed.

The assertion is the heart of the test. In JUnit, assertions are static method calls: `assertEquals(expected, actual)`, `assertTrue(condition)`, `assertNull(value)`, `assertThrows(ExceptionType.class, executable)`. The expected value is the requirement. The actual value is what the method produced. When they match, the assertion is silent. When they do not match, the assertion throws an error with a message that shows both values.

This structure — expected, actual, compared — is the test translated directly into code. Before you write a single assertion, you should be able to answer two questions: what are the exact inputs I am passing to the method? And what does the requirement say the output should be for those inputs? If you cannot answer the second question, you do not have a requirement yet. You have a hope.

Here is the full structure for the multiple-results test:

```java
@Test
void searchReturnsAllMatchingBooks() {
    Library library = new Library();
    library.addBook(new Book("Design Patterns", "Gang of Four"));
    library.addBook(new Book("Clean Code", "Martin"));
    library.addBook(new Book("The Design of Everyday Things", "Norman"));

    List<Book> results = library.search("Design");

    assertEquals(2, results.size());
}
```

Setup: a library with three books, two of which contain "Design" in their titles. Execution: call `search("Design")`. Check: the result list has exactly two elements.

Notice what this test does not check: it does not verify which two books were returned, or in what order. If the requirement says search results should be sorted alphabetically, this test does not enforce that. If it says the search should match on author names as well as titles, this test does not cover that case either. A passing test is evidence that this specific claim holds. It is not evidence that every unstated claim also holds.

---

## The Boundary Cases

The five tests the module uses are not arbitrary. They follow a pattern that applies to nearly any method that takes input and returns output: the normal case, the edge case, the empty case, the no-match case, and the invalid input case.

The normal case is what the method is primarily designed to do. The happy path. One result, multiple results. These tests confirm that the method works at all.

The empty case is what happens when the input collection is empty. The library has no books. The inventory has no items. The schedule has no appointments. Many methods fail silently on empty collections — they return null when they should return an empty list, or they throw an exception they were never supposed to throw. The empty-case test catches this.

The no-match case is what happens when the method operates correctly but finds nothing satisfying the criterion. Search finds no books matching the query. The no-match case is easy to conflate with the empty case, but they are different: in the no-match case, the collection has items — they just do not match. The method should return an empty list, not null.

The boundary case in general is wherever the method's behavior transitions. If a method returns the first three results and discards the rest, the test at three results (inclusive) and four results (inclusive) defines the boundary. If a method rejects patron borrowing above the limit, the tests at the limit and one above are the boundary tests.

The invalid input case is what happens when the caller violates an implicit assumption. Null where a string was expected. Negative numbers where only positive were documented. These cases reveal assumptions the implementation made without stating. The null-query test is the example from this module: the method assumed the query was non-null, and the assumption was never written down as a requirement or enforced at the method's entry point.

| case name | what it tests | library example | inventory example | scheduling example |
| --- | --- | --- | --- | --- |
| normal case, empty case, no-match case, boundary case, invalid input case | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | Use the chapter example as the concrete test case. | Use the chapter example as the concrete test case. |
| student uses this as a checklist when designing their own test suite for any method | A concrete checkpoint for applying the chapter concept. | Use the chapter example as the concrete test case. | Use the chapter example as the concrete test case. | Use the chapter example as the concrete test case. |

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## What A Passing Test Proves

This must be said clearly, because the temptation to overclaim is strong: a passing test does not prove the method is correct. It proves the method produced the required output for the specific inputs the test supplied.

This sounds like a limitation. It is also a strength. Before the test existed, the claim "search returns all matching books" was unverifiable without running the program manually and inspecting the output. After the test exists, the claim is verifiable automatically, every time the test suite runs, by anyone with access to the codebase. The claim has been made precise — it applies to the specific setup the test creates — and has been made executable.

The testing question is not "how do I prove my code is correct?" The question is "how do I make my claims about what this code does explicit and checkable?" Tests are answers to the second question. They are not answers to the first.

This matters for how you think about test coverage. Coverage is not a goal in itself. A test suite with fifty tests that all exercise the same happy-path scenario with slightly different inputs is less valuable than five tests that cover the normal case, the empty case, the no-match case, a boundary, and an invalid input. The goal is to cover the claims you care about, including the claims about failure modes. Not to maximize the count.

---

## Regression

There is a specific use of tests that becomes important as the semester project grows: regression tests.

A regression is a failure of behavior that previously worked. The search method working on Monday and failing on Tuesday is a regression. The checkout method correctly enforcing the three-book limit until Module 11 adds a hold feature that accidentally skips the limit check — that is a regression.

Regressions are easy to introduce and hard to detect without automated tests. When you modify one part of the system, you cannot manually retest every other part before every commit. You can run a test suite. If the test suite was written to cover the behaviors that matter, a regression shows up as a newly failing test. The test was passing before the change. It is failing now. The change broke something.

This is the value of tests over time — not just when you write them, but every time the codebase changes. A test written for Module 1's checkout logic is still running in Module 13. If anything introduced in Modules 2 through 12 broke checkout, the test caught it at the moment it broke, not at the demo.

The student who writes tests as they build each module has, by Module 13, a safety net for the entire project. The student who writes tests only at Module 13 has a safety net for Module 13 and no knowledge of what is already broken.

![Regression timeline (M1–M13) rendered as a timeline / progression for the chapter concept.](images/13-collections-and-iterators-fig-01.png)
*Figure 13.2 — Regression timeline (M1–M13)*


---

## Lambda Expressions and Streams

Lambda expressions arrived in Java 8 to solve a specific problem: passing behavior as an argument without writing an entire anonymous class.

Before lambdas, filtering a list of books required either a loop with an if-statement or an anonymous class implementing a comparator or predicate interface. The intent was simple — keep the available books — but the code required substantial scaffolding to express it.

With a lambda, the same intent is expressed directly:

```java
List<Book> available = catalog.stream()
    .filter(book -> book.isAvailable())
    .collect(Collectors.toList());
```

The lambda `book -> book.isAvailable()` is an anonymous function: given a `Book`, return whether it is available. The `filter` method takes this function and applies it to each element of the stream, passing through only the elements for which the function returns true. The `collect` method gathers the results into a new list.

This is not a new capability. You could filter a list before lambdas. What the lambda and stream syntax provide is clarity of expression: the code says what it does at the level of intent, not at the level of mechanical steps. You are not reading a loop and inferring that it filters — you are reading `filter` and knowing immediately that it filters.

The question this module asks about lambdas is the same question it asks about everything: can you explain the behavior? Not "does the code compile" but "what does this lambda do, and what should the collection contain afterward?" A lambda that compiles and produces output is not the same as a lambda whose behavior you can state and verify.

The test for a lambda-based collection operation is the same test as for any other method: setup a known collection, execute the operation, assert what the result should be. If you can write that test, you understand the lambda. If you cannot, the code may be doing what you intend, but you are guessing.

---

## Where AI Fits Into This Picture

The AI boundary for this module is specific: AI may suggest test cases. It may not write the JUnit code first, because the student must translate requirements into assertions.

The ordering is the point. Writing a test requires making a requirement explicit — naming the input, stating what the output should be, deciding what the edge cases are. That translation from business requirement to executable assertion is the cognitive step the module is teaching. If AI writes the assertion before you have made the requirement explicit, AI has substituted its inference about your requirement for your actual requirement.

AI's inference is often reasonable. It is not your requirement. AI does not know whether your search method is case-sensitive or case-insensitive, whether it searches titles only or also authors, whether it returns sorted results or unsorted results. It will assume one of these. The assumption may match your implementation. It may not.

The student who asks AI to write the tests first receives code that tests AI's assumptions about the method. The student who writes the assertions first — by hand, by stating: on this input, the output should be this — and then asks AI to suggest cases they may have missed, uses AI correctly. AI may suggest the null-input case. AI may suggest the case where the query contains leading whitespace. These are genuine contributions. They are contributions to a testing strategy the student has already started, not a replacement for starting.

The phase gate is the question: for every AI-suggested test case, does it test your requirement or AI's assumed requirement? If you cannot answer that question for a test case, you do not understand the requirement well enough to accept the test.

---

## The Module's Single Capability

Every module in this course adds exactly one concrete capability to the semester project. This module's capability is: correctness as stated and executable behavioral claims, not guesswork.

In practical terms: the semester project now has a test suite. At least three methods have tests. At least two of those tests cover cases beyond the basic happy path — an edge case, a boundary, or an invalid input. At least one test was initially failing and revealed an assumption in the implementation that was not stated in the requirement. The test suite runs automatically and can be run again after any change.

The lambda and stream refactoring is secondary to the testing, but it belongs in the same module because it requires the same discipline: stating what the operation should do and verifying it with a test, not accepting a lambda because it compiles and produces roughly the right output.

By the end of this module, you should be able to look at any method in the semester project and answer: what tests exist for this method? What cases do they cover? What cases are not covered? What would a failing test tell you? These are not test-engineering questions. They are questions about what you actually know versus what you are guessing.

When you can answer them for the library search method, you can answer them for the inventory quantity check and the scheduling conflict detector. The domain changes. The specific cases differ. But the structure — requirement stated as assertion, boundary cases named, failure as information — is identical.

![Test anatomy (Setup/Execution/Assertion) rendered as a annotated example for the chapter concept.](images/13-collections-and-iterators-fig-02.png)
*Figure 13.3 — Test anatomy (Setup/Execution/Assertion)*


---

## What You Are Building Toward

Twelve modules built the model, the transactions, the persistence, the GUI, the event handling. This module adds the mechanism for knowing — not just believing — that those pieces still work as the system grows.

The bridge question is: the application has tests. How do you defend the whole system? A test suite for individual methods is necessary but not sufficient for a complex application. The system's behavior emerges from the interaction between components — the model, the controller, the persistence layer, the GUI. A search method can pass its unit tests and still fail in the application because the controller passes it the wrong argument, or the persistence layer loads a malformed catalog, or the GUI displays the results before they have finished loading.

That question — how do you test behavior that spans multiple components — is the question the bridge is pointing toward. The answer involves integration tests, whose scope extends beyond a single method, and end-to-end verification, which checks the full path from user action to model state to displayed result. But the foundation for both is what you built here: the discipline of stating requirements as executable claims before deciding whether the implementation satisfies them.

---

## Key Terms

- **JUnit** — the Java testing framework used in this course; provides the `@Test` annotation, the assertion methods, and the test runner that executes test classes and reports results.
- **assertion** — a statement in a test that compares the method's actual output to the required output; silent when the comparison holds, throws an error with both values when it fails.
- **test case** — a single `@Test` method that sets up a specific initial condition, executes the method under test with specific inputs, and asserts a specific required output; covers one behavioral claim.
- **boundary** — a point at which a method's behavior transitions; the inputs immediately on both sides of the transition are the most important test cases, because implementations most often fail at boundaries rather than in the middle of a range.
- **regression** — a failure of behavior that previously worked; detected by a test that was passing before a change and fails after it; the test suite's ability to catch regressions is its most durable value.
- **lambda** — an anonymous function expressed inline; in Java, a lambda takes arguments and returns a value without requiring a named class; used with stream operations to express collection transformations at the level of intent.
- **stream** — a sequence of elements processed by a pipeline of operations; in Java, `collection.stream()` opens a pipeline; operations like `filter`, `map`, and `sorted` transform the elements; `collect` closes the pipeline into a result.

---

## Exercises

**Warm-up**

1. For the library search method, write the requirement statement for each of the five test cases before writing any JUnit code: empty catalog, one result, multiple results, no result, and null input. Each statement should take the form: "When [setup], calling `search([input])` should return [specific expected output]." If you find you cannot state the expected output precisely — for example, you are unsure whether null input should return an empty list or throw an exception — that is the requirement gap to resolve before writing the test, not during. *(Tests: translating behavioral requirements into precise assertions before touching JUnit syntax.)*

2. The following test is written for the checkout method. Identify what claim it makes, what case it covers, and what case it does not cover:
   ```java
   @Test
   void checkoutSucceedsForAvailableBook() {
       Library library = new Library();
       Book book = new Book("Refactoring", "Fowler");
       library.addBook(book);
       Patron patron = new Patron("Ali");
       library.addPatron(patron);
       library.checkout(patron, book);
       assertFalse(book.isAvailable());
   }
   ```
   Then name two additional test cases — one boundary case and one invalid input case — that this method needs but does not have. *(Tests: reading an existing test against the five-case taxonomy; identifying what a test does and does not prove.)*

3. The following lambda filters available books:
   ```java
   List<Book> available = catalog.stream()
       .filter(book -> book.isAvailable())
       .collect(Collectors.toList());
   ```
   Before running anything, state in plain English what `available` should contain if: (a) the catalog has three available books and two checked-out books; (b) the catalog is empty; (c) all books are checked out. Then write one `@Test` method that verifies case (a). *(Tests: stating lambda behavior as a requirement before executing it; connecting the stream operation to a verifiable claim.)*

**Application**

4. Write a test suite for your project's search method (or the most search-like method in your domain). Include at least five tests covering: the normal case with one result, the normal case with multiple results, the empty-collection case, the no-match case, and the null-input case. At least one test must initially fail and reveal an assumption in the implementation. Document the assumption it revealed and what you changed to address it. *(Tests: applying the full five-case taxonomy; experiencing a failing test as information rather than failure.)*

5. Refactor one collection operation in your project to use a lambda and stream. The operation must have at least one `@Test` method before the refactoring and the same test must pass after. Write one sentence explaining what the lambda expresses at the level of intent that the loop version did not express as directly. *(Tests: refactoring with test coverage as the safety net; articulating what the lambda-based version communicates differently.)*

6. Write an AI prompt that complies with this module's boundary — it describes the method under test and asks AI to suggest test cases you may have missed, after you have already written at least three assertions yourself. Run the prompt. For each AI-suggested case, answer: does this test my stated requirement, or AI's assumed requirement? Accept, reject, or modify each suggestion with a one-sentence explanation. *(Tests: applying the AI boundary in practice; distinguishing your requirements from AI's inferences about them.)*

**Synthesis**

7. A classmate's test suite has twelve tests, all of which pass. They conclude the project is correct. Write a two-paragraph response. The first paragraph describes a specific failure mode — a method, a case, and a scenario — that the test suite cannot detect because no test covers it, even with twelve passing tests. The second paragraph explains the difference between "the tested behaviors are correct" and "the system is correct," using the module's vocabulary (claim, assumption, coverage, regression). *(Tests: applying the limits-of-testing argument; reasoning about what passing tests do and do not prove.)*

8. The search method currently searches titles only. A new requirement says it should also search author names. You need to update the implementation without breaking the existing tests. Describe your approach: which existing tests would you run first, and why? Which new tests would you write before changing the implementation? What would a failing test at this point tell you that a passing test would not? *(Tests: using the test suite as a regression guard during a feature change; practicing test-first thinking for a requirement extension.)*

**Challenge**

9. The bridge question asks: the application has tests — how do you defend the whole system? Write your best current answer before reading Module 14. Specifically: you have unit tests for individual methods. Identify one place in your semester project where two components interact — the controller calling a model method, the persistence layer loading data into a collection, the search results being displayed in the `TableView` — and describe a failure that the unit tests for each individual component would not catch but that would still break the application. Then sketch what kind of test would catch it. There is no single correct answer; the goal is to make the gap between unit testing and system correctness visible before the next module addresses it. *(Points toward integration testing and the limits of testing components in isolation.)*

---

## Bridge

The application has tests. How do you defend the whole system?

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Optional Exercise: Collections and Iterators | Practice | For-Credit | Individual | Use collection methods and iterators to process objects. |
| Quiz - Collections and Iterators | Practice | NDNC | Individual | Check vocabulary and traversal concepts. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation — use for language and library facts, including the specification of lambda expressions and the stream API.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming" — use for common novice difficulties, including the tendency to conflate "it ran" with "it is correct."
- Peng et al. and Vaithilingam et al. — use for cautious, empirically grounded claims about AI coding assistance and the specific risks of AI-generated test suites that test inferred rather than stated requirements.

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
1. Oracle. Class ArrayList. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html
2. Oracle. Class HashMap. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html
3. Oracle. Interface Iterator. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html
4. Oracle. Interface Comparator. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html
