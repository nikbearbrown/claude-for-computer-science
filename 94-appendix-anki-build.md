# Appendix — The Anki Deck Script (`build-anki.py`)

*What `build-anki.py` does and how to run it.*

---

This is the script behind Chapter 16 (*Spaced Repetition: Anki and the Forgetting Curve*). It compiles the book's recall layer into an Anki deck the student double-clicks to import. Like the other scripts in these appendices, it is run, not pasted; the maintained copy lives in the book repository at the book root.

**Runs in:** a terminal (Python 3).

**Dependencies:** none. Standard library only (`sqlite3`, `zipfile`, `json`, `hashlib`). The deck is a real Anki package built by hand, not a third-party gem.

**Produces:** one `.apkg` file — an Anki deck (note type *Basic*, Front/Back) that imports into Anki on every desktop platform and into AnkiMobile/AnkiDroid.

---

## What it does

The recall layer is authored as plain `Q:`/`A:` pairs in markdown — either in a dedicated `recall/*.md` directory or in a `## Recall` (or `## Spaced Repetition`) section inside a chapter. `build-anki.py` reads those pairs and writes a `.apkg`: a ZIP containing `collection.anki2` (a SQLite database in Anki's schema 11, with the `col`, `notes`, and `cards` tables) and a `media` manifest (`{}` when the cards carry no images). One deck per book, named from `metadata.yaml`.

## How to run it

```
python3 build-anki.py
python3 build-anki.py --cards-dir recall --out output/ai+1.apkg
python3 build-anki.py --chapters-dir chapters   # also scan ## Recall sections
```

**Arguments and flags:**

- `--source-dir` — book root (defaults to the current directory).
- `--cards-dir` — directory of `Q:`/`A:` markdown (defaults to `<source>/recall`).
- `--chapters-dir` — also scan chapter files for `## Recall` sections.
- `--out` — output `.apkg` path (defaults to `output/<title-slug>.apkg`).

Card format:

```
Q: What does imsmanifest.xml do in a Common Cartridge package?
A: It is the course index — it tells the LMS what resources exist
   and how they are organized.

Q: Why does massed re-reading fail for long-term retention?
A: Without spacing, retrieval is never effortful, so the forgetting
   curve never resets.
```

A blank line separates cards.

## What it produces

Run against a sample recall set, the script reported:

```
Built: output/ai+1.apkg
  deck  : AI+1
  cards : 3
  note  : Basic (Front/Back), Anki schema 11
```

The package was validated structurally: it contains `collection.anki2` and `media`; the SQLite collection has exactly one `col` row at schema version 11; every card resolves to a note; and the note fields are joined by Anki's `0x1f` field separator.

## What it does *not* do

It does not open Anki and it does not sync. The student double-clicks the `.apkg` (or File → Import) to load the deck — that confirmation step is the reader's, exactly like the Canvas upload in Chapter 17. The script also does not write the cards for you: card quality (atomic, one fact each) is an authoring discipline, supported by the generator prompt in Appendix 93.

*Maintained copy:* the book repository at the book root (`build-anki.py`).
