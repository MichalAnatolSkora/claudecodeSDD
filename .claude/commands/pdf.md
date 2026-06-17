---
description: Convert markdown file(s) to PDF via pandoc + Prince
argument-hint: [input.md or glob] [output.pdf]
---

# Generate PDF from Markdown

Convert markdown to a PDF using pandoc with Prince as the PDF engine.

Arguments: `$ARGUMENTS`

## Steps

1. **Parse arguments:**
   - **Input** — first arg. If empty, default to `guides/*.md`.
   - **Output** — second arg. If empty, derive:
     - Single file (e.g. `README.md`) → `README.pdf`
     - Glob or multiple files → `output.pdf`

2. **Pre-check:**
   - Verify `pandoc` is on PATH (`which pandoc`). If missing, instruct: `brew install pandoc`.
   - Verify `prince` is on PATH (`which prince`). If missing, instruct: `~/.local/bin/prince` install path or refer to https://www.princexml.com/download/.
   - Stop if either is missing — do NOT attempt other PDF engines silently.

3. **Determine title:**
   - Single file: extract the first `# heading` from the file.
   - Glob/multiple: use the repo directory name (or "Document" as fallback).

4. **Run pandoc:**
   ```bash
   pandoc <input> -o <output> \
     --pdf-engine=prince \
     -H pdf-style-wide.html \
     --metadata title="<title>"
   ```
   - **No `--toc`**: each guide already has its own Table of Contents in markdown; an auto-TOC duplicates it.
   - `pdf-style-wide.html` is the unified house style (full-width text, small font, page-number footer). If it does not exist in the current working directory, omit the `-H` flag (use Prince defaults).
   - If the user wants to override the style, they can pass `-H pdf-style-compact.html` (narrower) or `-H <other.html>` — but only if explicitly asked; do not auto-create style files.

5. **Report:**
   - Output path (absolute)
   - File size
   - Any warnings from pandoc/prince (note: `unsupported properties: overflow-x` is harmless — Prince ignores web-only CSS)

## Constraints

- Do NOT modify the source markdown.
- Do NOT install dependencies without confirmation.
- Do NOT change the existing `pdf-style-*.html` files without confirmation.
- Do NOT auto-open the PDF unless the user asks.
