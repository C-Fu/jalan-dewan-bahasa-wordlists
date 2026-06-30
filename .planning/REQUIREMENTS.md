# Requirements: Jalan Dewan Bahasa Wordlists (v1.1)

**Defined:** 2026-07-01
**Core Value:** Malay speakers can generate secure, uniquely-decodable passphrases using common Malay words — without relying on English wordlists.

## v1.1 Requirements

### Web Passphrase Generator (WEB)

- [ ] **WEB-01**: `html/index.html` loads `db/jalan-dewan-bahasa-kecil-qwerty.db` via sql.js and generates random passphrases from the QWERTY wordlist
- [ ] **WEB-02**: Page follows DESIGN.md visual style (dark mode, #6366F1 primary accent, glassy card surfaces, system font, grid layout)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Multiple wordlist selection | v1.1 focuses on QWERTY only; extendable in future releases |
| Server-side rendering | Pure static HTML, no backend needed |
| Framework or build tools | Single HTML file with CDN-loaded sql.js |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| WEB-01 | Phase 5 | Pending |
| WEB-02 | Phase 5 | Pending |

**Coverage:**
- v1.1 requirements: 2 total (0 complete, 2 pending)
- Mapped to phases: 2
- Unmapped: 0 ✓

---
*Requirements defined: 2026-07-01*
