# Module 6: Basics of GUI Programming in Java
*The file gets read. What can they do with it?*

Two login systems pass the demo. Both accept a username and password. Both reject incorrect credentials. Both compile cleanly and run without errors. One of them is a security failure waiting for an attacker to show up.

The difference is not visible in the running application. It is visible only in the file where the credentials are stored. One system writes the password directly: `alice:password123`. The other writes a hash: `alice:ef92b778bafe771e89245b89ecbc08a44a4e166c`. If someone gains read access to that file — through a misconfigured server, a backup left in a public directory, a logging error that writes too much — the first system has handed them every user's password. The second system has handed them something they have to work hard to reverse, and for most purposes, cannot.

The question I want you to carry into this module is not "which one compiles?" It is: if someone reads my user database, what can they do?

That question is a threat model. It is a small one, appropriate for a course project. But the habit of asking it before writing authentication code is the whole point of this module. AI can generate a login system in seconds. It can even generate one that hashes passwords. What AI cannot do is ask your threat-model question and answer it in the context of your specific application, your specific data, and the specific people whose credentials you are about to store.

---


## EDGE Course Outline Alignment

**Module 6: Basics of GUI Programming in Java**

### Learning Objectives (LOs)

- Recall the fundamental characteristics of each GUI toolkit and articulate the primary distinctions among JavaFX, Swing, and AWT.
- Demonstrate the ability to configure a programming environment for JavaFX and apply basic programming skills to write and execute simple JavaFX applications.
- Create user interfaces using pages, groups, UI controls, and shapes.

### Applied Industry Skills

- Applied Industry Skills

### Topics / Lessons / Material

- GUIs
- JavaFX
- Panes, UI Controls, and Shapes
- Property Binding
- Layout Panes
- Shapes
- Polygon and Polyline
- Scene builder

## The Person Is Not the Account

Before the machinery, there is a design decision that most beginners get wrong, and getting it wrong creates problems that multiply across every subsequent module.

The mistake is conflating the person with the account.

In any application with users, there are two distinct things that need modeling. The first is the human participant in the business domain: the library patron who borrows books, the warehouse employee who manages inventory, the clinic patient who has appointments. This person has a name, an ID, a history of transactions. They are a participant in the business process. Their record exists because the business needs to track what they do.

The second thing is the credential: the mechanism by which the application verifies that someone claiming to be Alice is actually Alice. This is a username, a password or its hash, and the logic for checking that a supplied credential matches a stored one.

These two things are related — a `UserAccount` is associated with a `Person` — but they are not the same thing. A `Person` record exists because the business needs it. A `UserAccount` exists because the system needs to control access. They have different lifecycles: a patron record persists long after the patron stops using the system; a `UserAccount` might be suspended or deleted while the patron record remains. They have different sensitivity levels: a patron's name and borrow history are business data; a password hash is a security artifact that requires different handling.

When you mix them — when the `Patron` class has a `passwordHash` field, or when the login logic reads directly from the `Person` object — you create a coupling that makes both objects harder to reason about. You also create a security surface problem: now every piece of code that handles `Patron` objects is potentially touching authentication data.

The design move this module introduces is keeping them separate: a `Person` (or `Patron`, or `Employee`, or `Patient`) for the domain participant, and a `UserAccount` for the credential. A `LoginManager` that takes a username and password, checks the credential against the stored hash, and returns whether access is granted. The `LoginManager` does not become the `Person` record. It does not return the `Person` object. It answers one question: are these credentials valid?

![Conflated vs. separated credential design rendered as a comparison panels for the chapter concept.](images/06-basics-of-gui-programming-in-java-fig-01.png)
*Figure 6.1 — Conflated vs. separated credential design*


---

## What a Hash Actually Does

Let me explain hashing precisely, because the word gets used loosely and the imprecision leads to real mistakes.

A hash function takes an input of arbitrary length and produces an output of fixed length. The output — the hash — has two properties that matter for passwords. First, it is deterministic: the same input always produces the same output. Second, it is one-way: given the hash, you cannot reverse the function to recover the input. You can verify that a supplied password matches a stored hash by hashing the supplied password and comparing the two hash values. You cannot go from the stored hash back to the original password.

This is why storing hashes instead of plaintext passwords limits the damage of a file read. An attacker who reads `alice:ef92b778bafe771e89245b89ecbc08a44a4e166c` cannot directly recover Alice's password. They can try to reverse it by brute force — computing hashes of common passwords and looking for matches, a technique called a dictionary attack — but that requires computational effort and is limited by the quality of Alice's password.

