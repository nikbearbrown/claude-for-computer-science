# Module 4: Basics of Object-Oriented Programming Part 2

*Wrong output is not the problem. Wrong output is the symptom. The problem is somewhere upstream, waiting for a hypothesis.*

---


## EDGE Course Outline Alignment

**Module 4: Basics of Object-Oriented Programming Part 2**

### Learning Objectives (LOs)

- Demonstrate proficiency in object-oriented thinking.
- Understand wrapper classes.
- Use String, StringBuilder, and StringBuffer classes.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Passing Objects to Methods
- Arrays of objects
- Processing Primitive Data Types Values as Objects
- The valueOf methods
- Interned Strings
- Replacing and Splitting Strings
- The this Reference
- The String Class
- The StringBuilder and StringBuffer Classes

Here is the situation. The program compiles. It runs. It produces output. The output is wrong.

There is no stack trace. No red text. No `NullPointerException` pointing at line 47. The program finished cleanly and handed you something incorrect. You stare at the output. You stare at the code. You change a line. You run it again. The output is still wrong, or wrong in a different way, or briefly right in a way you cannot explain and therefore cannot trust.

This is where most debugging goes badly. Not because the tools are poor. Not because the bug is unusually subtle. Because the developer is patching output instead of diagnosing cause. They are changing things until the symptom disappears, without ever understanding what produced the symptom in the first place. The bug may be gone. Or it may be hidden. Or it may have moved. They do not actually know, because they never formed a theory.

This module is about forming a theory first.

---

## The Distinction That Changes Everything

Before any debugger, before any tool, before any line of code gets touched, there is a conceptual move that separates disciplined debugging from guessing with better equipment. The move is this: distinguish the symptom from the cause.

A symptom is what you observe. Wrong output is a symptom. A patron with the wrong book attached is a symptom. A balance that is off by exactly the amount of the last transaction is a symptom. Symptoms are how bugs announce themselves. They are not the bug.

A proximate cause is the immediate line or operation that produced the symptom. The assignment that attached the wrong patron. The arithmetic that applied the transaction twice. The method that returned before updating the record. Proximate causes are closer to the bug, but they are still not the root.

A root cause is the design decision, state error, or logic flaw that made the proximate cause possible. A shared reference that was modified when it should have been copied. A field initialized to `null` when it should have been initialized to an empty list. A method that assumes a precondition no one enforces. Root causes are upstream. They often exist far from where the symptom appears, which is exactly why the symptom is a misleading place to start.

The debugging workflow this module teaches — hypothesize, isolate, test — is an engine for moving from symptom to root cause without getting stuck at proximate cause. Each step narrows the search. Each step requires committing to a claim before checking whether the claim is true. That commitment is the thing that distinguishes diagnosis from tinkering.

![Symptom / proximate / root hierarchy rendered as a hierarchy for the chapter concept.](images/04-basics-of-object-oriented-programming-part-2-fig-01.png)
*Figure 4.1 — Symptom / proximate / root hierarchy*


---

## Hypothesize

A hypothesis is a falsifiable claim about what is wrong and where it is wrong.

"Something is off with the checkout" is not a hypothesis. It is a restatement of the symptom. "The `currentPatron` variable holds a stale reference at the point the book is assigned" is a hypothesis. It names a specific object, a specific field, and a specific moment in execution. It can be checked. It can be confirmed or refuted. That checkability is what makes it useful.

Forming a good hypothesis requires reading the code as a causal system. Every object has state. Every method modifies state or produces output from state. Every bug is a place where the actual state diverges from the expected state. Your hypothesis should name the divergence: which field, in which object, at which point in execution, holds a value that is wrong.

This sounds demanding. It is. The alternative is worse. If you do not form a hypothesis, you are not debugging — you are searching, and the search space for a non-trivial program is enormous. A hypothesis collapses the search space to a single location and a single claim. Even a wrong hypothesis is valuable, because ruling it out tells you something true about the system.

Here is the practical question: how do you form a hypothesis before you have confirmed anything? You read the code forward from where the correct behavior was last verified and backward from where the wrong output was first observed. Somewhere in that interval, the state went wrong. Your hypothesis is your best current guess about where.

