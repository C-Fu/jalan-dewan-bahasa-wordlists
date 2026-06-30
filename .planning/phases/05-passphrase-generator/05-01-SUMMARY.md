# Summary: Plan 05-01 — Passphrase Generator HTML Page

**Status:** Complete
**Duration:** ~5 min

## Deliverable

- `html/index.html` — 396 lines, single-file passphrase generator

## What Was Built

A dark-mode, single HTML page that uses sql.js (WebAssembly SQLite) to load `db/jalan-dewan-bahasa-kecil-qwerty.db` (1,296 words, 10.340 bits/word) and generates random Malay passphrases in-browser.

### Features
- Sql.js loads the wordlist database directly in the browser
- Configurable word count (3-12, default 6)
- Click any word chip to replace it with a new random word
- One-click copy to clipboard with success feedback
- Entropy estimate displayed for generated passphrase
- Dark-mode UI styled per DESIGN.md (indigo #6366F1 accent, glass card, gradient shell border, system font)

### File Structure
Created: `html/index.html`

### Requirements Satisfied
- **WEB-01**: sql.js-powered passphrase generator loading the QWERTY SQLite database ✓
- **WEB-02**: DESIGN.md visual style (dark mode, #6366F1 accent, glassy card, system font) ✓