The Java standard library provides `MessageDigest` for computing cryptographic hashes. For a course project, SHA-256 is a reasonable choice: it is widely available, produces a 256-bit output, and is not reversible by any known efficient algorithm. Here is the basic pattern:

```java
import java.security.MessageDigest;
import java.nio.charset.StandardCharsets;

public static String hash(String password) throws Exception {
    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    byte[] encoded = digest.digest(
        password.getBytes(StandardCharsets.UTF_8)
    );
    StringBuilder hex = new StringBuilder();
    for (byte b : encoded) {
        hex.append(String.format("%02x", b));
    }
    return hex.toString();
}
```

The method takes a password string, computes its SHA-256 hash, and returns the hash as a hexadecimal string. You call it once when creating a `UserAccount` to store the hash. You call it again at login, hash the supplied password, and compare the result to the stored hash.

![Hash function mechanism rendered as a process flowchart for the chapter concept.](images/06-basics-of-gui-programming-in-java-fig-02.png)
*Figure 6.2 — Hash function mechanism*


There is one caveat worth naming even at this level: SHA-256 without salting is vulnerable to precomputed lookup tables (rainbow tables) for common passwords. For production systems, you would use a purpose-built password hashing function like bcrypt or Argon2, which incorporate a random salt and are deliberately slow to compute. This module uses SHA-256 because the point is the design habit — separate the credential from the person, hash the password, ask the threat-model question — not the production-hardening details. The further reading section points to where those details live.

---

## The Object Structure

With the design principle and the hashing machinery in place, here is how the objects fit together.

`Person` (or the domain-specific equivalent — `Patron`, `Employee`, `Patient`) models the business participant. It knows nothing about credentials. Its fields are the fields the business process needs: name, ID, and whatever domain-specific data the module requires.

```java
public class Patron {
    private String patronId;
    private String name;
    private List<Book> borrowedBooks;

    // constructor, getters, domain methods
}
```

`UserAccount` models the credential. It knows the username, the stored password hash, and — if the design requires it — the patron ID that connects this account to a `Patron` record.

```java
public class UserAccount {
    private String username;
    private String passwordHash;
    private String patronId; // links to Patron, does not embed it

    public UserAccount(String username, String password, String patronId)
            throws Exception {
        this.username = username;
        this.passwordHash = hash(password);
        this.patronId = patronId;
    }

    public boolean checkPassword(String supplied) throws Exception {
        return hash(supplied).equals(this.passwordHash);
    }
}
```

Notice that `UserAccount` stores a `patronId` string, not a `Patron` object reference. This is deliberate. If `UserAccount` held a `Patron` reference, every piece of code that touches authentication would also have access to the patron's borrowing history, and the separation of concerns would erode. The `patronId` is a key — you use it to look up the `Patron` separately, after authentication succeeds.

`LoginManager` holds the account directory and performs the credential check. A `HashMap<String, UserAccount>` keyed by username is the appropriate structure: lookups by username are the only query this component needs to perform, and `HashMap` gives O(1) average-case lookup.

```java
public class LoginManager {
    private Map<String, UserAccount> accounts = new HashMap<>();

    public void addAccount(UserAccount account) {
        accounts.put(account.getUsername(), account);
    }

    public boolean login(String username, String password) throws Exception {
        UserAccount account = accounts.get(username);
        if (account == null) return false;
        return account.checkPassword(password);
    }
}
```

The `login` method returns a boolean. It does not return the `UserAccount`. It does not return the `Patron`. It answers one question: are these credentials valid? If the answer is yes, the calling code is responsible for deciding what the authenticated user is allowed to do next.

| concept | Java artifact | what AI may do | what the student must verify | evidence before submission |
| --- | --- | --- | --- | --- |
| with | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## The Threat Model, Made Concrete

The phase gate for this module requires you to answer one question before writing any code: if someone reads my user database, what can they do?

For the plaintext system, the answer is: read every user's password. Because people reuse passwords, that may give the attacker access to other accounts — email, banking — that have nothing to do with your library application. The damage extends far beyond your application.

For the SHA-256 hashed system, the answer is: attempt a dictionary attack against the hashes for common passwords. If a user's password is "password123" or "letmein," it will be found quickly. If the password is long and random, the attacker will not find it in any practical timeframe. The damage is bounded by the quality of each user's password.

This is what a threat model does: it converts a vague concern ("security is important") into a specific question with a specific answer. The answer tells you what properties your implementation must have, and lets you verify that the implementation achieves them.

For this module, the threat model has three rows:

| Threat | Plaintext system | Hashed system |
|--------|-----------------|---------------|
| File read | All passwords exposed immediately | Dictionary attack required; strong passwords safe |
| Application compromise | Attacker logs in as any user | Same as file read |
| Credential reuse | Other accounts at risk | Damage limited to weak passwords |

Write your own version of this table for your domain before writing any authentication code. The table is the spec. The code implements the table.

| threat scenario | what the attacker gains | what the plaintext design exposes | what the hashed design exposes | what would make the hashed design stronger |
| --- | --- | --- | --- | --- |
| for their own domain before writing authentication code | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A specific, evidence-linked version that readers can verify. |

---

## What AI Can and Cannot Do Here

AI is capable of generating a complete, syntactically correct, functioning login system. It will hash passwords. It will use a `HashMap`. It may even separate `Person` from `UserAccount`. The output will look like the code in this module.

That is exactly what makes the boundary important.

AI generates authentication code based on what authentication code typically looks like. It does not generate authentication code based on the threat model for your specific application, the sensitivity of the data your users are trusting you with, or the specific failure modes that matter in your domain. A healthcare scheduling application that stores appointment data has different exposure than a library book tracker. The authentication code may look identical. The implications of a credential exposure are not.

The phase gate exists to put the threat-model reasoning before the code generation. When you have answered "if someone reads my user database, what can they do?" in writing, AI can scaffold the `HashMap`-based directory and explain the `MessageDigest` operations. At that point AI is implementing a spec you have already produced. Without the threat model, AI is producing a spec and an implementation simultaneously, and you have no way to evaluate whether the spec is correct.

Here is the prompt that keeps the boundary where it belongs:

```text
I am working on Module 6: Basics of GUI Programming in Java in INFO 5100.
My domain is [library / inventory / scheduling].
My threat model is: [paste your completed threat-model table].
My current object design is: Person has [fields], UserAccount has [fields].
Help only within this boundary: AI may scaffold a Map-based account
directory and explain HashMap operations. It may not generate
authentication logic before the threat model exists.
```

The prompt works because the threat model is already written before AI is involved. AI receives it as input, not as something it needs to produce.

---

## The Inventory and Scheduling Shape

The library domain uses `Patron` and `UserAccount`. The same design applies elsewhere.

In an inventory system, the domain participant is an `Employee` or a `Manager`. The business record tracks which employee made which stock adjustments, for audit purposes. The `UserAccount` tracks the credential. The `LoginManager` checks whether a warehouse worker logging in is who they say they are. After authentication, the application may need to enforce that a regular employee cannot delete a product record — that is authorization, which this module distinguishes from authentication but does not fully implement.

In a healthcare scheduling application, the domain participant is a `Patient` or a `Provider`. A `Patient` object exists because the clinic needs to track appointments, medical history flags, and contact information. A `UserAccount` exists because the patient logs in to view and book appointments. The credential must be kept separate from the medical record: the fields on `Patient` that a doctor might read are not the same fields that the authentication system needs to touch.

In both domains, the object separation is identical to the library: domain participant, credential, login manager. The threat-model question is the same. The sensitivity of the answer differs. Healthcare data carries legal obligations under HIPAA that a library book tracker does not. Knowing your domain tells you how much the threat-model answer matters.