![Hypothesize → Isolate → Test loop rendered as a cycle / process flowchart for the chapter concept.](images/04-basics-of-object-oriented-programming-part-2-fig-02.png)
*Figure 4.2 — Hypothesize → Isolate → Test loop*


---

## Isolate

Once you have a hypothesis, you need evidence. Isolation is the process of creating conditions under which the hypothesis can be confirmed or refuted.

In the NetBeans debugger, isolation has a concrete form: the breakpoint. A breakpoint is an instruction to the runtime to pause execution at a specific line and give you control. When execution pauses, you can inspect the state of every object in scope — the values of fields, the contents of collections, the identity of references. You are no longer reading what the code says should happen. You are seeing what is actually happening, at this moment, in this run.

Set the breakpoint at the line your hypothesis names. If your hypothesis is that `currentPatron` holds the wrong reference at the point of assignment, set the breakpoint on that assignment. When execution pauses there, inspect `currentPatron`. Either it holds what you expected, in which case your hypothesis is wrong and you need a new one, or it holds something else, in which case your hypothesis is confirmed and you now need to ask a different question: how did `currentPatron` arrive at this line holding the wrong value?

That second question is how you move from proximate cause toward root cause. You step back upstream. You find the last point at which `currentPatron` held the correct value. You find the first point at which it held the wrong value. Between those two points is the operation that introduced the error. That operation is the root cause, or close to it.

The debugger gives you two navigation tools for this: step over and step into. Step over executes the current line and pauses at the next one, without descending into any method called on that line. Step into executes the current line and, if it calls a method, pauses at the first line inside that method. The choice between them is a choice about where you think the bug lives. If your hypothesis says the bug is in the current method, step over. If your hypothesis says the bug is inside a method being called, step into. The hypothesis guides the navigation. Without a hypothesis, you are stepping randomly through someone else's program.

---

## The Specific Bug This Module Teaches

Let me make the worked example concrete so the method has something to run against.

The library checkout program has a sequence-dependent bug. When a patron checks out a single book, everything is correct. When a patron checks out two books in succession, the second checkout attaches the book to the wrong patron. The output names the right patron for the first book and the wrong patron for the second.

This is a symptom. The proximate cause is wherever the wrong patron gets connected to the book. But notice: the first checkout works. The bug appears only on the second. That observation is already a hypothesis generator. Something changes between the first checkout and the second. The most likely candidates are: a reference that was supposed to be temporary is being reused; a variable that was supposed to be reset between checkouts is not being reset; a shared object is being modified when it should have been copied.

A reasonable first hypothesis: `currentPatron` is not reset between checkouts, and the second checkout operates on the same patron reference as the first.

Set a breakpoint at the start of the checkout method. Run the second checkout. When execution pauses, inspect `currentPatron`. If it holds the first patron's name instead of the second patron's name, the hypothesis is confirmed. The bug is upstream — wherever `currentPatron` is supposed to be updated between checkouts, the update is not happening. Step back and find the assignment. Find why it was skipped.

If the hypothesis is wrong — `currentPatron` correctly names the second patron — then the bug is further downstream. The patron reference is correct at the start of checkout but gets corrupted before the assignment. Update the hypothesis: inspect the patron reference at each step between the start of checkout and the book assignment. Find where it diverges.

This is the loop: hypothesis, isolation, observation, revision. It converges. Guessing does not.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## What The Debugger Actually Shows You

There is a distinction between running a program and understanding a program that the debugger makes viscerally clear.

When you run a program, you see its output. The output is the end of a causal chain. Everything that produced the output is invisible. You have one data point: final state. When you debug a program with breakpoints, you see intermediate state — the values of objects mid-execution, before the causal chain has finished. You have a time-slice view of the program's life. That is an entirely different kind of information.

This is why print statements, the oldest debugging technique, are still useful but limited. A print statement gives you one value at one moment, chosen in advance. A breakpoint gives you all the values at that moment, and you can choose which ones to look at after the program pauses. The debugger is a more powerful form of the same idea.

