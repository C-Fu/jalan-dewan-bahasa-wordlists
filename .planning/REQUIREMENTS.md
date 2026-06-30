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

### SQLite Database (SQLI)

- [ ] **SQLI-01**: `db/jalan-dewan-bahasa-kecil-alpha.db` — SQLite database with `words(roll, word)` table for alpha list
- [ ] **SQLI-02**: `db/jalan-dewan-bahasa-kecil-alpha-dadu.db` — SQLite database with `words(roll, word)` table for alpha dice variant
- [ ] **SQLI-03**: `db/jalan-dewan-bahasa-kecil-qwerty.db` — SQLite database for QWERTY list
- [ ] **SQLI-04**: `db/jalan-dewan-bahasa-kecil-qwerty-dadu.db` — SQLite database for QWERTY dice variant
- [ ] **SQLI-05**: `db/jalan-dewan-bahasa-sederhana.db` — SQLite database for medium list
- [ ] **SQLI-06**: `db/jalan-dewan-bahasa-dadu.db` — SQLite database for diceware list
- [ ] **SQLI-07**: `db/jalan-dewan-bahasa-dadu-bersih.db` — SQLite database for diceware clean variant
- [ ] **SQLI-08**: `db/jalan-dewan-bahasa-besar.db` — SQLite database for long list

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
| DOC-01 | Phase 1 | Complete |
| DOC-02 | Phase 1 | Complete |
| ALFA-01 | Phase 2 | Complete |
| ALFA-02 | Phase 2 | Complete |
| QWER-01 | Phase 2 | Complete |
| QWER-02 | Phase 2 | Complete |
| SEDE-01 | Phase 3 | Pending |
| DADU-01 | Phase 3 | Pending |
| DADU-02 | Phase 3 | Pending |
| BESA-01 | Phase 4 | Pending |
| SQLI-01 | Phase 2 | Complete |
| SQLI-02 | Phase 2 | Complete |
| SQLI-03 | Phase 2 | Complete |
| SQLI-04 | Phase 2 | Complete |
| SQLI-05 | Phase 3 | Pending |
| SQLI-06 | Phase 3 | Pending |
| SQLI-07 | Phase 3 | Pending |
| SQLI-08 | Phase 4 | Pending |
| XCUT-01 | Phase 1 | Complete |
| XCUT-02 | Phase 1 | Complete |
| XCUT-03 | Phase 2 | Complete |
| XCUT-04 | Phase 2 | Complete |
| XCUT-05 | Phase 2 | Complete |
| XCUT-06 | Phase 2 | Complete |
| XCUT-07 | Phase 4 | Pending |

**Coverage:**
- v1 requirements: 25 total (12 complete, 13 pending)
- Mapped to phases: 25
- Unmapped: 0 ✓

---
*Requirements defined: 2026-06-30*
*Last updated: 2026-06-30 after initial definition*
