# Roadmap: Jalan Dewan Bahasa Wordlists

## Overview

Translate the Orchard Street Wordlists (7 English passphrase wordlists) into Standard Malay using a semi-automated pipeline: DeepSeek v4 Pro translation → dedup → normalization (Tidy) → Sardinas–Patterson verification → Schlinkert pruning (Tidy `-K`) → final verification (Tidy `-AAAA`). Each wordlist is also published as a SQLite database (`db/*.db`) for programmatic use. Build smallest lists first to validate the pipeline before scaling to the largest (17,576 words).

## Phases

- [x] **Phase 1: Foundation & Documentation** — Malay README + FAQ; establish dedup/normalization tooling
- [x] **Phase 2: Short Lists & Pipeline Validation** — Alpha + QWERTY lists with dice variants; SQLite DBs for each; full pipeline including UD check, pruning, profanity screening, and DBP audit
- [ ] **Phase 3: Medium & Diceware Lists** — Medium (8,192) + diceware (7,776) + clean variant + SQLite DBs
- [ ] **Phase 4: Long List & Final Verification** — Long list (17,576) + SQLite DB + entropy and word count verification across all 7 output files

## Phase Details

### Phase 1: Foundation & Documentation
**Goal**: Malay documentation ready and pipeline tooling established for wordlist production
**Depends on**: Nothing (first phase)
**Requirements**: DOC-01, DOC-02, XCUT-01, XCUT-02
**Success Criteria** (what must be TRUE):
   1. README.md exists in Malay covering project overview, all 7 list descriptions, and CC BY-SA 4.0 licensing
   2. FAQ.md exists in Malay answering common questions about passphrase generation, wordlist usage, and translation approach
   3. Deduplication and normalization tooling is installed, configured with Malay locale (`--locale ms`), and confirmed working on a sample English→Malay translation
   4. The pipeline workflow (translate → dedup → normalize) is reproducible and documented for repeatable use across all list sizes
**Plans**: 2 plans (both complete)
    
Plans:
- [x] `01-01-PLAN.md` — Malay Documentation (README.md + FAQ.md translation)
- [x] `01-02-PLAN.md` — Pipeline Tooling & Setup (Tidy install, scratch dir, pipeline script)

### Phase 2: Short Lists & Pipeline Validation
**Goal**: Both short lists (alpha + QWERTY) complete with dice-roll variants, proving the full pipeline including UD check, Schlinkert pruning, profanity screening, and DBP orthography audit
**Depends on**: Phase 1
**Requirements**: ALFA-01, ALFA-02, QWER-01, QWER-02, SQLI-01, SQLI-02, SQLI-03, SQLI-04, XCUT-03, XCUT-04, XCUT-05, XCUT-06
**Success Criteria** (what must be TRUE):
   1. `jalan-dewan-bahasa-kecil-alpha.txt` exists with exactly 1,296 uniquely decodable Malay words (min 3 chars, alphabetically sorted, keyboard-optimized)
   2. `jalan-dewan-bahasa-kecil-alpha-dadu.txt` exists with same 1,296 words and tab-separated 4-dice-roll prefixes (roll range 1–6)
   3. `jalan-dewan-bahasa-kecil-qwerty.txt` exists with exactly 1,296 uniquely decodable Malay words (QWERTY-optimized, min 3 chars)
   4. `jalan-dewan-bahasa-kecil-qwerty-dadu.txt` exists with same 1,296 words and tab-separated 4-dice-roll prefixes
   5. All short lists pass `tidy -AAAA` (result: "Uniquely decodable? : true"), contain no profane words, and use DBP-standard Malay orthography (no Indonesian -itas/-sih variants)
   6. `db/jalan-dewan-bahasa-kecil-alpha.db`, `kecil-alpha-dadu.db`, `kecil-qwerty.db`, and `kecil-qwerty-dadu.db` exist with correct word tables
**Plans**: 3 plans

Plans:
- [x] `02-01-PLAN.md` — Alpha list translation, pipeline processing, dice variant, and SQLite databases
- [x] `02-02-PLAN.md` — QWERTY list translation, pipeline processing, dice variant, and SQLite databases
- [x] `02-03-PLAN.md` — Profanity screening, DBP orthography audit, and final cross-verification

### Phase 3: Medium & Diceware Lists
**Goal**: Medium (8,192) and diceware (7,776 + clean variant) lists completed through the proven pipeline
**Depends on**: Phase 2
**Requirements**: SEDE-01, DADU-01, DADU-02, SQLI-05, SQLI-06, SQLI-07
**Success Criteria** (what must be TRUE):
   1. `jalan-dewan-bahasa-sederhana.txt` exists with exactly 8,192 uniquely decodable Malay words (~13 bits/word)
   2. `jalan-dewan-bahasa-dadu.txt` exists with exactly 7,776 Malay words with 5-dice-roll prefixes (~12.9 bits/word)
   3. `jalan-dewan-bahasa-dadu-bersih.txt` exists with same 7,776 words without dice-roll prefixes
   4. All lists from this phase pass `tidy -AAAA` unique decodability check and meet word count targets
   5. `db/jalan-dewan-bahasa-sederhana.db`, `dadu.db`, and `dadu-bersih.db` exist with correct word tables
**Plans**: 3 plans

Plans:
- [x] `03-01-PLAN.md` — Medium list (sederhana): Translate 8,192 words → pipeline → SQLite DB
- [x] `03-02-PLAN.md` — Diceware lists (dadu + dadu-bersih): Translate 7,776 words → pipeline → dice + clean variants → 2 SQLite DBs
- [x] `03-03-PLAN.md` — Quality gate: Profanity screening, DBP audit, final verification

### Phase 4: Long List & Final Verification
**Goal**: Long list (17,576) complete; all 7 output files verified for unique decodability, entropy, and word count
**Depends on**: Phase 3
**Requirements**: BESA-01, SQLI-08, XCUT-07
**Success Criteria** (what must be TRUE):
   1. `jalan-dewan-bahasa-besar.txt` exists with exactly 17,576 uniquely decodable Malay words (~14.1 bits/word)
   2. `db/jalan-dewan-bahasa-besar.db` exists with correct word table
   3. All 8 output files (7 .txt + 8 .db) pass verification
   4. All .txt files pass `tidy -AAAA` with result "Uniquely decodable? : true"
   5. All .txt files meet or exceed target entropy (bits/word) and exact word count for each list variant
   6. README.md is updated with final file listing, accurate word counts, and entropy values for all produced lists
**Plans**: 2 plans

Plans:
- [ ] `04-01-PLAN.md` — Long list translation (17,576 words) → pipeline → SQLite database + profanity/DBP screening
- [ ] `04-02-PLAN.md` — Cross-verification of all 8 .txt + 8 .db files + README.md update with final stats

## Progress

| Phase | Reqs | Plans Complete | Status | Completed |
|-------|------|----------------|--------|-----------|
| 1. Foundation & Documentation | 4 | 2/2 | Complete | 2026-06-30 |
| 2. Short Lists & Pipeline Validation | 12 | 3/3 | Complete | 2026-06-30 |
| 3. Medium & Diceware Lists | 6 | 3/3 | Complete | 2026-07-01 |
| 4. Long List & Final Verification | 3 | 0/2 | Planned | - |
