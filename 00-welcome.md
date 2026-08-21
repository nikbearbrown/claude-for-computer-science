# Module 0: Welcome
*The first thing you verify is whether your tools are telling you the truth.*

There is a particular kind of confusion that happens before you have written a single line of Java. The environment fails — not the code, the environment — and the error message looks exactly like a programming error. You spend forty minutes reading Stack Overflow about `NullPointerException` when the actual problem is that your IDE is pointing at a Java runtime that has no compiler. You are debugging a phantom. The bug is not in your program. You have not written a program yet.

I want to start here because this confusion is not a beginner's embarrassment. It is structural. Setup failures and programming failures produce nearly identical symptoms, and the only way to tell them apart is to know what a working environment looks like before you go looking for what went wrong inside one.

That is what this module is: a way of knowing what working looks like.

---


## EDGE Course Outline Alignment

**Module 0: Welcome**

### Learning Objectives (LOs)

- Identify the course structure, communication channels, and academic integrity expectations.
- Introduce yourself to the learning community and state your collaboration preference.
- Locate the syllabus, faculty contact information, and required course resources.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- Welcome and course introduction
- Faculty Bio & Contact
- Syllabus
- Academic Integrity
- Meet your fellow learners

## What We Are Actually Doing

Most textbooks treat setup as ceremony. Install this, click that, move on to the real work. I think that framing is wrong, and it creates a specific kind of trouble.

When setup is ceremony, students skip the verification steps — not because they are careless, but because the book gave them the impression that verification was optional housekeeping. They assume the environment works, write their first program, hit an error, and have no way of knowing whether the problem is their Java or their toolchain. Everything is suspect. The diagnostic surface is enormous.

Setup done right shrinks that surface before the semester starts.

Here is the concrete thing I want you to be able to do by the end of this module: run two commands and read the output of both, create a project in NetBeans, run Hello World, then find the source file and the compiled output on disk. That is the whole checklist. It sounds small. It is not small. A student who can do all of that without uncertainty has separated their environment from their programming problem space. Every lab after this one starts from a smaller, cleaner diagnostic surface.

---

## The Machinery

Let me explain what is actually happening when Java runs, because the setup confusion usually traces back to not knowing this.

When you write Java, you write in a language humans can read. The computer cannot run that language directly. Before your program can execute, it has to be translated — *compiled* — into something the machine understands. That translation is the job of the Java compiler, which is called `javac`. The result of compilation is a `.class` file containing bytecode: a representation that is not quite machine code, but is much closer to it than the source you wrote.

Then there is a second step. That bytecode does not run directly on the hardware either. It runs inside a piece of software called the Java Virtual Machine, which reads the bytecode and executes it. The JVM is what makes Java programs portable — the same bytecode runs on any machine that has a JVM installed, regardless of whether that machine is Windows or Mac or Linux.

Now here is why this matters for setup. The tool that contains the compiler — `javac` — is part of a package called the **JDK**, the Java Development Kit. The tool that contains only the runtime — the JVM — is part of a smaller package called the **JRE**, the Java Runtime Environment. If you install a JRE but not a JDK, you can *run* Java programs that someone else compiled. You cannot compile your own. NetBeans needs a JDK. A student who accidentally installed only a JRE will see errors that look like programming failures when they are actually missing-compiler failures.

This is the machine you are about to verify exists and is correctly assembled.


---

## Verification as a Habit, Not a Checklist

I could give you a checklist. But I want to give you something more durable: a way of thinking about verification that will serve you for the whole semester.

Here is the principle. A working environment is not a belief. It is *evidence*. You do not assume Java is installed correctly because the installer said "Setup Complete." You assume nothing. You run a command and look at what it prints.

The two commands are:

```bash
java --version
javac --version
```

Run both. Look at both outputs. They should report the same version, or very close to the same version. If `java --version` returns a version number but `javac --version` says "command not found," that is the JRE-without-JDK problem I just described. If both return version numbers but they differ significantly — say, one says 11 and one says 17 — your system has multiple Java installations and NetBeans may be pointing at the wrong one.

