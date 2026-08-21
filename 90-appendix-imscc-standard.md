# Appendix — The Canvas Export Script (`build-imscc-standard.py`)

*What `build-imscc-standard.py` does and how to run it.*

---

This is the script behind Chapter 17 (*Canvas Course Export: `.imscc`*). Like `new_book.py` in Appendix C, a script is run, not pasted — and code ages faster than prose, so what follows is a usage reference. The script itself lives in the book repository at the book root, next to `build.sh` (in a working setup it sits wherever you keep that book's build tools).

**Runs in:** a terminal (Python 3) — not Cowork or Codex; it is a script you execute.

**Dependencies:** none. Standard library only (`zipfile`, `xml`, `argparse`, `re`, `uuid`). The whole point of the standard path is that it has nothing to install and nothing to break.

**Produces:** one `.imscc` file — a portable IMS Common Cartridge 1.3 package that imports into Canvas (Settings → Import Course Content → Common Cartridge 1.x Package), and also into Moodle, Blackboard, and Brightspace without modification.

---

## What it does

`build-imscc-standard.py` reads the same source that produces the EPUB — `chapters/*.md` plus `metadata.yaml` — and compiles it into a course package. Each chapter becomes a module with a page. Front matter becomes a *Start Here* module; appendices group into an *Appendices* module; back matter becomes *Back Matter*. Where a chapter has an exercises section, the script emits a second item — an assignment/discussion shell carrying the prompt.

The package is a ZIP whose root holds `imsmanifest.xml` — the course index that tells the LMS what resources exist and how they are organized — alongside a `web_resources/` folder of HTML pages generated from the markdown. The script deliberately writes *portable* Common Cartridge: it does **not** write the two Canvas-flavored trigger files (`course_settings/syllabus.html` and `course_settings/course_settings.xml`). Those belong to the optional Canvas-optimized path discussed in Chapter 17; adding them buys Canvas-specific richness at the cost of portability and maintenance.

## How to run it

```
python3 build-imscc-standard.py
python3 build-imscc-standard.py --out output/ai+1.imscc
python3 build-imscc-standard.py --source-dir . --tiktoc TIKTOC.md
```

**Arguments and flags:**

- `--source-dir` — the book root that contains `chapters/` and `metadata.yaml` (defaults to the current directory).
- `--out` — output `.imscc` path (defaults to `output/<title-slug>.imscc`).
- `--tiktoc` — optional `TIKTOC.md`, used only to confirm module order.

## What it produces

Run against this book's own source, the script reported:

```
Built: output/ai+1.imscc
  course title : AI+1
  modules      : 15
  web pages    : 32
  manifest     : imsmanifest.xml (Common Cartridge 1.3.0)
```

The resulting package was validated structurally: the manifest is well-formed (confirmed by an XML parser and by `xmllint`), it declares schema version `1.3.0`, every resource `href` exists inside the ZIP, and no organization item points at a missing resource.

## What it does *not* do

It does not call the Canvas API and it does not import anything. The professor uploads the one file. Confirming the import — opening Canvas, running Settings → Import Course Content, and reviewing the result against the Blueprint — is the reader's step, and it needs a Canvas course (a sandbox is fine). Import is not publication: the post-import diff review is the human gate Chapter 17 describes.

*Maintained copy:* the book repository at the book root (`build-imscc-standard.py`).
