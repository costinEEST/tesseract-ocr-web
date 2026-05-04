# PDF → Markdown / DOCX

A browser-only PDF converter that turns native, scanned, and mixed PDFs into Markdown or DOCX.

No server. No uploads. The PDF is processed locally in your browser.

## Table of contents

- [Overview](#overview)
- [How it works](#how-it-works)
- [Usage](#usage)
- [Supported PDF types](#supported-pdf-types)
- [Native text detection](#native-text-detection)
- [OCR processing](#ocr-processing)
- [Page layout handling](#page-layout-handling)
- [Page separators](#page-separators)
- [Language support](#language-support)
- [Romanian post-processing](#romanian-post-processing)
- [DOCX export](#docx-export)
- [Privacy](#privacy)
- [Libraries used](#libraries-used)

---

## Overview

**PDF → Markdown / DOCX** converts PDF documents into editable Markdown and optionally exports the result as a Word document.

The converter supports:

- native PDFs with selectable text
- scanned PDFs with image-only pages
- mixed PDFs where some pages have text layers and others require OCR
- archival PDFs that contain generated footer/header text but no meaningful selectable page text
- single-column and multi-column layouts
- multilingual OCR
- Markdown and DOCX export

---

## How it works

The converter processes the PDF page by page.

For each page, it first tries to extract selectable text using **PDF.js**. It then checks whether that text is meaningful. This is important because some scanned archival PDFs contain a small selectable text layer made only of generated metadata, page numbers, or source URLs.

If the cleaned text layer contains enough real text, the page is treated as a native PDF page.

If not, the page is rendered to a canvas and processed with **Tesseract.js OCR**.

The final text is cleaned, post-processed, joined with the selected page separator, and displayed as Markdown.

---

## Usage

### 1. Open the page

Open `index.html` in a browser or visit the deployed tool URL.

No installation or sign-in is required.

### 2. Load a PDF

Drag and drop a `.pdf` file onto the drop zone, or click the drop zone to select a file.

The file is validated by checking that it starts with the `%PDF-` header. Once a valid PDF is selected, the **Convert** button becomes active.

### 3. Choose an OCR language

Use the **OCR language** dropdown to select the main language of the scanned document.

This setting affects only OCR pages. Native PDF text is extracted directly from the PDF text layer.

Choosing the correct OCR language improves recognition of diacritics, accents, ligatures, and non-Latin scripts.

### 4. Choose a page layout

Use the **Page layout** dropdown to control how pages are read.

| Option              | Behaviour                                                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Auto-detect columns | For OCR pages, tries to detect a reliable central gutter and split into two columns only when both sides contain text. |
| Single column       | OCRs the whole page as one continuous text block.                                                                      |
| Two columns         | Splits the rendered page at 50% width and reads left column first, then right column.                                  |
| Three columns       | Splits the rendered page at 33% and 66% width and reads left, middle, then right.                                      |

For native PDF pages, explicit two-column and three-column modes reorder PDF text items by their x/y coordinates. Auto-detect does not run pixel analysis on native PDF text layers.

### 5. Choose a page separator

Use the **Page separator** dropdown to control how pages are joined in the Markdown output.

| Option            | Markdown output                                         |
| ----------------- | ------------------------------------------------------- |
| Horizontal rule   | `---` between pages                                     |
| Heading + rule    | `## Page N` before each page, with a rule between pages |
| Double blank line | Two blank lines between pages                           |
| HTML comment      | `<!-- Page N -->` between pages                         |

### 6. Convert

Click **Convert**.

A progress bar shows the current page. For OCR pages with multiple strips, the status also shows the current column, such as `col 1/2`.

### 7. Review the output

The converted Markdown appears in the output text area.

### 8. Export

| Button | Action                                                       |
| ------ | ------------------------------------------------------------ |
| Copy   | Copies Markdown to the clipboard                             |
| .md    | Downloads the Markdown output                                |
| .docx  | Downloads a Word document generated from the Markdown output |

---

## Supported PDF types

| PDF type                                    | Processing method                  | Notes                                                                             |
| ------------------------------------------- | ---------------------------------- | --------------------------------------------------------------------------------- |
| Native / selectable text                    | PDF.js text extraction             | Fastest path. Preserves existing Unicode text.                                    |
| Scanned / image-only                        | Canvas render + Tesseract.js OCR   | Slower, depends on scan quality.                                                  |
| Mixed native/scanned                        | Per-page detection                 | Each page is handled independently.                                               |
| Archival scans with footer-only text layers | Boilerplate cleanup + OCR fallback | Prevents generated source URLs and page labels from being mistaken for real text. |

---

## Native text detection

The converter does not treat a page as native merely because `getTextContent()` returns some text.

Before deciding, it removes known full-line boilerplate emitted by archival PDF generators, currently including ONB-style patterns such as:

- `https://data.onb.ac.at/rep/...`
- `Seite X von Y`
- `https://data.onb.ac.at/rep/... Seite X von Y`
- `Erstelldauer HH:MM:SS.sss`

After boilerplate cleanup, the page is considered meaningful native text only when it contains enough letters, words, or non-empty lines.

This prevents scanned PDFs with generated page footers from bypassing OCR.

---

## OCR processing

When a page needs OCR, the converter:

1. renders the PDF page to a canvas
2. uses a higher OCR render scale of `3.25`
3. crops a small footer band from the bottom of the rendered canvas
4. optionally splits the canvas into column strips
5. sends each strip to Tesseract.js
6. joins the OCR results in reading order
7. removes known boilerplate
8. applies generic and language-specific post-processing

The footer crop is conservative and is intended for archive-generated footer bands such as source URLs and `Seite X von Y` page labels.

The OCR worker is configured with:

```js
preserve_interword_spaces: "1";
user_defined_dpi: "300";
```

This helps preserve spacing and gives Tesseract a stable DPI hint.

---

## Page layout handling

### Auto-detect columns

Auto-detection runs only on rendered OCR canvases.

The algorithm:

1. samples the middle 50% of page height
2. computes the fraction of dark pixels for each x position
3. smooths the signal over about 3% of page width
4. searches for the cleanest vertical gutter in the centre 30% of the page
5. splits only if both sides contain text and the gutter is much cleaner than the surrounding text areas

Auto-detect currently returns either:

- no split
- one central split for a two-column page

It does not auto-detect three columns. Use the explicit **Three columns** option for that.

### Explicit column modes

For OCR pages:

- **Two columns** splits the rendered canvas at 50%.
- **Three columns** splits the rendered canvas at 33% and 66%.

Each strip is OCRed separately, then joined left to right.

For native PDF pages:

- text items are bucketed into columns by PDF x-coordinate
- each column is sorted top to bottom using PDF y-coordinate
- columns are joined left to right

---

## Page separators

The selected page separator affects both Markdown and DOCX output.

### Horizontal rule

Markdown:

```md
---
```

DOCX:

A simple horizontal text divider is inserted.

### Heading + rule

Markdown:

```md
## Page N

Page text...
```

DOCX:

A level-2 heading is inserted for each page.

### Double blank line

Markdown:

Two blank lines are inserted between pages.

DOCX:

Two empty paragraphs are inserted.

### HTML comment

Markdown:

```md
<!-- Page N -->
```

DOCX:

A muted `[Page N]` marker is inserted.

---

## Language support

The OCR language selector uses Tesseract language codes.

### Latin script — with diacritics

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

### Latin script — ASCII

| Language | Code  |
| -------- | ----- |
| English  | `eng` |

### Cyrillic

| Language  | Code  |
| --------- | ----- |
| Russian   | `rus` |
| Ukrainian | `ukr` |
| Bulgarian | `bul` |

### Other scripts

| Language | Code  |
| -------- | ----- |
| Greek    | `ell` |
| Hebrew   | `heb` |

### Language combinations

| Selection           | Codes     |
| ------------------- | --------- |
| Romanian + English  | `ron+eng` |
| Russian + English   | `rus+eng` |
| Ukrainian + Russian | `ukr+rus` |
| Spanish + English   | `spa+eng` |

Multi-language OCR can improve results for bilingual documents, but it may increase model download size and OCR startup time.

---

## Romanian post-processing

For Romanian OCR, the converter applies conservative fixes for common OCR mistakes.

Current fixes include:

| OCR output                                       | Corrected output |
| ------------------------------------------------ | ---------------- |
| `gi`                                             | `și`             |
| `Gi`                                             | `Și`             |
| `gi-`                                            | `și-`            |
| `si`                                             | `și`             |
| `Si`                                             | `Și`             |
| selected legal/document suffixes ending in `ti`  | `ți`             |
| selected legal/document suffixes ending in `tii` | `ții`            |
| `pentru ca`                                      | `pentru că`      |
| `ca sa`                                          | `că să`          |
| `In`                                             | `În`             |
| selected lowercase `in` contexts                 | `în`             |

These fixes are intentionally limited. They do not attempt to fully reconstruct Romanian diacritics in arbitrary text.

---

## DOCX export

The DOCX export uses the generated Markdown as input.

Supported formatting includes:

- `# Heading` as Word heading level 1
- `## Heading` as Word heading level 2
- normal lines as standard paragraphs
- selected page separators as Word paragraphs or markers

The DOCX export is intended for readable document output, not full visual reproduction of the original PDF layout.

---

## Privacy

All PDF processing happens locally in the browser.

The selected PDF is not uploaded to a server by this tool.

Tesseract language models are downloaded from the configured public CDN when needed and may be cached by the browser.

Closing the tab discards the current converted output unless you copy or download it.

---

## Libraries used

- [PDF.js 5](https://mozilla.github.io/pdf.js/) — PDF parsing, text extraction, and rendering
- [Tesseract.js 6](https://tesseract.projectnaptha.com/) — browser-based OCR
- [docx.js 9](https://docx.js.org/) — DOCX generation
