# PDF → Markdown / DOCX

A client-side tool for converting PDFs to Markdown or DOCX. No server, no uploads — all processing runs in your browser.

## Table of contents

- [How it works](#how-it-works)
- [Usage](#usage)
- [Supported PDF types](#supported-pdf-types)
- [Language support](#language-support)
- [Privacy](#privacy)
- [Libraries used](#libraries-used)

## How it works

The converter uses two strategies depending on the PDF type:

- **Native PDF** (selectable text): text is extracted directly via PDF.js — instant and accurate, preserving Unicode characters, diacritics, and bullet points.
- **Scanned PDF** (image-only pages): Tesseract.js OCR is used as a fallback. Only triggered for pages where no selectable text is found.

---

## Usage

### 1. Open the page

Navigate to the GitHub Pages URL. No installation or sign-in required.

### 2. Load a PDF

Either **drag and drop** a `.pdf` file onto the drop zone, or click it to open a file browser. The Convert button becomes active once a file is selected.

### 3. Convert

Click **Convert**. A progress bar tracks extraction page by page. Native pages process in under a second each; scanned pages take longer while OCR runs.

### 4. Review the output

The result appears in the text area as Markdown. Each page is wrapped in a `## Page N` heading and separated by `---`. You can edit the text directly in the area before exporting.

### 5. Export

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

> Accuracy depends on scan quality — higher resolution scans produce better results.

---

## Language support

Language selection only affects **scanned pages**. Native PDFs are extracted directly by PDF.js and are unaffected by this setting.

Tesseract.js supports 100+ languages. The most common European ones:

| Language  | Code  |
| --------- | ----- |
| English   | `eng` |
| Romanian  | `ron` |
| German    | `deu` |
| Hungarian | `hun` |
| French    | `fra` |
| Spanish   | `spa` |
| Italian   | `ita` |
| Polish    | `pol` |
| Czech     | `ces` |
| Slovak    | `slk` |
| Croatian  | `hrv` |
| Bulgarian | `bul` |
| Russian   | `rus` |
| Ukrainian | `ukr` |
| Dutch     | `nld` |
| Greek     | `ell` |
| Turkish   | `tur` |

For a full list of supported language codes see the [Tesseract.js language list](https://github.com/naptha/tesseract.js/blob/master/docs/tesseract_lang_list.md).

### Multiple languages

Tesseract accepts multiple languages simultaneously, e.g. `eng+ron+deu`. Each additional language increases the model download size and extends OCR initialisation time, so select only what the document actually contains.

---

## Privacy

Nothing leaves your machine. The file is read locally by the browser; no data is sent to any server. Closing the tab discards everything.

---

## Libraries used

- [PDF.js 5](https://mozilla.github.io/pdf.js/) — PDF parsing and rendering
- [Tesseract.js 6](https://tesseract.projectnaptha.com/) — OCR engine (WASM)
- [docx.js 9](https://docx.js.org/) — DOCX generation