The NetBeans debugger presents this information as a variables panel. When execution is paused, every variable in scope appears with its current value. For primitive types — `int`, `boolean`, `double` — the value is the number or flag. For objects, the value is a reference, and you can expand that reference to see the object's fields, and expand those fields if they are also objects. This is how you trace a reference chain: you follow the arrows from one object to the next until you find the field whose value is wrong.

The skill is not operating the tool. The skill is reading what the tool shows you against your hypothesis. The tool shows you state. Your hypothesis predicts what the state should be. The comparison between prediction and observation is the diagnostic act. Developers who are good at debugging are not people who know more keyboard shortcuts. They are people who form precise predictions and read the evidence carefully.

| property | print statement | breakpoint |
| --- | --- | --- |
| when you decide what to observe, how many values visible at once, can you follow a reference chain, can you navigate upstream, effort to add | remove | student should see that breakpoints extend, rather than replace, the print-statement intuition |

---

## Where AI Fits Into This Picture

The AI boundary for this module is specific, and the specificity matters: AI may explain error categories or stack traces. It may not inspect code until the student states a hypothesis.

The ordering is not arbitrary. It is the same ordering as the debugging workflow itself. The workflow requires forming a hypothesis before gathering evidence. The AI boundary requires forming a hypothesis before presenting code to AI for analysis. If you present code to AI without a hypothesis, you are asking AI to do the diagnostic work — to read the causal system and identify the divergence. AI will do this. The answer may even be correct. But you will have skipped the cognitive step that builds the skill.

The cognitive step is forming the hypothesis. That requires holding the causal structure of the program in your head, tracing the state forward and backward, and committing to a claim about where the state goes wrong. That mental model — the causal map of the program — is what debugging competence actually is. If AI builds the map for you, you leave the exercise without the map. The next bug will be no easier.

The productive use of AI here is as a check on a hypothesis you already hold. You form the hypothesis. You state it to AI. You ask whether the hypothesis is plausible given the code you show. AI may confirm it, refine it, or point out a case you had not considered. That is AI as a thinking partner, not AI as a replacement for thinking. The difference is who forms the initial claim.

This boundary also protects you from a specific failure mode in AI-assisted debugging: AI confidently identifies the wrong root cause. It happens. AI reads the code, finds a plausible issue, explains it clearly, and the explanation is wrong. If you have not formed your own hypothesis, you have no independent basis for evaluating AI's hypothesis. You will accept it because it sounds right. You will apply the suggested fix. The symptom may disappear. The root cause may remain, dormant until a different execution sequence triggers it again. You have not fixed the bug. You have moved it.

---

## Reading Code As A Causal System

The deeper skill this module is building — underneath the workflow, underneath the tool — is reading code as a causal system.

Every Java program is a sequence of state transformations. An object starts in some initial state. Methods are called. Fields are modified. References are reassigned. Collections are updated. At the end, some final state exists and produces some output. Every visible output is the consequence of every state transformation that preceded it. A bug is a state transformation that went wrong.

Reading code this way means asking, for every operation: what state does this touch? What was the state before? What is the state after? Is the after-state correct? This is not how you read code when you are writing it — when you are writing, you think about intent. This is how you read code when you are debugging — you think about what actually happened, not what you meant to happen.

The gap between intent and execution is where bugs live. You meant to copy a reference. You shared it. You meant to reset a variable. You reused it. You meant to check a precondition. You assumed it. Every one of these gaps is a place where your mental model of the program — what you thought the code does — diverged from the program's actual behavior.

The debugging workflow closes that gap by forcing you to make the mental model explicit. Your hypothesis is a statement about your mental model: I believe the state at this point is X. The debugger tests that belief. Where the belief is wrong, the mental model is wrong. Correcting the mental model is the learning. The fix to the code is almost incidental.

This is why debugging is not a remedial skill. It is not what you do when something goes wrong. It is a primary method for understanding programs that someone else wrote, programs that have grown beyond what you can hold entirely in your head, and programs that behave differently from your expectations. Every working software engineer debugs constantly — not because their code is bad, but because the gap between intent and execution is always present and always requires attention.

---

## The Module's Single Capability

Every module in this course adds exactly one concrete capability to the semester project. This module's capability is: debugging as causal diagnosis rather than output patching.

