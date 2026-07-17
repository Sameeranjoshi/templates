# Paper → Conference Slides — reusable prompt

Paste the block below into Claude Code with the paper attached (or give the PDF path).
Fill the four PARAMETERS. Everything else is the workflow that produced the GLOW deck.

---

You are creating a clean, **academic conference-style** slide deck (SC/HPC style) that
summarizes the attached paper: **<PAPER.pdf>**.

GOAL: a deck that faithfully reproduces the paper's **own figures and tables** and follows
the paper's structure. **Never redraw a figure. Never invent a number.**

PARAMETERS (fill these in):
- Output format: **PowerPoint .pptx** (default; alt: LaTeX Beamer PDF, or reveal.js HTML)
- Accent palette: **<e.g. deep blue + teal>** — one primary + one accent + grays (no logo colors unless asked)
- Author / affiliation / venue: **<name / org / SC'xx>**
- Depth: **comprehensive (~30–40 slides, tag backup)** or **talk-length (~20)**

STEP 1 — READ THE WHOLE PAPER first, page by page (use the PDF reader with page ranges).
Capture every section, figure, table, and key number. Write down any place where the paper
**contradicts itself** (text vs. table vs. figure) — you will resolve these deliberately later.

STEP 2 — CHECK TOOLING and report what's present:
`python-pptx`, `Pillow`, `matplotlib`, poppler (`pdftoppm`, `pdfimages`), and `soffice`
(LibreOffice). These decide the figure-extraction and verification paths.

STEP 3 — EXTRACT FIGURES FAITHFULLY into `figures/`:
- `pdfimages -png -p paper.pdf out/img` → pulls **embedded rasters at native resolution**
  (matplotlib plots often land here as *complete* figures, 2–3× sharper than a page render). View them; keep whole figures.
- For **vector** figures/tables (schematics, some plots, tables) not in `pdfimages`:
  `pdftoppm -png -r 300 paper.pdf out/page` to render pages, then **crop with Pillow** by bounding box.
  Loop: view the page → estimate the box → crop → **view the crop** → tighten until clean.
- **View every crop** before using it. Split composite figures (e.g. a top/bottom figure) into separate images if they support two different slides.

STEP 4 — BUILD THE DECK with `python-pptx` (16:9, blank layout). Write a small helper layer once:
- palette constants; a master that draws **title bar + accent underline + footer (short title + page #) + section tag**;
- a **bullet helper** using real bullets — set `a:buChar`/`a:buFont`/`a:buClr` and `marL`/`indent` on the paragraph's `pPr` for proper hanging indents (don't fake with "• ");
- an **image-fit helper**: scale to fit a box preserving aspect ratio (use Pillow for pixel dims);
- a **native-table helper**: turn OFF `firstRow`/`bandRow`, remove `tableStyleId`, set every cell fill manually, and make **column widths SUM to the table width** (mismatch makes the table overrun its neighbours);
- **section-divider** slides and **stat-callout** boxes for the TL;DR;
- **speaker notes on every slide** (short talking points).
- Fonts: **Arial** for body, **Consolas** for code/monospace. Use only universal fonts — they render on the *opener's* machine (PowerPoint/Keynote/Google Slides/LibreOffice), not at build time.

STEP 5 — CONTENT follows the paper's own layout. Typical skeleton:
Title → TL;DR (3 stat callouts + one-line thesis) → Outline → **per section**: where the
application/method is used & why, how it looks, key algorithms/kernels, hardware details, the
proposed approach & techniques, evaluation (per-figure), roofline/analysis → Lessons → Related
→ Conclusion → Backup "key numbers". **Put each paper figure on its own slide** with a caption
and 3–5 explanatory bullets. Keep slide bullets short; push detail into the speaker notes.

STEP 6 — TABLES: rebuild the **2–3 headline tables natively** (crisp, editable) and **verify
every digit** against the PDF; highlight the key row/column in the accent color. Embed **dense
tables as cropped images** to guarantee fidelity.

STEP 7 — VERIFY VISUALLY AND ITERATE (this is the most important step):
- If `soffice` exists: `soffice --headless --convert-to pdf deck.pptx` then `pdftoppm` → view
  each slide (pixel-accurate).
- Else: write a **matplotlib preview** that reads the `.pptx` back — real shape geometry, the
  real embedded images (`shape.image.blob`), table grids, and wrapped text — renders each slide
  to PNG, and **flags any text box whose estimated content height exceeds its box (overflow) or
  any shape off-slide**. (Gotchas: give text a high `zorder` so fills don't cover it; the preview
  font is wider than Arial, so estimate wraps with Arial-ish metrics and render ~0.94×.)
- Fix issues and re-render until **zero warnings and every slide looks clean**. Spot-check the
  title, dividers, every figure/table slide, and the busiest text slides.

ACCURACY RULES:
- Use the paper's own figures — never redraw.
- Verify every transcribed number.
- When the paper contradicts itself, use the value from **the figure/table you are showing** and
  note the discrepancy in the speaker notes — don't propagate the ambiguous one.
- Keep one baseline / naming convention consistent across the whole deck.

DELIVER: the deck file, `figures/`, the generator script, the preview script, and a short summary
of the structure plus any inconsistencies you resolved.
