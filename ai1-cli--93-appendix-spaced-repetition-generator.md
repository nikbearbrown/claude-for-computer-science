# Appendix D — The Spaced-Repetition Card Generator

*The prompt behind Chapter 16's recall layer — and the two-line command that compiles it.*

---

This appendix carries two things: the prompt you paste into Claude (or ChatGPT) to generate atomic Q:/A: recall cards from a chapter draft, and a usage note for `build-anki.py`, the script that compiles those cards into an Anki deck.

The prompt is reusable across chapters. Paste it once per chapter, attach the chapter text, and you get a starter card set. Expect to edit: the generator is good at coverage and atomicity but will occasionally produce compound cards or cards that are too trivial. The editorial pass takes less time than authoring from scratch, and the prompt's instructions are explicit enough that you can point Claude at the specific failure ("card 7 is compound — split it").

---

## Spaced-Repetition Card Generator

```
You are generating spaced-repetition recall cards for a textbook chapter.
Your output is used directly in Anki. Follow every constraint exactly.

INPUT
-----
I will paste a chapter draft (or section) below. Read it fully before writing any cards.

CARD FORMAT (exact)
-------------------
Each card is a Q: line followed by an A: line (or lines). Blank line between cards.
Multi-line answers are allowed — indent continuation lines to the same column as A:.

Q: [question]
A: [answer — one fact only]

Q: [question]
A: [answer line one
   answer line two if needed]

ATOMIC CARD RULE (enforce strictly)
------------------------------------
Each card must test exactly one fact, one definition, one relationship, or one mechanism.
If a question asks two things, split it into two cards.
If an answer contains "and" joining two independent facts, split the card.

COVERAGE REQUIREMENTS
---------------------
- Every named concept introduced in the chapter gets at least one card.
- Every mechanism explained (how something works, why something happens) gets a card.
- Every named trade-off gets a card.
- Key numbers, dates, or named sources get cards only when they are genuinely testable and important — do not card trivial facts.
- Omit connective prose, transitions, and framing sentences that are not themselves factual claims.

DIFFICULTY CALIBRATION
----------------------
- Do not write cards where the question contains the answer. ("Q: What is the ____ effect, named after Ebbinghaus?" is bad.)
- Do not write cards with one-word answers unless the term genuinely stands alone. Prefer answers that are one complete sentence.
- If a concept has multiple parts, write one card per part, not one card that lists all parts.

OUTPUT FORMAT
-------------
Write all cards under the heading:

## Recall

Then list the Q:/A: pairs in the order they appear in the chapter, earliest first.

After the cards, write a one-line note:

> Cards: [N] — review for atomicity and coverage before committing to recall/.

Do not add any other commentary. Output only the ## Recall section and the note.

CHAPTER TEXT
------------
[paste chapter text here]
```

---

## Running build-anki.py

**What it reads:** Q:/A: pairs from `recall/*.md` files and from `## Recall` sections inside any `chapters/*.md` file. Both sources are combined into a single deck.

**What it produces:** `output/[book-slug].apkg` — a ZIP archive containing a SQLite database (`collection.anki2`, Anki schema 11) and an empty media manifest. The student double-clicks the `.apkg` to import the deck into Anki.

**Basic usage:**

```
python3 build-anki.py
```

**With explicit paths:**

```
python3 build-anki.py \
  --cards-dir recall \
  --chapters-dir chapters \
  --out output/my-book.apkg
```

The script reports the deck name (from `metadata.yaml`), card count, and schema version on success. If no cards are found, it exits with an explicit error rather than writing an empty deck.