In practical terms, that means: when the project produces wrong output, you form a hypothesis before you change a line. You use the debugger to isolate evidence. You trace from symptom to proximate cause to root cause. You fix the root cause. You verify that the fix works not by checking whether the symptom is gone but by confirming that the state is now correct at the point where it was wrong.

This is different from the natural instinct, which is to change things until the output looks right. The natural instinct is not wrong because it is impatient. It is wrong because it does not produce understanding. A program fixed by changing things until the output looks right is a program whose bugs are invisible until the next execution sequence finds them. A program debugged by tracing causes is a program whose behavior you understand.

By the end of this module, you should be able to take a program with hidden wrong behavior — output that is incorrect but not obviously broken — and work through three documented steps: a written hypothesis naming the object, field, and moment where you expect the state to be wrong; a screenshot or trace showing the debugger's evidence at the breakpoint; and a written explanation of the root cause and why the fix addresses it rather than masking it.

When you can do that for the library checkout bug, you can do it for the inventory discrepancy and the scheduling conflict, because the workflow is the same. The domain changes. The objects get new names. The causal structure is different. But hypothesize, isolate, test follows the same logic in every Java program that has ever existed.


---

## What You Are Building Toward

Module 1 gave you the object model: business things with state and behavior. Modules 2 and 3 gave you collections of objects and the relationships between them. This module gives you the ability to find where the model breaks — where the state of an object diverges from the requirement, quietly, without announcing itself.

The next question the course asks is a design question: how do you build a system that has fewer hidden bugs in the first place? Not zero — no system has zero bugs — but fewer, and more findable. The answer involves writing code in a way that makes state transitions legible, that limits the places where shared references can be corrupted, and that makes preconditions explicit rather than assumed.

That question is Module 5's subject. But you cannot think about it clearly until you have spent time with a program that hides its bugs well, traced the hiding back to a specific design choice, and felt the difference between fixing the output and fixing the cause. That is what the lab for this module is for.

The bridge question is: you can debug one class. How do you design a system with fewer hidden bugs? Hold that question. You have everything you need now to start forming an answer.

---

## Key Terms

- **symptom** — the observable wrong output or behavior; what the bug announces, not what the bug is.
- **hypothesis** — a falsifiable claim naming the object, field, and moment where you expect the state to be wrong; the precondition for productive debugging.
- **isolation** — the act of creating conditions under which the hypothesis can be confirmed or refuted; in Java, typically achieved with a breakpoint.
- **breakpoint** — an instruction to the runtime to pause execution at a specific line and transfer control to the debugger; the primary tool for observing intermediate state.
- **step over** — a debugger navigation command that executes the current line and pauses at the next, without descending into method calls; use when the bug is not inside the called method.
- **step into** — a debugger navigation command that, when the current line calls a method, pauses at the first line inside that method; use when the bug may be inside the called method.
- **root cause** — the design decision, state error, or logic flaw that made the wrong behavior possible; upstream from the proximate cause, upstream from the symptom.

---

## Exercises

**Warm-up**

1. The library checkout program produces wrong output on the second checkout but correct output on the first. List three specific hypotheses — each naming an object, a field, and a moment in execution — that could explain this sequence-dependent behavior. For each hypothesis, write one sentence describing what breakpoint you would set to test it. Do not run the program yet. *(Tests: forming falsifiable hypotheses before touching the debugger; translating a symptom into candidate causes.)*

2. Classify each of the following as a symptom, a proximate cause, or a root cause: (a) the output shows Book B under Patron A's name; (b) the assignment `loan.setPatron(currentPatron)` executes with the wrong reference; (c) `currentPatron` was never reassigned after the first checkout because the reset call was inside an `if` branch that only runs when the patron is new. For each, write one sentence explaining how you decided which level it belongs to. *(Tests: applying the symptom / proximate cause / root cause taxonomy to a concrete case.)*

3. Without running any code, describe the difference between what you would see if you set a breakpoint and used step over versus step into when the paused line is `patron.borrow(book)`. Under what hypothesis about the bug's location would you choose step over? Under what hypothesis would you choose step into? *(Tests: mapping debugger navigation choices to hypotheses about where the bug lives.)*

