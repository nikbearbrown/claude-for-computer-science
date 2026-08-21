# Module 8: Abstract Classes and Interfaces

*The program remembered everything. Until it didn't.*

---


## EDGE Course Outline Alignment

**Module 8: Abstract Classes and Interfaces**

### Learning Objectives (LOs)

- Define interfaces.
- Design and use abstract classes.
- Create classes that implement interfaces.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Abstract methods
- Abstract class
- Interfaces
- Definition and Example of Abstract Class
- Interface Definition
- Example of Using Interface Design in Java: Integer and BigInteger Classes

Here is a failure you will not notice until it is too late.

You spend an hour entering books into your library catalog. You add titles, authors, ISBNs. You run the program. Everything works. You close the program. You open it again.

The catalog is empty.

Not corrupted. Not partially saved. Empty. As if the hour never happened.

Your code is not broken, exactly. It did what you told it to do. You told it to store books in an array. The array lived in memory. Memory is temporary. When the process ended, the memory was reclaimed. The books were never anywhere else.

The user calls this broken. The code calls it in-memory. Both are right.

This is the moment persistence becomes necessary — not as a nice-to-have feature, but as a requirement. Data that cannot survive a program restart is not data. It is a calculation. And the moment you understand that distinction, you realize that every system you have built so far in this course has been doing arithmetic, not record-keeping. The objects were real. The records were not.

This module is about making them real.

---

## What CRUD Actually Means

CRUD is an acronym: Create, Read, Update, Delete. It is the full lifecycle of a piece of data in a system that cares about persistence.

Before persistence, none of these are interesting. You create a `Book` object with `new`. You read it with a getter. You update it with a setter. You delete it by letting the reference go out of scope. These operations happen in memory, at the speed of the processor, with no consequences for any future session.

After persistence, each operation becomes a commitment. To create a book is to write a record to storage that will survive the process ending. To read a book is to reconstruct an object from that record at startup or on demand. To update a book is to change the stored record, not just the in-memory object — and to change it in a way that is safe even if the write is interrupted. To delete a book is to remove it from storage, not just from the current session.

The gap between "it works in memory" and "it works with persistence" is where most data management bugs live. Not logic errors. Not syntax errors. Design errors: the engineer assumed the operation was complete when the object was updated, but the file was never written. The engineer assumed the file was current, but it was written before the last update. The engineer assumed the read succeeded, but the file was missing and the exception was silently caught.

CRUD is not a checklist of features. It is a commitment to what the data lifecycle requires at each step.

