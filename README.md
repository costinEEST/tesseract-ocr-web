# PDF → Markdown / DOCX

A client-side tool for converting PDFs to Markdown or DOCX. No server, no uploads — all processing runs in your browser.

## Table of contents

- [How it works](#how-it-works)
- [Usage](#usage)
- [Supported PDF types](#supported-pdf-types)
- [Page layout detection](#page-layout-detection)
- [Language support](#language-support)
- [Privacy](#privacy)
- [Libraries used](#libraries-used)

---

## How it works

The converter uses two strategies depending on the PDF type:

- **Native PDF** (selectable text): text is extracted directly via PDF.js — instant and accurate, preserving Unicode characters, diacritics, and bullet points. Column order is reconstructed from the x/y coordinates of each text item.
- **Scanned PDF** (image-only pages): Tesseract.js OCR is used as a fallback. Only triggered for pages where no selectable text is found. Before OCR runs, the page canvas is optionally split into column strips so each column is read top-to-bottom independently, preventing line interleaving.

---

## Usage

### 1. Open the page

Navigate to the tool URL. No installation or sign-in required.

### 2. Load a PDF

Either **drag and drop** a `.pdf` file onto the drop zone, or click it to open a file browser. The Convert button becomes active once a valid file is selected.

### 3. Choose an OCR language

Select the primary language of the document from the **OCR language** dropdown. This only affects scanned pages — native PDFs are unaffected. Choosing the correct language model is the single most important factor for diacritic accuracy (e.g. ș/ț/ă for Romanian, ő/ű for Hungarian).

Multi-language combinations (e.g. `Romanian + English`) are available for bilingual documents. Each extra language increases model download size and OCR initialisation time, so select only what the document actually contains.

### 4. Choose a page layout

Select how the page is structured using the **Page layout** dropdown.

| Option                  | Behaviour                                                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Auto-detect _(default)_ | Analyses each scanned page for a vertical whitespace band in the centre third. Splits there if found; otherwise reads as a single column. |
| Single column           | Reads the full page width as one continuous flow.                                                                                         |
| Two columns             | Splits at 50% page width. OCRs the left strip first, then the right.                                                                      |
| Three columns           | Splits at 33% and 66%. Reads left → middle → right.                                                                                       |

For native PDFs, explicit column settings reorder text items by their PDF x-coordinate rather than splitting a canvas.

### 5. Choose a page separator

Select how pages are delimited in the output using the **Page separator** dropdown.

| Option                        | Output                          | Best for                                              |
| ----------------------------- | ------------------------------- | ----------------------------------------------------- |
| Horizontal rule               | `---` between pages             | General use, broadly supported                        |
| Heading + rule                | `## Page N` heading + `---`     | When page numbers need to be visible                  |
| Double blank line _(default)_ | Two empty lines                 | Prose — no visual noise in rendered output            |
| HTML comment                  | `<!-- Page N -->` between pages | Pipelines — invisible when rendered, machine-readable |

### 6. Convert

Click **Convert**. A progress bar tracks extraction page by page. For multi-column scanned pages the status shows the current column (`col 1/2`, `col 2/2`). Native pages process in under a second each; scanned pages take longer while OCR runs.

### 7. Review the output

The result appears in the text area as Markdown. You can edit it directly before exporting.

### 8. Export

| Button  | Output                                            |
| ------- | ------------------------------------------------- |
| 📋 Copy | Copies Markdown to clipboard                      |
| ⬇ .md   | Downloads a `.md` file                            |
| ⬇ .docx | Downloads a formatted Word document with headings |

---

## Supported PDF types

| Type                                    | Method                  | Speed  |
| --------------------------------------- | ----------------------- | ------ |
| Native / selectable text                | PDF.js text extraction  | Fast   |
| Scanned / image-only                    | Tesseract.js OCR        | Slower |
| Mixed (some native, some scanned pages) | Per-page auto-detection | Varies |

Accuracy for scanned documents depends on scan resolution — higher DPI produces better results. The tool renders each page at 2.5× scale before OCR to improve character recognition.

---

## Page layout detection

Multi-column documents (contracts, newspapers, academic papers) have their columns read in the wrong order when a naive full-page OCR is applied — the engine reads one line across both columns before moving down. This tool addresses this at two levels.

### Auto-detect (scanned PDFs)

For each scanned page the rendered canvas is analysed before OCR:

1. The middle 50% of the page height is sampled (headers and footers are excluded).
2. For each x position, the fraction of dark pixels (luminance < 200) is computed.
3. The result is smoothed over a window of ~3% of page width.
4. The minimum dark-pixel fraction is found within the centre 30% of page width.
5. If that minimum is below 1.5%, a clear column gap has been detected and the page is split there.

If no gap is found, the page is treated as single-column regardless of the layout setting.

### Explicit column settings

When auto-detection misses (e.g. a scan with stray marks in the gutter), use the **Two columns** or **Three columns** options to force a fixed split at equal intervals.

### Native PDFs

For PDFs with embedded text, PDF.js provides the x/y coordinates of every text item. When a column mode is active, items are bucketed by x-range and each bucket is sorted top-to-bottom (descending PDF y, since the coordinate origin is at the bottom-left) before being joined.

---

## Language support

Language selection only affects **scanned pages**. Native PDFs are extracted directly by PDF.js and are unaffected by this setting.

The correct language model is critical for diacritic accuracy. Without it, Tesseract maps unfamiliar glyphs to the nearest ASCII equivalent — the most common Romanian example being `ș` misread as `g` (producing `gi` instead of `și`). The tool also applies a language-specific post-processing pass on top of Tesseract's output to correct systematic misreads that persist even with the correct model.

### Available languages

#### Latin script — with diacritics

| Language  | Code  |
| --------- | ----- |
| Romanian  | `ron` |
| Hungarian | `hun` |
| Turkish   | `tur` |
| Spanish   | `spa` |
| Italian   | `ita` |
| German    | `deu` |
| French    | `fra` |
| Latin     | `lat` |

#### Latin script — ASCII

| Language | Code  |
| -------- | ----- |
| English  | `eng` |

#### Cyrillic

| Language  | Code  |
| --------- | ----- |
| Russian   | `rus` |
| Ukrainian | `ukr` |
| Bulgarian | `bul` |

#### Other scripts

| Language | Code  |
| -------- | ----- |
| Greek    | `ell` |
| Hebrew   | `heb` |

#### Combinations

| Selection           | Codes     |
| ------------------- | --------- |
| Romanian + English  | `ron+eng` |
| Russian + English   | `rus+eng` |
| Ukrainian + Russian | `ukr+rus` |
| Spanish + English   | `spa+eng` |

Tesseract.js supports 100+ languages in total. For a full list of supported codes see the [Tesseract.js language list](https://github.com/naptha/tesseract.js/blob/master/docs/tesseract_lang_list.md).

### Romanian diacritic repair

Beyond using the `ron` model, the tool applies an additional post-processing pass targeting the most common systematic errors in Romanian contract and legal text:

| Misread            | Corrected   | Rule               |
| ------------------ | ----------- | ------------------ |
| `gi`               | `și`        | `ș` misread as `g` |
| standalone `si`    | `și`        | conjunction `și`   |
| `In`               | `În`        | preposition `în`   |
| `pentru ca`        | `pentru că` | conjunction `că`   |
| `-ti` word endings | `-ți`       | inflection suffix  |

---

## Privacy

Nothing leaves your machine. The file is read locally by the browser; no data is sent to any server. Tesseract language models are downloaded from a public CDN at first use and cached by the browser. Closing the tab discards all processed content.

---

## Libraries used

- [PDF.js 5](https://mozilla.github.io/pdf.js/) — PDF parsing and rendering
- [Tesseract.js 6](https://tesseract.projectnaptha.com/) — OCR engine (WASM)
- [docx.js 9](https://docx.js.org/) — DOCX generation