Notice what we are doing here. We are not trusting the installation. We are asking the system to produce output we can inspect. The output is the evidence. This is a habit.

The same habit applies after you create your first project. You do not assume it worked. You find the source file on disk. You find the compiled `.class` file. You identify the command NetBeans is running when it hits the Run button. You can describe, without looking at the IDE, where your program lives and what runs it.

That is not paranoia. That is engineering.


---

## The Specific Failure This Module Is Designed to Prevent

Let me describe the failure mode precisely, because understanding it is most of the protection against it.

A student installs Java. They open NetBeans. They create a project. They paste Hello World — or ask an AI to generate it — and click Run. It works. They believe setup is done.

Three weeks later, in a lab on object-oriented design, they create a new project and see a different error. They spend an hour debugging their class definition. The actual problem: their second project used a different project template than their first, and the template difference produces a structure NetBeans handles differently. They do not know this because they never learned what the first project's structure actually was.

Or: they copy a AI-generated snippet into their working project, and the AI used a version-specific API that is not available in the JDK their course requires. The program no longer compiles. They debug the class they wrote. The problem is the class they copied.

Both failures have the same root. The student trusted the output without understanding what produced it. They did not know what their environment looked like, so they could not see when it changed.

The habit we are building is knowing what it looks like. That way, when it changes — and it will change — you notice.


---

## The First Project

Here is what the first project actually establishes, step by step.

You open NetBeans and create a new Java Application project. NetBeans will ask for a project name and location. Use something you can find: a folder on your desktop, inside a clearly named directory. Do not let NetBeans pick a default location buried three directories deep. You will need to find this folder.

After the project is created, find the main class. NetBeans will have created it for you. Look at its structure: there is a package declaration at the top, a class definition, and inside the class, a method called `main`. The `main` method is the entry point — when you run the program, the JVM starts here. Every Java application has exactly one entry point. Know where yours is.

Run the program. NetBeans will compile and execute it. Look at the output panel at the bottom of the IDE. You should see whatever the `main` method prints — for Hello World, that is the string `Hello World!` followed by a newline.

Now close the IDE and navigate to your project folder in your file explorer. Find the `src` directory: this is where your source files live. Find a folder called `build` or `target` or `dist` — the name depends on the project type — and inside it, find a folder called `classes`. In that folder, find a file with the `.class` extension. That is the compiled bytecode for your program. Touch it. Know it exists. You built it.


This is the verification checklist for Module 0:

1. `java --version` and `javac --version` both report the same supported version.
2. A NetBeans Java Application project exists and has a known name and location.
3. The `main` method can be identified in the main class.
4. Running the project produces visible output in the output panel.
5. The source file and compiled `.class` file can be located on disk.

Five items. When all five are checked, setup is done. Not assumed — done.

---

## Where AI Fits, and Where It Does Not

I want to be direct about this because the boundary matters and it is easy to get wrong.

AI can help you with setup. Specifically, AI is good at two things in this module: explaining error messages you do not understand, and translating unfamiliar terminology into language you can work with. If NetBeans shows you an error you have never seen before, pasting that error into an AI and asking "what does this mean and what are the likely causes?" is a legitimate use of the tool.

What AI cannot do — what you should not ask it to do — is choose a different IDE, rewrite the project structure, or tell you what version of the JDK your course requires. Those decisions are not the AI's to make. The IDE and version are set by the course. If an AI suggests you switch to IntelliJ or install a different Java version, that suggestion is outside its authority here, and acting on it will create downstream incompatibilities that are very hard to diagnose.

The test for whether an AI suggestion is appropriate is simple: does acting on this suggestion require you to verify something you can actually verify? If the AI says "your error is likely caused by a JRE-only installation — run `javac --version` to check," that is a good suggestion because it gives you something to inspect. If the AI says "try using a Maven project instead of a NetBeans Application project," that requires a design decision that belongs to the course structure, and the AI is operating outside what it can know.