| operation | in-memory behavior | persistent behavior |
| --- | --- | --- |
| Create (new object vs write record to file | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| Read (getter vs reconstruct from file | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| Update (setter vs update file record safely | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| Delete (dereference vs remove from storage) | student should see that persistence transforms each operation from trivial to consequential | A concrete checkpoint for applying the chapter concept. |


---

## CSV as a Deliberate Choice

There are many ways to store data outside memory: relational databases, document stores, binary files, object serialization. This module uses CSV — comma-separated values — and the choice is deliberate.

CSV is not the right tool for production systems with thousands of users and concurrent writes. It is the right tool for learning what persistence requires, because it is transparent. You can open the file in a text editor and see exactly what was written. You can corrupt it manually and see exactly what breaks. You can trace the path from object to file to object again without any intermediary system obscuring the steps.

A CSV file for a library catalog looks like this:

```
Thinking in Java,Bruce Eckel,978-0-131-87248-6,true
Effective Java,Joshua Bloch,978-0-134-68599-1,true
Clean Code,Robert Martin,978-0-132-35088-4,false
```

Each line is one book. Each comma-separated field maps to one attribute. The file has no headers here — you could add them, but they require special handling on read. The simplicity is the point. Every design decision in CSV persistence is visible.

The Java side has two operations: writing objects to a file, and reading them back.

Writing:

```java
public void saveToFile(String filename) throws IOException {
    BufferedWriter writer = new BufferedWriter(new FileWriter(filename));
    for (int i = 0; i < count; i++) {
        Book b = books[i];
        writer.write(b.getTitle() + "," +
                     b.getAuthor() + "," +
                     b.getIsbn() + "," +
                     b.isAvailable());
        writer.newLine();
    }
    writer.close();
}
```

Reading:

```java
public void loadFromFile(String filename) throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader(filename));
    String line;
    while ((line = reader.readLine()) != null) {
        String[] parts = line.split(",");
        String title     = parts[0];
        String author    = parts[1];
        String isbn      = parts[2];
        boolean available = Boolean.parseBoolean(parts[3]);
        addBook(new Book(title, author, isbn, available));
    }
    reader.close();
}
```

These two methods are the full persistence layer for a simple catalog. They are also not enough on their own, and understanding why requires understanding what can go wrong.

![Write path / read path with failures rendered as a process flowchart (paired) for the chapter concept.](images/08-abstract-classes-and-interfaces-fig-01.png)
*Figure 8.2 — Write path / read path with failures*


---

## The Lifecycle in Full

Here is the data lifecycle for a session with file-based persistence:

1. The program starts. It calls `loadFromFile`. The catalog is populated from disk.
2. The user adds a book. The catalog's in-memory array is updated.
3. The user checks out a book. The availability flag is changed in memory.
4. The program ends. It calls `saveToFile`. The catalog is written to disk.

This works. Now introduce a single variation: the program crashes between step 3 and step 4.

The availability flag was changed in memory. The file on disk still shows the book as available. The next session loads the old file. The book appears available when it is not.

This is a data loss scenario — not dramatic, not a crash on startup, but real. The record does not match the world. A patron reserves a book that is already checked out. A reconciliation is required that the system cannot do automatically, because it does not know a crash happened.

The question this raises is not technical. It is a policy question: how much data loss is acceptable? Losing the last transaction before a crash is often acceptable in a learning project. It is not acceptable in a banking system, an inventory system with financial consequences, or a healthcare system where a missing appointment record could affect patient care.

You cannot answer this question in code. You have to answer it before you write the code, because the answer determines the code. There are three different implementations of the save operation depending on the answer:

**Save on exit only:** Simple. One write at shutdown. Loses all changes since last save if the process is killed. Acceptable for personal-use catalogs with low stakes.

**Save after every mutation:** Every `addBook`, every `checkout`, every `updateTitle` triggers a file write. Safe. Slow. May be fine for small catalogs. Gets expensive with large files.

**Save to a temporary file, then rename:** Write the new state to `catalog.tmp`, then rename it to `catalog.csv`. On any OS that makes rename atomic, this means the file is either the old version or the new version, never a half-written state. The standard technique for crash-safe persistence in simple file-based systems.

| strategy | when save runs | data loss window | performance cost | when it is appropriate |
| --- | --- | --- | --- | --- |
| save on exit, save after every mutation, atomic rename (write to .tmp then rename) | student should use this to make the policy decision before writing code, not after | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

The decision is yours. The code follows from the decision. Getting the code right without making the decision is not possible — you will get code that makes the decision for you, and you may not like what it decided.

---

## Exception Handling Is Not Optional

The `saveToFile` and `loadFromFile` methods above both declare `throws IOException`. This is Java's way of saying: something can go wrong here that I cannot handle internally, and the caller must decide what to do.

The things that can go wrong:

- The file does not exist when you try to read it. First run, after deletion, wrong path.
- The file exists but is not readable. Permissions, corruption, network drive disconnect.
- The write fails partway through. Disk full, permissions revoked mid-write, process killed.
- The file contains data in an unexpected format. Missing fields, extra commas, encoding issues.

Each of these requires a different response. A missing file on first run is not an error — it means the catalog is empty, and the correct response is to start with an empty catalog. A missing file after three months of use is a problem — it may mean the file was deleted, and the correct response is to warn the user and refuse to overwrite with an empty catalog. A partial write is serious — the file may be corrupted, and the correct response may be to restore from a backup.

Here is the minimum correct exception handling for the load operation:

```java
public void loadFromFile(String filename) {
    File f = new File(filename);
    if (!f.exists()) {
        // First run or file deleted: start empty, this is not an error
        return;
    }
    try {
        BufferedReader reader = new BufferedReader(new FileReader(f));
        String line;
        while ((line = reader.readLine()) != null) {
            String[] parts = line.split(",");
            if (parts.length < 4) {
                // Corrupted or incomplete line: skip and log, do not crash
                System.err.println("Skipping malformed line: " + line);
                continue;
            }
            try {
                String title     = parts[0];
                String author    = parts[1];
                String isbn      = parts[2];
                boolean available = Boolean.parseBoolean(parts[3]);
                addBook(new Book(title, author, isbn, available));
            } catch (Exception e) {
                System.err.println("Could not parse line: " + line);
            }
        }
        reader.close();
    } catch (IOException e) {
        System.err.println("Failed to load catalog: " + e.getMessage());
    }
}
```

This is longer than the original. Every additional line represents a decision about what the system should do when something goes wrong. The original version — the one that declared `throws IOException` and let the caller worry about it — was not wrong. It was incomplete. It deferred the decision. This version makes the decisions explicit.

Notice what the decisions are:

- Missing file → start empty, not an error.
- Malformed line → skip and log, do not crash the load.
- Parse failure on a field → skip that record, log the problem.
- IO failure during read → log, accept partial catalog.

You may disagree with some of these. That is the point. Exception handling is not a matter of adding try-catch blocks until the compiler stops complaining. It is a matter of deciding, for each failure mode, what the correct behavior is and implementing that behavior explicitly. The compiler will accept silent swallowing of exceptions. The user will not accept losing data because you did not decide what to do when the disk was full.

| failure mode | what the naive code does | what the correct response is | the Java pattern that implements it |
| --- | --- | --- | --- |
| missing file on first run (crash vs return silently | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| malformed line (ArrayIndexOutOfBoundsException vs skip and log | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| IO failure during read (propagates vs catch and partial load | The pattern becomes easy to misuse or overlook. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| write failure during save (silent vs log and warn) | student should be able to match each row to a specific block in the loadFromFile code above | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Tracing the Full Path

The Feynman test for persistence: can you trace a single book from user input to storage and back to display, naming the Java artifact at each step?

Start with creation. The user enters a title, author, and ISBN. The input handling code creates a `Book` object and calls `catalog.addBook(book)`. The in-memory array now contains the book.

The book is in memory. It is not persisted. If the program ends here, it is gone.

Save is called — either immediately, on exit, or at a defined checkpoint. `saveToFile` iterates the in-memory array, formats each book as a CSV line, and writes it to the file. The book is now on disk.

The book is on disk. It is not in memory in the next session. If the program restarts without loading, the catalog starts empty.

Load is called at startup. `loadFromFile` reads the file line by line, parses each line into a `Book` object, and calls `addBook`. The in-memory catalog is reconstructed from disk.

The book is now back in memory. It can be displayed. The full lifecycle: object → file → object.

Each transition is a potential failure point. The object-to-file transition fails if the write throws an exception, the disk is full, or the process is killed mid-write. The file-to-object transition fails if the file is missing, the format is wrong, or a field cannot be parsed. The in-memory state fails to match the file if a mutation happened between load and the next save.

![Object → file → object lifecycle rendered as a process flowchart for the chapter concept.](images/08-abstract-classes-and-interfaces-fig-02.png)
*Figure 8.3 — Object → file → object lifecycle*


You cannot prevent all failures. You can design for them — decide which ones are fatal, which ones should be logged and skipped, and which ones should trigger a warning to the user. That design is the persistence layer. The file I/O code is the implementation of the design.

---

## The Update Problem

Create and Read are conceptually clear. Update is where persistence gets complicated.

In memory, updating a book is simple:

```java
book.setTitle("New Title");
```

One line. Immediate. The object now has a new title. Nothing else changes.

With CSV persistence, the same update requires more work. You changed the in-memory object. The file still has the old title. They are now out of sync. The next time you call `saveToFile`, the change will be written. Until then, a program restart will reload the old title.

The gap between "updated in memory" and "written to file" is the window in which data loss can occur. For simple cases, the solution is to call `saveToFile` immediately after every mutation. For larger catalogs, this is expensive — you are rewriting the entire file to change one field.

The more sophisticated approach is to track which records are dirty — changed in memory but not yet saved — and write only those. This requires a more complex data model. It also introduces a new failure mode: the dirty-tracking mechanism itself can have bugs.

The simplest approach that is correct: save the whole file after every mutation. It is slow for large catalogs. It is correct. When you understand why it is slow and what the alternatives cost, you can optimize. Not before.

The Delete operation has the same structure. Deleting a book from the in-memory array is easy. The record still exists in the file until the next full save. If the program crashes before the save, the "deleted" book reappears on restart. Whether this is acceptable depends on the same policy question as before: how much data loss is acceptable?

![Memory vs. file timeline with data-loss window rendered as a timeline / progression for the chapter concept.](images/08-abstract-classes-and-interfaces-fig-03.png)
*Figure 8.4 — Memory vs. file timeline with data-loss window*


---

## The Java Artifacts That Carry This

For persistence, the artifacts are:

**The writer method** — the code that serializes in-memory objects to a file format:

```java
public void saveToFile(String filename) throws IOException { ... }
```

**The reader method** — the code that deserializes a file format back to in-memory objects:

```java
public void loadFromFile(String filename) { ... }
```

**The call sites** — where in `main` or the application lifecycle save and load are called:

```java
catalog.loadFromFile("catalog.csv");   // at startup
// ... session ...
catalog.saveToFile("catalog.csv");     // at shutdown or after mutation
```

**The exception handlers** — the explicit decisions about what to do when each failure mode occurs.

Four artifacts. If you can find all four in your project, explain what each one does, and answer the three policy questions — what if the write fails, what if the file is missing, how much data loss is acceptable — you have a persistence layer. If you have the writer and reader but not the call sites, you have methods that are never called. If you have the call sites but not the exception handlers, you have code that will crash on the first file system anomaly.

The exception handlers are not boilerplate. They are the design. AI can generate the writer and reader. It cannot decide what your system should do when the disk is full, because that depends on requirements you have and AI does not.

| artifact | what it is | the specific code pattern | the failure mode if it is missing |
| --- | --- | --- | --- |
| writer method (saveToFile | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| reader method (loadFromFile | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| call sites (load at startup | save at mutation or exit | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |
| exception handlers (explicit decisions per failure mode) | student should check all four are present and named before submitting the lab | A concrete checkpoint for applying the chapter concept. | The pattern becomes easy to misuse or overlook. |

---

## The AI Boundary

AI may generate CSV boilerplate. It may not decide exception handling or acceptable data loss.

The boundary is sharper here than in any previous module, because the failure modes of persistence code are domain- and requirement-specific in a way that is not visible from the code alone.

A missing file on first run is an expected condition that should be handled silently. A missing file after six months of production use is a serious problem that should be logged and escalated. The code for both is a check for `f.exists()` before the read. But the behavior — what to do when the check returns false — is completely different, and it depends on context that only you have.

If you ask AI to generate persistence code, you will get writer and reader methods that compile and run. You may also get exception handling that swallows errors silently, or that crashes the program on conditions that should be recoverable, or that accepts data loss scenarios that are not acceptable in your domain. These are not bugs in the AI's output. They are consequences of the AI not knowing your requirements.

The phase gate for this module: before AI writes persistence code, answer three questions in writing.

What if the write fails? Name the specific behavior: log and continue, retry, crash, restore the previous file, prompt the user.

What if the file is missing? Name the specific behavior: start empty, refuse to run, restore from backup, prompt the user for a path.

How much data loss is acceptable? Name the window: zero — save after every mutation; one session — save only on exit; manual — save when the user explicitly requests it.

Write those answers. They do not need to be long. One sentence each is enough. Then give the answers to AI along with the code structure and ask it to generate persistence code that respects those decisions.

What you get back is an implementation of your design. Your job is to verify that the implementation matches your decisions, not that it looks like reasonable persistence code in the abstract.

![Three-question persistence design form rendered as a annotated example / decision form for the chapter concept.](images/08-abstract-classes-and-interfaces-fig-04.png)
*Figure 8.5 — Three-question persistence design form*


---

## What Persistence Reveals

Return to the opening. The catalog that vanished on restart.

That system had objects. It had methods. It had a working session. What it lacked was a commitment to the data: a decision that this information matters beyond the current process, that it should be findable the next time the program runs, and that the code should do the work of writing it somewhere and reading it back.

Persistence is that commitment made concrete. A writer method, a reader method, call sites at the right moments, and exception handlers that reflect a real decision about failure tolerance. Four artifacts. A set of policy answers that determine their behavior.

The interesting thing about data loss is that it is almost never caused by a single catastrophic failure. It is caused by a sequence of reasonable-looking decisions: store it in memory now, save it at the end, handle the exception by catching and continuing. Each decision was plausible. The combination was wrong. The user lost an hour of work.

The fix is not better exception handling in isolation. It is understanding the lifecycle — create, read, update, delete, save, load, recover — and making an explicit decision at each transition point about what failure means and what the response should be.

That decision cannot be delegated. It can be informed by AI, by colleagues, by documentation. But the decision itself is yours, because you are the only one who knows what the system is for, who depends on it, and what losing the data would cost.

The code is the record of the decision. If you cannot explain the decision, you cannot defend the code. And if you cannot defend the code, you do not understand the persistence layer. You have implemented it, which is not the same thing.

The bridge question is this: data survives. How do users search, sort, and target it?

---

## Exercises

**Warm-up**

1. Add CSV persistence to the library catalog from Module 5. Implement `saveToFile` and `loadFromFile`. Call load at startup and save on exit in `main`. Then test restart behavior: add three books, exit, restart, confirm the books are present. *(Tests: writer method, reader method, call sites, basic persistence cycle.)*

2. Introduce a deliberate failure: after populating the catalog and before calling `saveToFile`, add `System.exit(0)` to simulate a crash. Restart the program. Which books are present? Write one sentence explaining why, naming the specific lifecycle transition that was interrupted. *(Tests: data loss analysis, lifecycle understanding.)*

**Application**

3. Answer the three policy questions for your domain in writing before touching the code: (a) What if the write fails? (b) What if the file is missing at startup? (c) How much data loss is acceptable? Then implement exception handlers in `loadFromFile` that match your answers. For each handler, write an inline comment explaining which policy decision it implements. *(Tests: policy-before-code discipline, exception handler design, requirements traceability.)*

4. Implement the update operation end-to-end: change a field on an in-memory object, immediately call `saveToFile`, restart the program, confirm the change persisted. Then deliberately corrupt the CSV file — remove a field from one line — and confirm your load method handles it without crashing. Describe what it does with the corrupted record. *(Tests: update lifecycle, exception handling for malformed data.)*

**Synthesis**

5. Implement the Delete operation: remove a record from the in-memory collection, then call `saveToFile`. Verify that the deleted record does not reappear on restart. Then answer: what is the window of data loss between the delete call and the save call? Under what conditions could the "deleted" record reappear? Write the answer in prose, not code. *(Tests: delete lifecycle, data loss window analysis.)*

6. Write a `main` that demonstrates all four CRUD operations in sequence — create, read, update, delete — with a save after each mutation and a load at startup. Add a comment before each operation naming which CRUD letter it represents and which Java artifact carries it. After running the full sequence, verify that the file on disk reflects the final state. *(Tests: full CRUD integration, lifecycle tracing, artifact identification.)*

**Challenge**

7. The save-on-every-mutation approach rewrites the entire file for every change. For a catalog of 10,000 books, this means writing 10,000 lines to update one field. Research the concept of a dirty flag — a boolean on each object that tracks whether it has been modified since last save — and design a `saveIfDirty` method that writes only the modified records. Describe the trade-offs: what does the dirty flag buy you, what does it cost, and what new failure mode does it introduce? You do not need to implement it in full — designing it and naming the trade-offs is the exercise. *(Tests: extension beyond module scope, performance trade-off reasoning, failure mode analysis.)*

---

## Key Terms

- **CRUD:** Create, Read, Update, Delete — the four operations that define the full lifecycle of a persistent data record. Define this term in the context of your semester project before using it in lab work.
- **CSV:** Comma-Separated Values — a plain-text file format where each line is one record and each comma-delimited field maps to one attribute. Define this term in the context of your semester project before using it in lab work.
- **persistence:** The property of data that allows it to survive the end of the process that created it. Define this term in the context of your semester project before using it in lab work.
- **exception:** A runtime condition that Java cannot handle internally and must surface to the calling code for a decision. Define this term in the context of your semester project before using it in lab work.
- **IOException:** The exception Java throws when a file system operation fails — file not found, permission denied, disk full, read error. Define this term in the context of your semester project before using it in lab work.
- **data integrity:** The property of a data store where every record accurately reflects the current state of the system it models, with no stale, missing, or corrupted records. Define this term in the context of your semester project before using it in lab work.
- **load:** The operation of reading records from persistent storage and reconstructing in-memory objects from them. Define this term in the context of your semester project before using it in lab work.
- **save:** The operation of writing in-memory object state to persistent storage in a format that can be reloaded. Define this term in the context of your semester project before using it in lab work.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Optional Exercise: Defining an Interface | Practice | For-Credit | Individual | Declare and implement an interface. |
| Optional Exercise: Example of Using Integer and Date Classes With Their compareTo() Method | Practice | For-Credit | Individual | Use comparable behavior in existing Java classes. |
| Quiz - Basics of Abstract Classes and Interfaces | Summative | For-Credit | Individual | Assess abstract classes, interfaces, and implementation choices. |
| Graded Lab - Abstract Classes and Interfaces | Formative | For-Credit | Individual | Design classes that use abstraction appropriately. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation: authoritative source for language and library facts, including `BufferedReader`, `BufferedWriter`, `FileReader`, `FileWriter`, and the `java.io` exception hierarchy.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": on common novice difficulties and what instruction actually changes.
- Peng et al. and Vaithilingam et al.: on cautious claims about AI coding assistance, verification risk, and what delegation costs.

---

## References

<!-- Fact-check pass references. -->
1. Oracle. Method Invocation Expressions. Java Language Specification, Java SE 21, 2023. https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.12.4.4
