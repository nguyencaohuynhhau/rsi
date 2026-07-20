# Deep Reader Skill

**Language / Ngôn ngữ / 语言:** [English](./README.md) | [Tiếng Việt](./README.vi.md) | [中文](./README.zh.md)

This skill helps an agent deep-read long books and papers — the way a careful
human researcher does, not by stuffing the whole text into one prompt.

## What Deep Reading Means Here

An LLM's attention dilutes over long contexts and loses track of material
buried in the middle of a big prompt ("lost in the middle"). Loading a
500-page book into one context and asking questions about it produces answers
that quietly forget chapter 3 by the time you're discussing chapter 12.

This skill fixes that with the same method careful human researchers use:
never hold the whole book in working memory at once. It combines three
established reading methods into one pipeline:

- **Mortimer Adler's *How to Read a Book*** — inspectional, analytical, and
  syntopical reading, applied in that order.
- **SQ3R's Recite step** — after drafting a chapter's notes, re-skim the
  chapter and check every claim and quote against the text before moving on.
- **S. Keshav's three-pass method** for papers, theses, and surveys.

The book is read in passes — first for structure, then chapter by chapter for
content — and everything learned is externalized into a **page-anchored notes
workspace** as you go. The notes become the durable memory; the raw text only
needs to come back into view when actively reading a specific chapter or
verifying a specific claim. Page numbers are the coordinate system that makes
every claim traceable back to the source, in this session or a later one.

## Two Modes

- **overview** — Adler's inspectional pass only: a book map plus a
  goal-directed summary of the chapters that matter most to your reading
  purpose. Fast, no per-chapter notes.
- **study** — the full pipeline: reading purpose → inspectional map →
  chapter-by-chapter analytical notes with Recite verification →
  hierarchical synthesis answering Adler's four questions → mechanical quote
  verification.

You pick a mode (or the agent proposes one from the book's size and your
stated purpose, and you confirm or override).

## When To Use This Skill

Use it whenever you ask an agent to read, study, summarize, analyze, review,
or answer questions about a long book, textbook, PDF, EPUB, thesis,
dissertation, survey paper, or any document of roughly 50+ pages. It also
applies when a notes workspace from a previous session already sits next to a
source file and you ask a follow-up question about that book — the agent
searches the existing notes instead of re-reading anything.

Don't expect it to kick in for short documents (under ~30 pages): those fit
comfortably in context, and the pipeline would be pure overhead.

**Trigger phrases:** "read this book for me", "study this PDF", "summarize
this textbook", "analyze this paper", "deep-read this thesis", "đọc sách",
"tóm tắt sách", "phân tích luận án", "nghiên cứu paper này"

## Requirements

- **bash** — required, all scripts are POSIX-ish shell (coreutils, awk, grep,
  sed).
- **pdftotext** (from **poppler**) — required only for PDF sources.
- **pandoc** — required only for EPUB/DOCX sources. TXT/MD sources need
  neither.

Install on macOS:

```bash
brew install poppler pandoc
```

Install on Debian/Ubuntu:

```bash
apt-get install poppler-utils pandoc
```

If a dependency is missing, the prepare script exits with a clear error
instead of failing silently, and the skill falls back to reading the
PDF/EPUB directly with the agent's Read tool in page batches — you lose the
mechanical quote-verification pass in that fallback, so the agent compensates
with more frequent manual re-reads of anything it quotes.

## Usage

In an agent that supports skills, ask for tasks like:

```text
Deep-read "Domain-Driven Design" and tell me how it applies to our
current service boundaries — I want the full study, not just a summary.
```

```text
I just need the shape of this 400-page handbook before a meeting tomorrow —
give me an overview: what are the main parts and which chapters actually
matter for someone building a BI pipeline?
```

```text
(later, same book) What does the book say about aggregate roots versus
bounded contexts? Check your existing notes first.
```

For the third example, the agent searches the workspace's `terms.md` and
`notes/` files it already built rather than re-reading the book — that's the
entire point of externalizing notes as it goes.

## The Notes Workspace

Everything the pipeline produces lives in a workspace directory, by default a
sibling of the source file, so it survives across sessions:

```text
<slug>-notes/
├── source.txt          # full text with [[page N]] markers — the coordinate system
├── chapters.tsv         # confirmed structure: chNN<TAB>from<TAB>to<TAB>title
├── chapters/            # per-chapter files cut by split-chapters.sh
│   └── ch01-<slug>.txt
├── map.md               # inspectional pass output
├── terms.md             # key-terms ledger, cross-chapter
├── notes/
│   └── ch01-<slug>.md   # one analytical note per chapter
└── synthesis.md         # final synthesis + verification log
```

Page numbers anchor every note back into `source.txt`, which is what lets a
later session answer a follow-up question by searching notes instead of
re-reading the book.

## Scripts

Seven scripts drive the mechanical parts of the pipeline; the agent supplies
the judgment (structure confirmation, note-writing, synthesis) around them.

| Script | Purpose |
| --- | --- |
| `prepare-text.sh` | Convert the source to paged `source.txt`, create the workspace (idempotent) |
| `extract-structure.sh` | Heuristic outline seed — a starting guess, confirmed against the real table of contents |
| `read-pages.sh` | Print a page or page range, with `[[page N]]` markers intact |
| `search-book.sh` | Grep the whole book, page-annotated results |
| `split-chapters.sh` | Cut confirmed chapters into standalone files |
| `build-diagram.sh` | Mechanical Mermaid mindmap of the confirmed structure |
| `verify-quotes.sh` | Catch fabricated quotes and wrong page citations |

An eighth script, `self-test.sh`, is not part of the reading pipeline itself —
it's an install doctor. Run `bash ./scripts/self-test.sh` any time the
scripts misbehave; it generates its own synthetic fixture, exercises every
script, and reports which optional dependencies (`pdftotext`, `pandoc`) are
present, so you can tell an environment problem from a usage problem.

## Anti-Hallucination By Construction

Every quote a study-mode note carries gets checked mechanically:
`verify-quotes.sh` normalizes and matches each quoted string against the
cited page (± 1 page, to allow for sentences spanning a page break) and flags
`FAIL`/`NEAR` results for anything fabricated or mis-cited. In field testing,
the pipeline was used to deep-read three real 370–600-page books — *Domain-Driven
Design*, *Inspired*, and an ERP/BI handbook — and on the two study-mode runs
the verifier passed 458/458 and 120/120 citations respectively.

## Installation

### 1. Using CLI (Recommended)

```bash
npx skills add tronghieu/agent-skills --skill deep-reader
```

### 2. Manual Installation (For Non-Technical Users)

1. **Download:** Go to the `skills/` folder in this repository and download `deep-reader.zip`.
2. **Extract & Copy:** Extract `deep-reader.zip` and copy the `deep-reader` folder into one of the following directories:

**For a Specific Project:**
Copy the `deep-reader` folder to `.agents/skills/` or `.claude/skills/` in your project's root directory.

**Globally (Available for all projects):**
* **Mac / Linux:** `~/.agents/skills/` or `~/.claude/skills/`
* **Windows:** `%USERPROFILE%\.agents\skills\` or `%USERPROFILE%\.claude\skills\` (usually `C:\Users\<YourUsername>`)
