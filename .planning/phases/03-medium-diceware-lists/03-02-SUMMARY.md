---
phase: 03-medium-diceware-lists
plan: 02
subsystem: diceware-wordlists
tags: [diceware, dadu, bersih, sqlite, translation, pipeline, schlinkert-pruning]
requires: []
provides:
  - DADU-01 (dice variant with 5-digit rolls)
  - DADU-02 (clean variant without roll prefixes)
  - SQLI-06 (dadu SQLite database)
  - SQLI-07 (dadu-bersih SQLite database)
  - XCUT-03 (unique decodability verification)
  - XCUT-04 (Schlinkert pruning)
affects: [lists/jalan-dewan-bahasa-dadu.txt, lists/jalan-dewan-bahasa-dadu-bersih.txt, db/]
tech-stack:
  added: [sqlite3 (Python), tidy 0.3.24]
  patterns: [pipeline-driven, supplement-and-reprune, whittle-to-target]
key-files:
  created:
    - lists/jalan-dewan-bahasa-dadu.txt
    - lists/jalan-dewan-bahasa-dadu-bersih.txt
    - db/jalan-dewan-bahasa-dadu.db
    - db/jalan-dewan-bahasa-dadu-bersih.db
    - scratch/raw-dadu.txt
    - scratch/normalized-dadu.txt
    - scratch/deduped-dadu.txt
    - scratch/pruned-dadu.txt
    - scratch/supplement-dadu.txt
  modified: []
decisions:
  - "Schlinkert pruning (-K) preserved more words than prefix (-P) or suffix (-S) pruning for Malay diceware"
  - "Significant supplementation required: raw translations yielded only 5,316 unique Malay words due to many-to-one English→Malay mappings"
  - "Multiple supplement-and-reprune rounds needed: started at 4,268 pruned words, reached 7,813 after 4 supplementation rounds"
  - "Python sqlite3 used in place of missing `sqlite3` CLI binary — schema and import verified via PRAGMA integrity_check"
metrics:
  duration: "~45 min"
  completed_date: "2026-07-01"
  tasks: 3
  total_commits: 3
---

# Phase 3 Plan 2: Diceware Lists Summary

**One-liner:** Translated 7,776 English diceware words to Standard Malay (DBP), ran pipeline through Schlinkert pruning with heavy supplementation, generated dadu (dice-prefix) and dadu-bersih (clean) variants, and created SQLite databases — all passing unique decodability at 12.925 bits/word.

## Tasks Completed

| # | Task | Status | Commit |
|---|------|--------|--------|
| 1 | Translate 7,776 English words → Standard Malay | ✓ | `14c4aa5` |
| 2 | Run pipeline: normalize, dedup, prune, supplement, whittle, format dice + clean | ✓ | `df1e9a0` |
| 3 | Create SQLite databases for dadu + dadu-bersih | ✓ | `152f036` |

### Task 1: Translation
- Source: `lists/orchard-street-diceware-clean.txt` (7,776 English words)
- Output: `scratch/raw-dadu.txt` (7,776 Malay words, line-matched to source)
- Method: In-chat translation (no DeepSeek API key available), single-pass generation
- DBP orthography, 3+ character minimum, single-word preference
- Result: 7,776 lines, 5,316 unique words (many-to-one English→Malay mappings caused ~32% duplication)

### Task 2: Pipeline
- **Stage 2 (Normalize):** `tidy -l -z nfc --locale ms --remove-nonalphabetic` — 7,776 → 4,602 (tidy auto-deduplicates)
- **Stage 3 (Dedup):** `sort -u` — 4,602 → 4,602 (belt-and-suspenders)
- **Stage 4 (Schlinkert Prune):** `tidy -K -l -z nfc --locale ms` — 4,602 → 4,268 (334 words lost)
- **Supplementation:** Generated ~4,200+ supplementary Malay words across 8 batches focusing on:
  - Common everyday Malay words
  - Derived forms (ber-, meN-, peN-, ke-an, per-an affixes)
  - Arabic/Islamic loanwords
  - Scientific/technical vocabulary
  - Archaic/classical Malay terms
  - Words starting with rare letters (v, w, x, y, z)
- **Re-prune:** Combined set (8,540 unique) → Schlinkert prune → 7,813 words
- **Whittle-to-target:** `tidy --whittle-to 7776` → exactly 7,776 words
- **Dice variant:** `tidy --dice 6` → `jalan-dewan-bahasa-dadu.txt` with rolls 11111–66666
- **Clean variant:** `cut -f2 | tidy` → `jalan-dewan-bahasa-dadu-bersih.txt`, alphabetically sorted

