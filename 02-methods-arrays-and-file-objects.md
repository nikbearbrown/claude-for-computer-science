# Module 2: Methods, Arrays, and File Objects

*The blueprint doesn't age. The buildings do.*

---


## EDGE Course Outline Alignment

**Module 2: Methods, Arrays, and File Objects**

### Learning Objectives (LOs)

- Define what a method is and explain its purpose in programming.
- Understand how to create, initialize, and manipulate arrays to solve programming challenges.
- Write programs in Java to read data from existing files, write data to new files, and manipulate file content effectively.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Methods
- Arrays
- File objects

Here is something that should bother you.

You write a class called `Patron`. You give it a field called `age`. You make two patrons — call them `patronA` and `patronB`. You set `patronA.age` to 34 and `patronB.age` to 51. Then you print them both. Different ages. Of course different ages. You set them yourself.

So why does this feel like it needs explaining?

Because what you just did is not obvious. It only looks obvious after you understand it. The printout showed you two different ages, but it did not show you *why* they are two different ages. It did not show you what Java is actually doing underneath — where those two numbers live, how the two names reach them, and what would happen if you did something slightly different and got numbers you did not expect. The printout hid all of that. And the moment you hit a bug — the moment the two ages are *not* different when they should be, or *are* the same when they shouldn't be — the printout has nothing more to tell you.

That is the moment this module is for.

---

## The Blueprint and the Building

A class is a blueprint. Not a metaphor for a blueprint — an actual blueprint, in the sense that it describes a structure without being the structure. The blueprint for a house specifies how many rooms, where the windows go, what material the walls are made of. The blueprint does not have rooms. It describes rooms.

When Java reads your `Patron` class, it reads a description. It learns that every Patron will have a `name` (a String), an `age` (an int), and a `libraryId` (a String). It learns what methods a Patron can perform. It does not yet create anything you could point to in memory. The class is inert until you ask Java to build from it.

The build instruction is `new`:

```java
Patron patronA = new Patron("Alice", 34, "LIB-0042");
```

Now something exists. Java allocates a block of memory on the heap — the region of memory used for objects — and writes three values into it: the string `"Alice"`, the integer `34`, the string `"LIB-0042"`. That block of memory is an object. It is one specific instance of the Patron blueprint.

The word `patronA` on the left side of the assignment is not the object. This is the part that causes the most confusion, and it is worth stopping here long enough to feel the distinction.