| domain | domain participant object | UserAccount fields | what the login manager returns | threat-model severity |
| --- | --- | --- | --- | --- |
| library (Patron | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| inventory (Employee | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |
| scheduling (Patient) | the purpose is to show that the design shape is identical while the stakes differ | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. | A concrete checkpoint for applying the chapter concept. |

---

## Authentication vs. Authorization

This module implements authentication: verifying that a user is who they claim to be. It does not implement authorization: determining what an authenticated user is allowed to do.

The distinction matters because mixing them is a common mistake. A `LoginManager` that returns the authenticated user's role — "librarian" or "patron" — is beginning to do authorization. That is a reasonable next step, but it is a different responsibility from authentication, and putting both in `LoginManager` starts to blur the separation of concerns.

The bridge question for this module is: users can log in. What different actions can different user types perform? That question is authorization. It appears in the next module. For now, the `LoginManager` answers only: are these credentials valid? The calling code handles what comes next.

---

## Tracing the Login Flow

Let me trace a complete login attempt from user input to result, because making the invisible machinery concrete is how you verify it.

The user types a username and password into the login panel and clicks Submit. The submit handler fires. It reads the username string and the password string from the input fields. It calls `loginManager.login(username, password)`.

Inside `login`, `accounts.get(username)` looks up the `UserAccount` in the `HashMap`. If the username is not in the map, `get` returns `null` and `login` returns `false`. If the username is found, `account.checkPassword(password)` is called.

Inside `checkPassword`, the supplied password is hashed using the same `hash` method that was used when the account was created. The resulting hash is compared to the stored `passwordHash` field using `.equals()`. If they match, the method returns `true`. If not, `false`.

Back in the submit handler, if `login` returned `true`, the application navigates to the authenticated screen and passes the username or patronId forward. If `login` returned `false`, the handler displays an error message and stays on the login screen.

Notice what is not in this trace. The `Patron` object is never retrieved during authentication. The login flow does not know what books Alice has borrowed. It knows only whether the credential is valid. If the authentication succeeds and the application needs to display Alice's account, a separate lookup retrieves the `Patron` using the patronId stored in the `UserAccount`. Authentication and domain data access are separate operations.

That separation is the design. It is also the verification target. After implementing the login flow, trace it manually and confirm: does authentication touch the `Patron` object? It should not. Does the hash comparison use the same algorithm in both directions? It must. Does a failed login give the caller any information about why it failed — username not found vs. wrong password? It probably should not, because that distinction helps an attacker enumerate valid usernames.

![Login flow sequence (5 swimlanes) rendered as a systems diagram (sequence) for the chapter concept.](images/06-basics-of-gui-programming-in-java-fig-03.png)
*Figure 6.3 — Login flow sequence (5 swimlanes)*


![Attacker reads: plaintext vs. hashed rendered as a comparison panels for the chapter concept.](images/06-basics-of-gui-programming-in-java-fig-04.png)
*Figure 6.4 — Attacker reads: plaintext vs. hashed*


---

## What This Module Does Not Settle

We have separated the person from the account, implemented password hashing, and established the threat-model habit. We have not implemented role-based access control. We have not addressed session management: after login, how does the application know the user is still authenticated as they navigate between screens? We have not addressed the question of what happens when a user forgets their password.

These are real problems. In a production system they require serious engineering. In this course project they will be addressed incrementally. The foundation this module establishes — separate objects, hashed credentials, threat-model reasoning before code — is the ground that makes those later decisions tractable.

The bridge question: users can log in. What different actions can different user types perform?

---

## Lab: Account Directory and Login Manager

Implement `Person` (or your domain equivalent) and `UserAccount` as separate classes. Implement `LoginManager` with a `HashMap`-based account directory and a `login` method that returns a boolean. Hash passwords using `MessageDigest` SHA-256 on storage and verification.

Before writing any code, submit the threat-model table: threat scenario, plaintext exposure, hashed exposure. One row per threat.

The implementation must demonstrate that the domain participant object and the credential object are separate, that the login check hashes the supplied password before comparing, and that the `LoginManager` does not return the domain participant object.

If you used AI to scaffold the `HashMap` directory or explain `MessageDigest`, include the AI Use Disclosure: what you asked, what the AI produced, what you verified, and what the AI did not and could not decide — specifically, what your threat-model reasoning determined before AI was involved.

---

## Exercises

**Warm-up**

1. Draw the object structure for your domain: the domain participant class (name it), the `UserAccount` class, and the `LoginManager`. For each class, list its fields. For `UserAccount`, identify which field links to the domain participant and explain in one sentence why that link is a string ID rather than an object reference. *(Tests: Person/UserAccount separation; reference vs. key distinction.)*

2. Trace the hash computation for the password `"library2024"` through the `hash()` method step by step: what does `getBytes` produce, what does `digest()` do to it, and what format does the final return value take? You do not need to compute the actual SHA-256 output — describe what happens at each step and what would change if the charset were omitted. *(Tests: hash() method mechanics; charset significance.)*

**Application**

3. A classmate's `UserAccount` class has this constructor: `public UserAccount(String username, String password)` — and the `password` parameter is stored directly as a field named `passwordHash`. The `login` check compares the stored field to the supplied password using `.equals()`. The application runs and the demo passes. Write the threat-model row for "file read" for this system, and identify which line of code would need to change to fix the security property. *(Tests: threat model applied to broken implementation; plaintext vs. hashed storage.)*

4. Your `LoginManager.login()` method currently returns `false` for both "username not found" and "wrong password." A classmate suggests returning an enum — `LOGIN_SUCCESS`, `USER_NOT_FOUND`, `WRONG_PASSWORD` — so the UI can display a more specific error message. Apply the threat-model habit: what does this change expose to an attacker, and what does the UI gain? Make a recommendation and defend it. *(Tests: threat-model reasoning applied to an API design decision; information leakage.)*

5. AI generates a `LoginManager` that stores `UserAccount` objects with a `Patron` field instead of a `patronId` string. The code compiles. Identify which separation-of-concerns rule this violates, describe what downstream effect it would produce when a new feature requires iterating over all patrons, and write the one-line fix. *(Tests: object reference vs. key; separation of concerns; AI boundary verification.)*

**Synthesis**

6. Translate the library threat-model table to your own domain (inventory or scheduling). For at least two of the three threat rows, explain how the sensitivity of the answer differs from the library case — specifically, what real-world consequence a credential exposure would have that the library tracker would not. If your domain is healthcare scheduling, reference the HIPAA implication by name. *(Tests: domain transfer; threat-model severity reasoning; stakes differ while design shape is identical.)*

7. Your application has been running for three weeks when you discover that one `UserAccount` was created with a plaintext password because a developer called `this.passwordHash = password` instead of `this.passwordHash = hash(password)`. Write the remediation steps: what must change in the stored data, what must change in the code, and what must the user do. Explain why you cannot simply re-hash the stored plaintext to fix the problem without user involvement. *(Tests: hash one-way property; recovery from implementation error; verification of correct hashing at storage time.)*

**Challenge**

8. The `LoginManager` in this module holds accounts in a `HashMap` that is populated at application startup and lives in memory for the session. Describe two structural limitations of this design that would matter in a real deployed application — one related to persistence, one related to concurrent users. For each limitation, name the Java mechanism or design pattern that addresses it, and explain why introducing either would require changes to the `UserAccount` or `LoginManager` interface. *(Open-ended; anticipates persistence and session management topics in later modules.)*

---

- **Person** — the domain participant object: the human the business process needs to track. In the library, `Patron`. In inventory, `Employee`. In scheduling, `Patient`. Does not contain credentials.
- **UserAccount** — the credential object: username, password hash, and a link (by ID, not by reference) to the domain participant. Knows how to verify a supplied password against the stored hash.
- **authentication** — the process of verifying that a user is who they claim to be. Returns a yes or no. Does not determine what the authenticated user is allowed to do.
- **authorization** — the process of determining what an authenticated user is permitted to do. Distinct from authentication. Not implemented in this module; introduced in the next.
- **hash** — the fixed-length output of a one-way function applied to a password. The same password always produces the same hash. The hash cannot be efficiently reversed to recover the password.
- **threat model** — a structured answer to "if an attacker gains access to this component, what can they do?" In this module: what can an attacker do if they read the credential file?
- **HashMap** — a Java collection that stores key-value pairs with O(1) average-case lookup. In `LoginManager`, the key is the username string and the value is the `UserAccount` object.

---

## Assessments

| Assessment | Type | Credit status | Group or Individual | Alignment note |
| --- | --- | --- | --- | --- |
| Optional Exercise: Setting up JavaFX | Practice | For-Credit | Individual | Configure the environment and confirm JavaFX execution. |
| Optional Exercise: Creating a JavaFX Project | Practice | For-Credit | Individual | Create a runnable JavaFX project. |
| Optional Exercise: Continuing with Creating a JavaFX Project | Practice | For-Credit | Individual | Extend the first JavaFX project with additional controls. |
| Optional Exercise: Creating a JavaFX Application using Image and ImageView | Practice | For-Credit | Individual | Use image display elements in a GUI. |
| Optional Exercise: Creating an App | Practice | For-Credit | Individual | Build a small JavaFX interface. |
| Quiz - Basics of GUI programming in Java | Formative | For-Credit | Individual | Check GUI vocabulary, JavaFX setup, and layout understanding. |
| Optional Lab - Basic GUI Program | Practice | For-Credit | Individual | Create a basic GUI application. |

*Please label your assignments as Group or Individual.*


## Further Reading

- *Java Language Specification* and the Java SE API documentation: authoritative sources on `MessageDigest`, `HashMap`, and `Map` operations.
- OWASP Password Storage Cheat Sheet: authoritative guidance on password hashing in production systems, including bcrypt, scrypt, and Argon2. Use this when the course project is behind you and you are building something real.
- Robins, Rountree, and Rountree, "Learning and Teaching Programming": research on the design-confusion patterns this module addresses.
- Peng et al. and Vaithilingam et al.: empirical work on verification risk in AI-generated security code; the threat-model phase gate in this module is grounded in their findings.

---

## References

<!-- Fact-check pass references. -->
1. OpenJFX. Class Node. JavaFX 21 API Specification, 2023. https://openjfx.io/javadoc/21/javafx.graphics/javafx/scene/Node.html