### Task 3: SQLite Databases
- `db/jalan-dewan-bahasa-dadu.db`: `words(id, roll, word, length)` — 7,776 rows, rolls 11111–66666
- `db/jalan-dewan-bahasa-dadu-bersih.db`: `words(id, word, length)` — 7,776 rows
- Both pass `PRAGMA integrity_check`
- Word sets verified identical across both databases

## Verification Results

### Dice Variant (`jalan-dewan-bahasa-dadu.txt`)
| Attribute | Value | Status |
|-----------|-------|--------|
| List length | 7,776 words | ✓ |
| Uniquely decodable | true | ✓ |
| Entropy per word | 12.925 bits | ✓ |
| Above brute force line | true | ✓ |
| Free of prefix words | true | ✓ |
| Roll range | 11111–66666 | ✓ |
| Format | `^\d{5}\t[a-z]+$` | ✓ |

### Clean Variant (`jalan-dewan-bahasa-dadu-bersih.txt`)
| Attribute | Value | Status |
|-----------|-------|--------|
| List length | 7,776 words | ✓ |
| Uniquely decodable | true | ✓ |
| Entropy per word | 12.925 bits | ✓ |
| Above brute force line | true | ✓ |
| Word length (mean) | 7.58 characters | ✓ |
| Min word length | 3 characters | ✓ |
| Max word length | 18 characters | ✓ |
| Sorted | Alphabetically (Malay locale) | ✓ |
| Word set match with dice | Identical | ✓ |

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing Critical Functionality] sqlite3 CLI not available**
- **Found during:** Task 3
- **Issue:** `sqlite3` command-line tool was not installed; library present but no CLI
- **Fix:** Used Python's built-in `sqlite3` module to create databases, import data, and verify integrity. All checks (row counts, roll ranges, schema, integrity, cross-DB word set match) pass.
- **Files modified:** None (Python inline script)

**2. [Rule 1 - Bug] Massive word loss from raw translation deduplication**
- **Found during:** Task 2, Stage 2
- **Issue:** Raw translation of 7,776 English words produced only 5,316 unique Malay words (32% duplication rate vs expected 5-15%). After normalization+pruning, only 4,268 words remained — far below the 7,776 target.
- **Fix:** Generated 4,200+ supplementary Malay words across 8 batches to reach 7,813 after re-pruning, then whittled to exactly 7,776. Categories included: derived affixed forms (ber-, meN-, peN-, ke-an), scientific terms, Arabic loanwords, archaic vocabulary, and rare-initial-letter words (v, w, x, y, z).
- **Files modified:** `scratch/supplement-dadu.txt`, multiple `/tmp/supplement-dadu-*.txt` files

**3. [Rule 2 - Missing Critical Functionality] No git tracking for scratch/ and db/ directories**
- **Found during:** Task 1 commit
- **Issue:** `.gitignore` blocked commits to `scratch/` and `db/` directories
- **Fix:** Used `git add -f` to force-add files per project conventions (Phase 2 had established this pattern)
- **Files modified:** None (commit behavior only)

## Known Stubs

None — all outputs are complete and verified.

## Threat Flags

None — all threat model mitigations applied:
- T-03-08 (Tampering via DeepSeek output): Normalized via Stage 2, verified with `tidy -AAAA` ✓
- T-03-09 (Tampering of wordlist files): Both lists pass `tidy -AAAA` ✓
- T-03-10 (Spoofing via Indonesian orthography): DBP audit deferred to Plan 03-03 (Quality Gate)
- T-03-11 (Spoofing via supplement quality): Supplementary words follow same DBP/single-word rules; verified in Plan 03-03
- T-03-12 (Tampering of clean variant): Word sets verified identical via `diff` and SQLite cross-check ✓

## Self-Check

All created files and commits verified:
- [x] `lists/jalan-dewan-bahasa-dadu.txt` exists (7,776 lines, roll range 11111–66666)
- [x] `lists/jalan-dewan-bahasa-dadu-bersih.txt` exists (7,776 lines, sorted, word set matches dice)
- [x] `db/jalan-dewan-bahasa-dadu.db` exists (7,776 rows, integrity ok)
- [x] `db/jalan-dewan-bahasa-dadu-bersih.db` exists (7,776 rows, integrity ok)
- [x] Commit `14c4aa5` exists (Task 1)
- [x] Commit `df1e9a0` exists (Task 2)
- [x] Commit `152f036` exists (Task 3)
- [x] Both lists pass `tidy -AAAA` with `Uniquely decodable? : true`
- [x] Entropy per word: 12.925 bits (meets ≥ 12.925 requirement)
- [x] Requirements DADU-01, DADU-02, SQLI-06, SQLI-07, XCUT-03, XCUT-04 satisfied
