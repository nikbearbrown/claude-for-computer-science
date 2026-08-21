# Appendix C — The Book Scaffold Script (`new_book.py`)

*What `new_book.py` does and how to run it.*

---

This is the script behind Chapter 5 (*Book Scaffold*). Unlike the prompt appendices, a script is run, not pasted — and code ages faster than prose, so what follows is a usage reference. The script itself lives in the book repository at `pantry/new_book.py` (in a working setup it sits wherever you keep your tools, e.g. `~/tools/new_book.py`).

**Runs in:** a terminal (Python 3) — not Cowork or Codex; it is a script you execute.

**Dependencies:** none required. *Optional:* a `TIKTOC.md` to scaffold from (`--tiktoc TIKTOC.md`).

**Produces:** a complete book directory — planning files, `chapters/` stubs, `pantry/`, `images/`, `d3/`, `SCRIPTS/`.

---


## What it does

`new_book.py` reads nothing and asks nothing beyond its arguments. It creates a complete, empty book directory — the structural commitment your `TIKTOC.md` describes, turned into folders and stub files. Its whole purpose is to make the spec's vagueness visible: if the scaffold has a blank where a capability statement should be, the spec was blank there too.

## How to run it

```
python new_book.py "My Book Title" "Author Name"
python new_book.py "My Book Title" "Author Name" --subtitle "The Evidence Problem" --volume 1 --chapters 14
python new_book.py "My Book Title" "Author Name" --dir ~/textbooks/books
python new_book.py "My Book Title" "Author Name" --publisher "My Press"
```

**Arguments and flags:**

- `title` (required) — the working book title.
- `author` (required) — author name.
- `--subtitle` — optional subtitle.
- `--volume` — volume number for a series.
- `--chapters N` — number of chapter stubs to create.
- `--dir` — parent directory for the new book folder (defaults to the current directory).
- `--publisher` — publisher name for the copyright page.

## What it produces

```
book.md                 ← book description + high-level outline (planning)
outline.md              ← starter table of contents (planning)
vision.md               ← Tic TOC Phase 1: vision and positioning
architecture.md         ← Tic TOC Phase 2: learning architecture
chapters-spec.md        ← Tic TOC Phase 3: chapter specifications
risks.md                ← Tic TOC Phase 4: scope, market, risks
chapters/
    00-frontmatter.md   ← copyright, dedication, preface
    01-introduction.md  ← introduction
    02-chapter-01.md … NN-chapter-XX.md
    99-back-matter.md   ← acknowledgments, about the author, notes, references
pantry/                 ← fragments, snippets, research notes
images/                 ← figures as PNG (the book uses these)
d3/                     ← interactive D3 HTML versions
SCRIPTS/
```

The `vision.md`, `architecture.md`, `chapters-spec.md`, and `risks.md` files map one-to-one onto the four Tic TOC phases — the scaffold is the spec made into a directory. If those files come out thin, the spec was thin, and the fix is in Tic TOC (Appendix A), not here.
