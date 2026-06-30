# Roadmap: Jalan Dewan Bahasa Wordlists

## Overview

Translate the Orchard Street Wordlists (7 English passphrase wordlists) into Standard Malay using a semi-automated pipeline: DeepSeek v4 Pro translation → dedup → normalization (Tidy) → Sardinas–Patterson verification → Schlinkert pruning (Tidy `-K`) → final verification (Tidy `-AAAA`). Build smallest lists first to validate the pipeline before scaling to the largest (17,576 words).

## Phases

- [ ] **Phase 1: Foundation & Documentation** — Malay README + FAQ; establish dedup/normalization tooling
- [ ] **Phase 2: Short Lists & Pipeline Validation** — Alpha + QWERTY lists with dice variants; full pipeline including UD check, pruning, profanity screening, and DBP audit
- [ ] **Phase 3: Medium & Diceware Lists** — Medium (8,192) + diceware (7,776) + clean variant
- [ ] **Phase 4: Long List & Final Verification** — Long list (17,576) + entropy and word count verification across all 7 output files

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
**Plans**: TBD

### Phase 2: Short Lists & Pipeline Validation
**Goal**: Both short lists (alpha + QWERTY) complete with dice-roll variants, proving the full pipeline including UD check, Schlinkert pruning, profanity screening, and DBP orthography audit
**Depends on**: Phase 1
**Requirements**: ALFA-01, ALFA-02, QWER-01, QWER-02, XCUT-03, XCUT-04, XCUT-05, XCUT-06
**Success Criteria** (what must be TRUE):
  1. `jalan-dewan-bahasa-kecil-alpha.txt` exists with exactly 1,296 uniquely decodable Malay words (min 3 chars, alphabetically sorted, keyboard-optimized)
  2. `jalan-dewan-bahasa-kecil-alpha-dadu.txt` exists with same 1,296 words and tab-separated 4-dice-roll prefixes (roll range 1–6)
  3. `jalan-dewan-bahasa-kecil-qwerty.txt` exists with exactly 1,296 uniquely decodable Malay words (QWERTY-optimized, min 3 chars)
  4. `jalan-dewan-bahasa-kecil-qwerty-dadu.txt` exists with same 1,296 words and tab-separated 4-dice-roll prefixes
  5. All short lists pass `tidy -AAAA` (result: "Uniquely decodable? : true"), contain no profane words, and use DBP-standard Malay orthography (no Indonesian -itas/-sih variants)
**Plans**: TBD

### Phase 3: Medium & Diceware Lists
**Goal**: Medium (8,192) and diceware (7,776 + clean variant) lists completed through the proven pipeline
**Depends on**: Phase 2
**Requirements**: SEDE-01, DADU-01, DADU-02
**Success Criteria** (what must be TRUE):
  1. `jalan-dewan-bahasa-sederhana.txt` exists with exactly 8,192 uniquely decodable Malay words (~13 bits/word)
  2. `jalan-dewan-bahasa-dadu.txt` exists with exactly 7,776 Malay words with 5-dice-roll prefixes (~12.9 bits/word)
  3. `jalan-dewan-bahasa-dadu-bersih.txt` exists with same 7,776 words without dice-roll prefixes
  4. All lists from this phase pass `tidy -AAAA` unique decodability check and meet word count targets
**Plans**: TBD

### Phase 4: Long List & Final Verification
**Goal**: Long list (17,576) complete; all 7 output files verified for unique decodability, entropy, and word count
**Depends on**: Phase 3
**Requirements**: BESA-01, XCUT-07
**Success Criteria** (what must be TRUE):
  1. `jalan-dewan-bahasa-besar.txt` exists with exactly 17,576 uniquely decodable Malay words (~14.1 bits/word)
  2. All 7 output files pass `tidy -AAAA` with result "Uniquely decodable? : true"
  3. All 7 output files meet or exceed target entropy (bits/word) and exact word count for each list variant
  4. README.md is updated with final file listing, accurate word counts, and entropy values for all produced lists
**Plans**: TBD

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Documentation | 0/0 | Not started | - |
| 2. Short Lists & Pipeline Validation | 0/0 | Not started | - |
| 3. Medium & Diceware Lists | 0/0 | Not started | - |
| 4. Long List & Final Verification | 0/0 | Not started | - |
