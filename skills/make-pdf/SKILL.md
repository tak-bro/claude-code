---
name: make-pdf
description: |
  Generate publication-quality PDFs from markdown. Clean typography with
  1in margins, Helvetica, curly quotes, optional cover page, TOC, and
  watermarks.
---

# make-pdf: Publication-Quality PDFs from Markdown

Turn `.md` files into PDFs that look like professional essays: 1in margins, left-aligned body, Helvetica throughout, curly quotes and em dashes, optional cover page and clickable TOC, diagonal DRAFT watermark when you need it.

## Setup

You need a PDF generation tool available. Check for common tools:

```bash
command -v pandoc >/dev/null 2>&1 && echo "PANDOC_AVAILABLE"
command -v wkhtmltopdf >/dev/null 2>&1 && echo "WKHTMLTOPDF_AVAILABLE"
command -v weasyprint >/dev/null 2>&1 && echo "WEASYPRINT_AVAILABLE"
command -v prince >/dev/null 2>&1 && echo "PRINCE_AVAILABLE"
```

Use whichever tool is available. If none, suggest installing one (pandoc is the most common).

## Core Patterns

### 80% case -- memo/letter

One command, no flags. Gets a clean PDF with page numbers.

```bash
pandoc letter.md -o letter.pdf --pdf-engine=xelatex -V geometry:margin=1in
```

### Publication mode -- cover + TOC + chapter breaks

```bash
pandoc essay.md -o essay.pdf --pdf-engine=xelatex \
  -V geometry:margin=1in --toc \
  -V title="On Horizons" -V author="Author Name"
```

### Fast iteration via preview

Convert to HTML first for quick visual checks:

```bash
pandoc essay.md -o preview.html --standalone
open preview.html
```

## Common flags (pandoc)

```
Page layout:
  -V geometry:margin=1in       Margins
  -V papersize:letter          Page size (letter, a4)

Structure:
  --toc                        Table of contents
  --toc-depth=2                TOC depth

Metadata:
  -V title="..."               Document title
  -V author="..."              Author
  -V date="..."                Date

Output:
  --pdf-engine=xelatex         PDF engine (xelatex for Unicode support)
  --pdf-engine=weasyprint      Alternative engine
```

## When to run

Watch for markdown-to-PDF intent:
- "Can you make this markdown a PDF"
- "Export it as a PDF"
- "Turn this letter into a PDF"
- "I need a PDF of the essay"

If the user has a `.md` file open and says "make it look nice", propose a publication-quality PDF and ask before running.

## Debugging

- Output looks empty -> check the markdown file exists and has content
- Unicode characters missing -> use xelatex engine instead of pdflatex
- External images missing -> ensure image paths are correct (relative to the markdown file)
- Generated PDF too tall/wide -> adjust margins or page size

## Output contract

The generated PDF path should be printed to the user. Open it with `open <path>` on macOS.