`patronA` is a reference variable. It holds an address — a way of finding the object in memory. Think of it as a sticky note with a street address written on it. The note is not the house. If you hand someone the note, they can find the house. If you lose the note, the house does not disappear immediately (though eventually Java's garbage collector will reclaim it). If you write a new address on the note, you are not demolishing the old house — you are just pointing somewhere else.

![Reference variable → heap object rendered as a structural schematic for the chapter concept.](images/02-methods-arrays-and-file-objects-fig-01.png)
*Figure 2.1 — Reference variable → heap object*


---

## Two Objects From One Blueprint

Now add a second patron:

```java
Patron patronB = new Patron("Bernard", 51, "LIB-0091");
```

Java does it again. Another block of memory on the heap. Another set of three values. Another address, this one held by the variable `patronB`.

You now have one class, two objects, two reference variables. Draw it:

```
Stack                  Heap
------                 ------
patronA  ──────────►  [ name: "Alice"   | age: 34 | libraryId: "LIB-0042" ]
patronB  ──────────►  [ name: "Bernard" | age: 51 | libraryId: "LIB-0091" ]
```

The stack is where Java keeps local variables and references. The heap is where it keeps objects. The arrows represent the reference — the address on the sticky note.

![Two independent objects rendered as a structural schematic for the chapter concept.](images/02-methods-arrays-and-file-objects-fig-02.png)
*Figure 2.2 — Two independent objects*


Now try this:

```java
patronA.age = 35;
```

You followed the reference from `patronA`, arrived at the first heap block, and changed the value there. `patronB.age` is still 51. You never followed `patronB`'s reference. You never touched its heap block.

This is the behavior you saw in the printout. Now you can explain it.

---

## What Can Go Wrong

The mental model earns its keep when things go wrong, not when they go right.

Suppose you write this:

```java
Patron patronC = patronA;
patronC.age = 99;
System.out.println(patronA.age);
```

What prints?

Most students guess 34. The answer is 99.

`patronC = patronA` does not copy the object. It copies the reference. Now two sticky notes have the same address. `patronC` and `patronA` point at the same heap block. When you change `patronC.age`, you arrive at that block and change the value. When you ask for `patronA.age`, you follow a different sticky note to the same block and read the changed value.

This is not a bug in Java. It is Java working exactly as designed. But it will feel like a bug every time you encounter it without the mental model, because the printout will show you a number you didn't expect and give you no indication why.

![Shared-reference trap rendered as a structural schematic for the chapter concept.](images/02-methods-arrays-and-file-objects-fig-03.png)
*Figure 2.3 — Shared-reference trap*


The reference-versus-object distinction is not a detail. It is the foundational model for understanding every null pointer exception, every unexpected mutation, every moment when two variables seem to be mysteriously synchronized. Those errors do not make sense without this. They make complete sense with it.

---

## The Java Artifact That Carries This

Every concept in this book has a corresponding artifact in Java source code — a line or structure you can point to, read, and modify. Abstract understanding is good. But the book's standard is higher: you should be able to locate the concept in code.

For the reference-versus-object model, the artifacts are these:

**The class definition** — the blueprint, inert until instantiated:

```java
public class Patron {
    String name;
    int age;
    String libraryId;

    public Patron(String name, int age, String libraryId) {
        this.name = name;
        this.age = age;
        this.libraryId = libraryId;
    }
}
```

**The constructor** — the instruction Java follows when `new` is called. Notice `this`. It distinguishes the object's field (`this.name`) from the parameter with the same name (`name`). Without `this`, the assignment would be self-referential and meaningless. The constructor is not optional decoration — it is the code that populates the heap block.

**The instantiation** — the moment `new` creates an object and a reference is born:

```java
Patron patronA = new Patron("Alice", 34, "LIB-0042");
```

**The field access** — the moment you follow a reference and read or write a value:

```java
patronA.age = 35;
System.out.println(patronA.name);
```

Four artifacts. One concept. If you can find all four in your own code and explain what each one does, you understand the concept. If you can only explain it in words, you are not there yet.

| Item | Meaning |
| --- | --- |
| mapping each artifact (class definition, constructor, instantiation, field access) to its role, the line of code that carries it, and what disappears from your understanding if you skip it | student should be able to use this as a self-check checklist |

---

## Displaying Multiple Objects

Now that you have two objects, you need to display them. Java gives you several options, and the one you choose reveals something about your priorities.

The simplest: call `System.out.println()` on each one.

```java
System.out.println(patronA.name + ", age " + patronA.age);
System.out.println(patronB.name + ", age " + patronB.age);
```

This works. It also scales badly. Three patrons means six lines. A hundred patrons means two hundred lines, which is not a program — it is a list.

The better move is a method on the class itself:

```java
public void display() {
    System.out.println(name + ", age " + age + " [" + libraryId + "]");
}
```

Now you call `patronA.display()` and `patronB.display()`. The object knows how to represent itself. This is not just cleaner — it is a design decision. You have moved the display logic inside the boundary of the class, which means it changes in one place when the format changes, and it is testable in isolation.

There is a second option Java provides: override `toString()`.

```java
@Override
public String toString() {
    return name + ", age " + age + " [" + libraryId + "]";
}
```

When `System.out.println()` receives an object, it calls `toString()` automatically. So `System.out.println(patronA)` produces the same output as the `display()` method above. The difference is that `toString()` makes your object readable anywhere Java might convert it to a string — in logs, in error messages, in debugging output. It is a small investment with wide returns.

| method | where it's called | who calls it | what it enables | when to use it — student should see that toString() is the wider-reaching choice |
| --- | --- | --- | --- | --- |
| two-column comparison of display() vs toString() — | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Tracing Before Running

Here is a practice worth building now, because it compounds.

Before you run a program, trace it. Take a piece of paper, or an empty file. Write the variable names on the stack side. Draw arrows as references are created. Draw boxes on the heap side as objects are instantiated. Update values as assignments happen.

Then run the program. Check whether the output matches your trace.

If it does, you understood the program. If it does not, you have a specific prediction that was wrong, which means you have a specific question: *where did my model break?* That question is far more tractable than the generic confusion of unexpected output with no prior prediction.

This is the Feynman standard applied to code: you understand something when you can predict what it will do before it does it. Not after. Not "oh yes, of course" when you see the answer. Before.

The trace habit is especially important for the reference-versus-object model because the relationship between variable names and heap blocks is invisible at runtime. The printout shows you values. It does not show you addresses, arrows, or which variables share a block. The trace makes the invisible visible before execution hides it again.

![Trace-before-run rendered as a comparison panels for the chapter concept.](images/02-methods-arrays-and-file-objects-fig-04.png)
*Figure 2.4 — Trace-before-run*


---

## The AI Boundary

The question of where AI assistance belongs in this module has a specific answer, not a vague one.

AI may quiz you on code you wrote. Give it your `Patron` class and ask it to ask you five questions about it. This is legitimate use: you have done the design and implementation work, and you are using AI to check your understanding from the outside, the way a study partner might. The verification work — the understanding — remains yours.

AI may identify possible design issues. If your constructor is assigning parameters in the wrong order, or your `toString()` is incomplete, AI can flag it. But notice what this requires: you must have written the class first. AI identifying issues in code that does not yet exist is not design review — it is generation disguised as review.

AI may not write the first class body.

This boundary is not arbitrary. The first class body is where the design decisions happen: what fields are necessary, what the constructor should require, what the methods should do and return. Those decisions are not derivable from a prompt. They depend on requirements you understand and AI does not — your project's domain, your instructor's acceptance standard, what a correct implementation must prove. If you delegate the first class body, you skip the step where understanding forms. You get output that might compile, which is not the same thing as getting a solution.

The phase gate for this module: write the class yourself first. Then ask AI to ask you five questions about it.

If you pass the five questions — if you can answer them without looking at the code — the class is yours. If you cannot, you have found the gap, which is exactly the information the module is designed to surface.

![AI boundary flowchart rendered as a process flowchart for the chapter concept.](images/02-methods-arrays-and-file-objects-fig-05.png)
*Figure 2.5 — AI boundary flowchart*


---

## What the Trace Reveals

Return to the opening. Three printouts. Same class. Three different ages. Where do the values live?

They live on the heap, in three separate object blocks, each reached by a different reference. The class contributed the structure — the fields, the constructor, the methods. The objects contributed the values. The references are the only connection between the code you wrote and the data that lives in memory at runtime.

When you change a value through one reference, you are reaching into a specific heap block. Other references that do not point to that block are unaffected. Other references that do point to that block will see the change.

That is the whole model. A class is a blueprint. An object is one instantiation of that blueprint, living in memory. A reference is the address by which code finds an object. Multiple references can reach the same object. Multiple objects can be built from the same blueprint.

The model is simple. The implications are not, and you will spend the rest of the course discovering them. But every discovery will be a version of this one, applied at a new scale or in a new context. The reference-versus-object distinction does not go away — it becomes the vocabulary in which the harder problems are stated.

When you understand it now, at the scale of two Patron objects and a printout, the harder problems will have a language. When you don't, they will feel like new problems every time, because you will not be able to see that they are the same problem in a different coat.

---

## The Lesson and Its Limits

The lesson from this module is specific: you can now create multiple objects from a single class, trace the references that connect variable names to heap blocks, and explain why two objects with the same structure can hold different values. You have the mental model. You have the four Java artifacts that carry it. You have a tracing practice that makes the model testable before runtime.

The limit is also specific. This module gives you one coherent slice. It does not give you collections — the structures that let you manage a hundred Patron objects without a hundred variable names. It does not give you the full picture of how Java manages memory, or what happens when no reference points to an object anymore, or how the mental model extends to objects that contain other objects. Those are coming. They are extensions of this, not replacements for it.

What you can do now is narrow but solid. A solid narrow foundation is how you build something that does not fall over.

The bridge question is this: you have objects. How does a user move through them?

---

## Exercises

**Warm-up**

1. Write a `Book` class with four attributes: `title`, `author`, `isbn`, and `yearPublished`. Write a constructor and a `toString()` method. Instantiate three distinct books and print each one. Confirm that changing one book's `title` does not affect the others. *(Tests: class definition, instantiation, reference independence.)*

2. Trace the following program on paper before running it. Predict the output for each `println`. Then run it and check your prediction.
   ```java
   Patron x = new Patron("Carla", 29, "LIB-0007");
   Patron y = x;
   y.age = 40;
   System.out.println(x.age);
   System.out.println(y.age);
   Patron z = new Patron("Carla", 29, "LIB-0007");
   System.out.println(x == z);
   ```
   Explain in one sentence why `x == z` prints what it does. *(Tests: reference-versus-object model, shared reference behavior.)*

**Application**

3. Choose either an inventory domain (products in a warehouse) or a scheduling domain (appointments in a clinic). Write a class appropriate to that domain with at least four attributes. Instantiate two distinct objects, display them, and write a one-paragraph explanation of your design choices: what fields you included, what the constructor requires, and why `toString()` is implemented the way it is. *(Tests: transfer of the blueprint-to-object model to a new domain.)*

4. Add a `display()` method and a `toString()` override to the same class. Call both on the same object. Write two sentences explaining the difference in where and how each is useful. *(Tests: method design, `toString()` role.)*

**Synthesis**

5. Write a short program that creates three objects from your domain class, stores them in three separate reference variables, then deliberately creates a shared-reference situation — two variables pointing at the same object. Modify a field through one variable and print through both. Annotate each line with a comment explaining what is happening on the stack and on the heap. *(Tests: integration of reference model with deliberate design choice.)*

6. Your `Patron` class has a `libraryId` field. Suppose two patrons checked out under the same library card — a real scenario in library systems. Using only what you have learned in this module, what are the limitations of the current design? What field or behavior is missing? Write the answer in prose, not code. *(Tests: design analysis, recognizing the boundary of current capability.)*

**Challenge**

7. The `==` operator on objects tests reference equality — whether two variables point to the same heap block. Java provides a `equals()` method for content equality — whether two objects have the same field values. Research and implement a correct `equals()` override for your domain class. Test it with three cases: two variables pointing at the same object, two variables pointing at objects with identical field values, and two variables pointing at objects with different field values. Explain why Java does not make `==` do content equality by default. *(Tests: extension beyond module scope, research skill, design reasoning.)*

---

## Key Terms

- **attribute / field:** A named variable declared inside a class, holding data that belongs to each instance. Define this term in the context of your semester project before using it in lab work.
- **method:** A named block of behavior declared inside a class, callable through a reference. Define this term in the context of your semester project before using it in lab work.
- **constructor:** The special method Java calls when `new` is used; it populates the new object's fields. Define this term in the context of your semester project before using it in lab work.
- **reference:** A variable that holds the address of a heap object, not the object itself. Define this term in the context of your semester project before using it in lab work.
- **stack:** The memory region where local variables and references are kept during method execution. Define this term in the context of your semester project before using it in lab work.
- **heap:** The memory region where objects live after `new` creates them. Define this term in the context of your semester project before using it in lab work.
- **instance:** One specific object built from a class blueprint. Define this term in the context of your semester project before using it in lab work.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Exercise: Methods | Practice | For-Credit | Individual | Write methods with clear parameters, return values, and call sites. |
| Exercise: getDouble() Methods | Practice | For-Credit | Individual | Practice input validation and reusable method design. |
| Exercise: Searching an Array | Practice | For-Credit | Individual | Traverse arrays and report matches. |
| Exercise: Reading from a File | Practice | For-Credit | Individual | Open and process file input safely. |
| Exercise: Writing to a File | Practice | For-Credit | Individual | Create file output and verify saved content. |
| Optional Lab - Fundamentals of Programming in Java | Practice | For-Credit | Individual | Integrate methods, arrays, and file objects. |
| Module quiz | Formative | For-Credit | Individual | Check readiness before object-oriented programming. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation: authoritative source for language and library facts.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": on common novice difficulties and what instruction actually changes.
- Peng et al. and Vaithilingam et al.: on cautious claims about AI coding assistance, verification risk, and what delegation costs.

---

## References

<!-- Fact-check pass references. -->
1. Oracle. Class Object. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html
2. Oracle. The Structure of the Java Virtual Machine. Java Virtual Machine Specification, Java SE 21, 2023. https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html
