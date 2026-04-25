# PDF → Markdown / DOCX

A client-side tool for converting PDFs to Markdown or DOCX. No server, no uploads — all processing runs in your browser.

---

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

| Type                                    | Method                     | Speed  |
| --------------------------------------- | -------------------------- | ------ |
| Native / selectable text                | PDF.js text extraction     | Fast   |
| Scanned / image-only                    | Tesseract.js OCR (English) | Slower |
| Mixed (some native, some scanned pages) | Per-page auto-detection    | Varies |

> OCR is English-only. Accuracy depends on scan quality — higher resolution scans produce better results.

---

## Privacy

Nothing leaves your machine. The file is read locally by the browser; no data is sent to any server. Closing the tab discards everything.

---

## Libraries used

- [PDF.js 5](https://mozilla.github.io/pdf.js/) — PDF parsing and rendering
- [Tesseract.js 6](https://tesseract.projectnaptha.com/) — OCR engine (WASM)
- [docx.js 9](https://docx.js.org/) — DOCX generation
