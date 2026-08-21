# Appendix — The React Site Script (`build-react-site.py`)

*What `build-react-site.py` does and how to run it.*

---

This is the script behind Chapter 18 (*The React Site: `.mdx` + `.tsx`*). It scaffolds a Next.js site from the book's markdown. The author runs the script; a developer runs and deploys the result. The maintained copy lives in the book repository at the book root.

**Runs in:** a terminal (Python 3) to scaffold. The site itself runs in Node.js / Next.js — the developer's environment, not the author's.

**Dependencies (to scaffold):** none. Standard library only. The generated site's own dependencies (Next.js, React, the MDX loader) are listed in the `package.json` it writes and installed later with `npm install`.

**Produces:** a Next.js App-Router project — `.mdx` content files, `.tsx` page components, a layout, an embedded `AskAI` component placeholder, and the config files needed to build.

---

## What it does

The source stays `.md` — the single source of truth. `build-react-site.py` converts each chapter to `content/<slug>.mdx` (markdown with room for JSX, with bare braces neutralized so MDX does not choke on prose), and writes one route per chapter at `app/<slug>/page.tsx` that imports the MDX and renders it. It also writes `app/page.tsx` (a table of contents), `app/layout.tsx`, `components/AskAI.tsx` (the placeholder for the embedded Ask-AI loop of Chapter 20), `mdx-components.tsx`, `next.config.mjs`, `tsconfig.json`, and `package.json`. The author never hand-writes `.mdx` or `.tsx`; the script generates them.

## How to run it

```
python3 build-react-site.py
python3 build-react-site.py --out site
```

**Arguments and flags:**

- `--source-dir` — book root (defaults to the current directory).
- `--out` — output directory for the scaffold (defaults to `./site`).

## What it produces

Run against this book's source, the script reported:

```
Scaffolded: site
  book        : AI+1
  chapters    : 41
  files       : 89
  next steps  : cd site && npm install && npm run dev  (developer's step)
```

For 41 chapter and appendix files it generated 89 files — one `.mdx` and one `.tsx` route per chapter, plus the shared layout, the table of contents, the `AskAI` placeholder, and the config files. The generated `package.json` and `tsconfig.json` are valid JSON and the `.tsx` files carry valid default exports.

## What it does *not* do

It does not run `npm`, build, style, or deploy the site. Those are the developer's steps (`npm install`, `npm run build`, then a host such as Vercel or Netlify). This is the one deployment target the author cannot finish alone — unlike the `.imscc` and the `.apkg`, a working public site needs a developer and a hosting account. Chapter 18 is written as the hand-off: what the site gives your students, and what to hand your developer.

*Maintained copy:* the book repository at the book root (`build-react-site.py`).