There is a deeper principle here that runs through the whole book. Using AI is fine. *Unverified delegation* is the problem. When you ask AI to diagnose your error and then run the diagnostic yourself to confirm, you have used AI correctly. When you ask AI to fix the error and then paste whatever it gives you without understanding what changed, you have outsourced your engineering judgment. The output might work. You will not know why, and you will not know when it breaks.

| task | AI-appropriate? | what the student must still do |
| --- | --- | --- |
| explaining an error message, choosing IDE, translating a term, selecting JDK version, scaffolding a diagnosis, verifying the fix works, generating Hello World, confirming the artifact matches the requirement | The pattern becomes easy to misuse or overlook. | A concrete checkpoint for applying the chapter concept. |

Here is a prompt that keeps the boundary where it belongs:

```text
I am working on Module 0: Setup in INFO 5100.
My operating system is [OS and version].
My JDK version is [version from java --version].
My NetBeans version is [version].
The full error message is: [paste the exact message].
Help me understand what this error means and what the most likely causes are.
Do not suggest switching to a different IDE or a different JDK version.
```

Notice what that prompt does. It gives the AI the information it needs to be useful — exact versions, exact error — and it explicitly closes off the decisions that are not the AI's to make. The AI's job is explanation. The verification and the decision remain yours.

---

## The Semester Project Shape

This course is built around a single project that grows across fifteen modules. Each module adds one capability. The domain you choose at the start — a library system, an inventory tracker, or a healthcare scheduling application — stays consistent through the semester.

This matters for setup in a specific way. The project structure you establish in Module 0 is the structure you carry forward. If you build it wrong — wrong project template, wrong package structure, wrong file locations — every subsequent module will carry that mistake. It is much harder to fix a structural problem in Module 6 than to get it right in Module 0.

That is why the verification checklist exists. Not because the checklist is interesting. Because the checklist is the thing that confirms the foundation is sound before we build on it.

The library example runs through the worked material in this book. If your domain is inventory or healthcare scheduling, the setup steps are identical — same JDK check, same project template, same verification checklist. The domain affects what you name your classes and what your `main` method eventually does. It does not affect the toolchain.

| concept | Java artifact that carries it | what AI may do | what the student must verify | evidence required before submission |
| --- | --- | --- | --- | --- |
| JDK installation, project creation, main class, compilation, runtime execution | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## The Diagnostic Habit in Practice

One more thing before the lab. I want to give you a way of thinking about errors that will serve you past Module 0.

When something fails, the first question is not "what is wrong with my code?" The first question is "what layer is this failure in?" There are three layers in a Java development environment, and they fail in different ways.

The *toolchain layer* is the JDK, the JVM, and the PATH configuration that lets your terminal find them. Failures here produce "command not found" errors, version mismatches, and compile errors that appear before you have written any interesting code.

The *project layer* is the structure NetBeans creates: the directories, the build configuration, the project template choices. Failures here produce "class not found" errors at runtime, unexpected package structures, and behaviors that differ between projects even when the code looks the same.

The *program layer* is your Java source code. Failures here produce compile errors about syntax and type, and runtime errors about logic and state.

Most setup failures live in the toolchain or project layer. Most programming failures live in the program layer. The diagnostic habit is knowing which layer to look in before you start looking.

When a student says "Java is broken," they almost always mean something in one of these three layers is misconfigured. Knowing which one cuts the debugging time dramatically. The verification checklist from this module gives you a clean toolchain and a correct project layer. After that, when something breaks, you can be much more confident about where to look.


---

## What This Module Does Not Settle

I want to be honest about what we have not done yet.

We have established that the environment works. We have not established anything about the programs we are going to write in it. We have confirmed that `javac` exists and produces `.class` files. We have not said anything about what Java is modeling, or how a Java program represents real-world things, or why the language makes the choices it makes.

Those questions start in Module 1, when the project moves from Hello World to the first class definition for your domain.

