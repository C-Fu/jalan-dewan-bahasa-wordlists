# Requirements: Jalan Dewan Bahasa Wordlists

**Defined:** 2026-06-30
**Core Value:** Malay speakers can generate secure, uniquely-decodable passphrases using common Malay words — without relying on English wordlists.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Documentation (DOC)

- [ ] **DOC-01**: README.md written in Malay covering project overview, list descriptions, and licensing
- [ ] **DOC-02**: FAQ.md written in Malay answering common questions about usage and creation

### Alpha Short List (ALFA)

- [ ] **ALFA-01**: `lists/jalan-dewan-bahasa-kecil-alpha.txt` — 1,296 Malay words optimized for alphabetical on-screen keyboards, uniquely decodable, min 3 characters
- [ ] **ALFA-02**: `lists/jalan-dewan-bahasa-kecil-alpha-dadu.txt` — Alpha list with 4-dice-roll prefixes

### QWERTY Short List (QWER)

- [ ] **QWER-01**: `lists/jalan-dewan-bahasa-kecil-qwerty.txt` — 1,296 Malay words optimized for QWERTY on-screen keyboards, uniquely decodable, min 3 characters
- [ ] **QWER-02**: `lists/jalan-dewan-bahasa-kecil-qwerty-dadu.txt` — QWERTY list with 4-dice-roll prefixes

### Medium List (SEDE)

- [ ] **SEDE-01**: `lists/jalan-dewan-bahasa-sederhana.txt` — 8,192 Malay words, 13 bits/word, uniquely decodable

### Diceware List (DADU)

- [ ] **DADU-01**: `lists/jalan-dewan-bahasa-dadu.txt` — 7,776 Malay words with 5-dice-roll prefixes, ~12.9 bits/word
- [ ] **DADU-02**: `lists/jalan-dewan-bahasa-dadu-bersih.txt` — Same words as DADU-01 without dice-roll prefixes

### Long List (BESA)

- [ ] **BESA-01**: `lists/jalan-dewan-bahasa-besar.txt` — 17,576 Malay words, ~14.1 bits/word, uniquely decodable

### Cross-Cutting (XCUT)

- [ ] **XCUT-01**: Translation deduplication — remove duplicate Malay words resulting from many-to-one English→Malay mappings before pruning
- [ ] **XCUT-02**: Normalization — lowercase, trim whitespace, UTF-8 NFC, remove non-alphabetic characters
- [ ] **XCUT-03**: Sardinas–Patterson unique decodability check on every list
- [ ] **XCUT-04**: Greedy Schlinkert pruning to restore unique decodability where needed
- [ ] **XCUT-05**: Profanity screening using curated Malay profanity reject list
- [ ] **XCUT-06**: DBP orthography audit — verify loanwords use Malaysian (-iti, -logi, -si) not Indonesian (-itas, -logi, -sih) spellings
- [ ] **XCUT-07**: Entropy and word count verification per list matches target

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Tooling

- **TOOL-01**: Automated pipeline script wrapping translation → normalization → pruning → verification
- **TOOL-02**: CI pipeline for automated list validation on changes

## Out of Scope

| Feature | Reason |
|---------|--------|
| Bilingual English/Malay documentation | Malay-only audience |
| Indonesian (KBBI) orthography variant | Standard Malay (Malaysia) only |
| Jawi script version | Rumi (Latin) only; password managers expect Latin text |
| Dialectal or regional Malay | Only Standard Malay (DBP) |
| Word frequency or ranking annotations | Plain one-word-per-line format only |
| Automated translation tooling | Translations via DeepSeek v4 Pro, manually directed |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DOC-01 | — | Pending |
| DOC-02 | — | Pending |
| ALFA-01 | — | Pending |
| ALFA-02 | — | Pending |
| QWER-01 | — | Pending |
| QWER-02 | — | Pending |
| SEDE-01 | — | Pending |
| DADU-01 | — | Pending |
| DADU-02 | — | Pending |
| BESA-01 | — | Pending |
| XCUT-01 | — | Pending |
| XCUT-02 | — | Pending |
| XCUT-03 | — | Pending |
| XCUT-04 | — | Pending |
| XCUT-05 | — | Pending |
| XCUT-06 | — | Pending |
| XCUT-07 | — | Pending |

**Coverage:**
- v1 requirements: 17 total
- Mapped to phases: 0
- Unmapped: 17 ⚠️

---
*Requirements defined: 2026-06-30*
*Last updated: 2026-06-30 after initial definition*