**Application**

4. Open the library checkout program. Introduce a bug of your own: modify one line so that the program produces wrong output without throwing an exception. Do not tell anyone what you changed. Then write up a full debugging session as if you had inherited this code: your hypothesis, your breakpoint, what the debugger showed, and your identification of the root cause. The write-up must reference specific variable names and field values from the debugger's variables panel. *(Tests: applying the full hypothesize–isolate–test loop on a known but unfamiliar bug; reading intermediate state against a prediction.)*

5. Translate the sequence-dependent bug scenario to your chosen domain. In inventory: describe a scenario where the wrong item gets decremented when two transactions happen in quick succession. In scheduling: describe a scenario where a patient gets assigned to a provider who is already booked, but only on the second scheduling attempt. For your scenario, write one hypothesis and name the Java artifact — class, field, and method — where you would set the first breakpoint. *(Tests: transferring the debugging workflow to a new domain; identifying the causal structure in a different object model.)*

6. Write an AI prompt that complies with this module's boundary — it presents a hypothesis and asks whether the hypothesis is plausible — and a second prompt that violates the boundary by asking AI to identify the root cause directly. Run both prompts. In one paragraph, describe what was different about the two responses and whether the compliant prompt produced more or less useful output. *(Tests: applying the AI boundary in practice; evaluating what hypothesis-first prompting actually changes about AI's response.)*

**Synthesis**

7. A classmate fixes the sequence-dependent bug by changing the method call order so the symptom no longer appears on their test cases. They submit without setting a breakpoint or writing a hypothesis. Write a two-paragraph response: the first paragraph describes a test scenario that would reveal whether the root cause was actually fixed or merely masked; the second paragraph explains what evidence they would need to provide before you would accept their fix as correct. *(Tests: distinguishing symptom removal from root cause repair; applying the verification standard from the module.)*

8. The debugging workflow (hypothesize, isolate, test) and the scientific method share a structure. Identify one way in which debugging a Java program is easier than running a controlled experiment in the physical world, and one way in which it is harder. Your answer should be grounded in specific properties of the Java runtime and the debugger, not in general analogies. *(Tests: integrating the module's conceptual framework with a broader understanding of empirical reasoning; reasoning about what the debugger does and does not give you.)*

**Challenge**

9. The module's bridge question asks: how do you design a system with fewer hidden bugs? Before reading Module 5, write your best current answer. What properties would a Java class need to have so that sequence-dependent bugs like the one in this module are structurally impossible — not just unlikely? Your answer should name at least one specific Java language feature or design pattern and explain the mechanism by which it prevents the class of error you diagnosed in this module. There is no single correct answer; the point is to make your current thinking explicit so you can test it against what Module 5 actually teaches. *(Points toward immutability, defensive copying, and the design choices that limit shared mutable state.)*

---

## Bridge

You can debug one class. How do you design a system with fewer hidden bugs?

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Optional Exercise: Array of Objects | Practice | For-Credit | Individual | Create and process collections of object references. |
| Optional Exercise: Passing Objects to Methods | Practice | For-Credit | Individual | Trace object references through method calls. |
| Optional Exercise: Integer Objects | Practice | For-Credit | Individual | Practice wrapper class behavior. |
| Optional Exercise: Parsing Strings into Numbers | Practice | For-Credit | Individual | Convert string input into numeric values safely. |
| Optional Exercise: Writing a Method Using Strings | Practice | For-Credit | Individual | Use string operations inside a reusable method. |
| Quiz - Basics of Object-Oriented Programming Part 2 | Summative | For-Credit | Individual | Assess object thinking, wrappers, and string handling. |
| Optional Lab - File I/O and Class Definition | Practice | For-Credit | Individual | Combine class definitions with file input/output. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and Java SE API documentation — use for language and library facts.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming" — use for common novice difficulties and why some debugging errors are harder to see than others.
- Peng et al. and Vaithilingam et al. — use for cautious, empirically grounded claims about AI coding assistance and the verification risks that attend it.

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
1. Oracle. Class Object. Java SE 21 API Specification, 2023. https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html