But I want to leave you with the bridge question, because it is the right question to be carrying when you close this module: once the toolchain works, what does a Java program actually model?

Think about what Hello World does. It has a class. It has a method. It has a string literal. It has an output destination. Even the simplest program in Java has structure. That structure is not arbitrary. It reflects a set of decisions the language designers made about how programs should be organized, and what concepts they should be organized around.

Module 1 is where we begin to understand those decisions. Setup is what lets us get there without losing a lab session to phantom errors.

---

## Lab: Setup Evidence Packet

The lab for this module is not a programming assignment. It is an evidence packet. Submit the following:

1. The output of `java --version` and `javac --version`, copied from your terminal.
2. A screenshot or transcript showing Hello World running in NetBeans.
3. The file path of your source file and the file path of the compiled `.class` file.
4. If you used AI for any part of setup — error diagnosis, term explanation, anything — a one-paragraph disclosure: what you asked, what the AI said, what you verified yourself, and what the AI got wrong or incomplete.

The fourth item is not punitive. It is practice for the AI disclosure habit the course requires across all fifteen modules. The goal is not to avoid using AI. The goal is to be able to describe, precisely, what the AI did and what you did.

A complete evidence packet is not proof that you are a Java programmer. It is proof that your toolchain is sound and that you know what your environment looks like. That is the right foundation.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Exercises

**Warm-up**

1. Run `java --version` and `javac --version` on your machine. Write down both version strings exactly as they appear. If they differ, name which layer of the three-layer model the discrepancy lives in and what the likely cause is. *(Tests: toolchain verification habit; three-layer model.)*

2. Without opening NetBeans, navigate to your project folder in your file explorer and write down the full path to your `.java` source file and your compiled `.class` file. If you cannot find one of them, return to the First Project section and identify which verification step you skipped. *(Tests: project layer verification; source vs. compiled artifact distinction.)*

**Application**

3. A classmate tells you their program "works fine" and shows you console output. You ask them to close NetBeans and find the `.class` file on disk. They cannot. Using the three-layer model, explain: (a) which layer is unverified, (b) what specific evidence is missing, and (c) what failure this leaves them exposed to in a later module. *(Tests: three-layer model applied to a realistic scenario; distinction between running and verified.)*

4. You paste a setup error into an AI and receive two suggestions: (a) "run `javac --version` to confirm the JDK is installed," and (b) "switch to a Maven project for better dependency management." Apply the AI boundary test from this module to each suggestion. Which is appropriate here, which is not, and why? *(Tests: AI boundary rule; verifiable-output test.)*

5. Write the diagnostic prompt you would use if NetBeans opened your project and showed the error: `error: package does not exist`. Include all fields the prompt template requires. Do not ask the AI to fix the project structure. *(Tests: diagnostic prompt construction; AI boundary in practice.)*

**Synthesis**

6. Suppose you complete Module 0 using a JRE instead of a JDK, and the error does not appear until Module 3 when you try to add a custom class. Trace the failure: which layer failed in Module 0, why it was invisible until Module 3, and what the Module 0 verification checklist item would have caught it. *(Tests: three-layer model; verification checklist as forward error prevention; JDK vs. JRE distinction.)*

7. A student in a different section used an AI to generate their entire Module 0 lab submission — version strings, file paths, and AI disclosure paragraph — without running any of the actual commands. Their submission passes a surface reading. Write a two-paragraph explanation of what they have and have not established, using the language of evidence vs. belief from this module. *(Tests: evidence-based verification principle; what the lab is actually certifying.)*

**Challenge**

8. The three-layer model in this module (toolchain / project / program) is a simplification. Identify at least one failure mode that does not fit cleanly into a single layer — for example, a failure that requires both the project layer and the toolchain layer to be in a specific state simultaneously. Describe the failure, explain why it crosses layers, and propose a verification step that would catch it before it manifests as a programming error. *(Open-ended; points toward the interaction effects that appear in later modules when the project grows in complexity.)*

---

## Key Terms

- **JDK** — the Java Development Kit: the package that includes the compiler (`javac`), the runtime (`java`), and the supporting tools for Java development. This is what you install for this course.
- **JRE** — the Java Runtime Environment: the subset of the JDK that includes only the runtime, not the compiler. You can run Java programs with a JRE but you cannot compile them. Installing a JRE instead of a JDK is the most common setup error.
- **compiler** — the program (`javac`) that translates your source code into bytecode. Compilation happens before execution. Compilation errors are about form and type. Runtime errors are about behavior.
- **IDE** — Integrated Development Environment: the software (NetBeans, in this course) that combines an editor, a compiler, and a runner into a single interface. The IDE does not change what Java does; it makes the development workflow faster.
- **project** — the NetBeans unit that organizes your source files, build configuration, and output. The project structure matters because it tells NetBeans where to look for things. Getting this structure right in Module 0 prevents structural problems in later modules.
- **source file** — the `.java` file you write. Human-readable. Cannot run directly.
- **diagnostic prompt** — a prompt to an AI that includes enough context (OS, versions, exact error message) to get a useful explanation, while explicitly excluding decisions that belong to the course structure.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Academic Integrity Attestation | N/A | For-Credit | Individual | Confirm understanding of course integrity expectations before beginning technical work. |
| Group preference survey | Community | For-Credit | Individual | Collect collaboration preferences for later course activities. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and the Java SE API documentation: authoritative sources for language and library facts. Use when the behavior you are seeing does not match what you expect.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": research on where novice programmers get stuck and why. The setup confusion this module addresses is documented there.
- Peng et al. and Vaithilingam et al.: empirical work on AI coding assistance and the verification risks that attend it. The AI boundary rules in this module are grounded in this literature.

*Current tool instructions, version-specific setup steps, and AI platform behavior require pre-offering verification.* [verify]
<!-- FACT-CHECK FLAG: UNVERIFIED — see factchecks/00-welcome-assertions.md -->

![A nested rectangle composition. Outer rectangle (JDK): Blue outlined 1pt, no fill, taking up approximately 70% of the panel area. Inside the outer rectangle, in its lower-left quadrant, an inner rectangle (JRE): Sky...](images/00-welcome-fig-01.png)
*Figure 0.1 — JDK vs. JRE Containment*


![A horizontal flow. From left to right: a small rectangle (HelloWorld.java) Sky Blue filled, a Black 1pt arrow labeled implicitly *javac*, a rectangle (HelloWorld.class) Blue filled, a Black 1pt arrow labeled...](images/00-welcome-fig-02.png)
*Figure 0.2 — Java Toolchain Pipeline*


![A two-column composition. Left column header band (blank, for "Belief"); right column header band (blank, for "Evidence"). Below each header, a vertical stack of four small indicator rectangles. Left column (Belief...](images/00-welcome-fig-03.png)
*Figure 0.3 — Belief-Based vs. Evidence-Based Verification*


![A vertical tree diagram. Root node at top center: a Black 1pt outlined rectangle labeled (in post) "MyProject". Two branches descend to two child folder nodes side by side: src/ on the left (Sky Blue filled...](images/00-welcome-fig-04.png)
*Figure 0.4 — NetBeans Project Folder Tree*


![A vertical stack of three horizontal bands of varying widths. Bottom band (widest, foundation): Blue filled, full panel width — the toolchain layer (JDK, JVM, PATH). Middle band (medium width, centered): Sky Blue...](images/00-welcome-fig-05.png)
*Figure 0.5 — Three-Layer Diagnostic Model*

---

## References

<!-- Fact-check pass references. -->
1. Oracle. The javac Command. Java SE 21 / JDK 21 Tool Guides, 2023. https://docs.oracle.com/en/java/javase/21/docs/specs/man/javac.html
2. Oracle. The Structure of the Java Virtual Machine. Java Virtual Machine Specification, Java SE 21, 2023. https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html
3. Oracle. The java Command. Java SE 21 / JDK 21 Tool Guides, 2023. https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html
