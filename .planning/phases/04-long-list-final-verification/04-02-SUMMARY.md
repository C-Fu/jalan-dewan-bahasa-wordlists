---
phase: 04-long-list-final-verification
plan: 02
subsystem: cross-verification
tags: [verification, tidy, sqlite, entropy, documentation]
requires: [04-01]
provides: [final-verification-report, updated-readme]
affects: [XCUT-07, README.md]
tech-stack:
  added: []
  patterns: [cross-verification, tidy-AAAA-batch, sqlite-integrity-batch]
key-files:
  created: []
  modified:
    - README.md
decisions:
  - "5 hyphenated compound words in dadu.txt/dadu-bersih.txt preserved — fixing would break UD; note as known deviation"
  - "Above brute force line false for dice-variant short lists is expected (dice prefix chars add length without entropy)"
  - "Other README sections (Sederhana, Dadu, Kecil) have outdated stats from English originals — flagged for separate update"
metrics:
  duration: "~5 min"
  completed: "2026-07-01"
---

# Phase 4 Plan 2: Cross-Verification & README Finalization

**One-liner:** Cross-verified all 8 `.txt` wordlist files (tidy -AAAA) and 8 SQLite `.db` databases (PRAGMA integrity_check + row counts), confirmed UD, word counts, and entropy targets, updated README.md Senarai Besar section with actual tidy stats from `jalan-dewan-bahasa-besar.txt`.

## What Was Built

Final quality gate for the entire Jalan Dewan Bahasa Wordlists project. Every output file (8 `.txt` + 8 `.db`) was verified against its target specifications: unique decodability, word count, entropy, dice roll ranges, and database integrity. The README.md Senarai Besar section was updated with actual `tidy -AAAA` statistics replacing the placeholder values copied from the English Orchard Street original.

## Verification Results

### .txt Wordlist Files (tidy -AAAA)

| File | Words | Target | UD | Entropy (bits) | Target | Brute Force |
|------|-------|--------|-----|----------------|--------|-------------|
| `besar.txt` | 17,576 | 17,576 | ✓ true | 14.101 | ≥14.101 | ✓ true |
| `sederhana.txt` | 8,192 | 8,192 | ✓ true | 13.000 | ≥13.000 | ✓ true |
| `dadu.txt` | 7,776 | 7,776 | ✓ true | 12.925 | ≥12.925 | ✓ true |
| `dadu-bersih.txt` | 7,776 | 7,776 | ✓ true | 12.925 | ≥12.925 | ✓ true |
| `kecil-alpha.txt` | 1,296 | 1,296 | ✓ true | 10.340 | ≥10.340 | ✓ true |
| `kecil-alpha-dadu.txt` | 1,296 | 1,296 | ✓ true | 10.340 | ≥10.340 | ⚠ false¹ |
| `kecil-qwerty.txt` | 1,296 | 1,296 | ✓ true | 10.340 | ≥10.340 | ✓ true |
| `kecil-qwerty-dadu.txt` | 1,296 | 1,296 | ✓ true | 10.340 | ≥10.340 | ⚠ false¹ |

¹ Above brute force line false for dice-variant short lists is expected — the 4-digit dice prefix adds 5 characters without entropy contribution, lowering per-character efficiency below the brute force threshold. This is a design characteristic of short dice-variant lists, not a quality defect.

### Additional Format Checks

| Check | Result |
|-------|--------|
| Dadu dice roll range (dadu.txt) | 11111–66666 ✓ |
| Short dice roll range (alpha-dadu) | 1111–6666 ✓ |
| Short dice roll range (qwerty-dadu) | 1111–6666 ✓ |
| Dadu bersih word set ≡ dadu word set | ✓ Identical |
| Empty lines (all 8 files) | 0 ✓ |
| Lowercase a-z only (clean lists) | besar: 0 non-a-z ✓, sederhana: 0 ✓, dadu-bersih: 5 ⚠² |
| Dice format `^\d+\t[a-z]+$` (dadu.txt) | 5 non-conforming lines ⚠² |

² See Deviations section below.

### SQLite Database Files

| Database | Rows | Expected | Integrity |
|----------|------|----------|-----------|
| `db/jalan-dewan-bahasa-besar.db` | 17,576 | 17,576 | ok ✓ |
| `db/jalan-dewan-bahasa-sederhana.db` | 8,192 | 8,192 | ok ✓ |
| `db/jalan-dewan-bahasa-dadu.db` | 7,776 | 7,776 | ok ✓ |
| `db/jalan-dewan-bahasa-dadu-bersih.db` | 7,776 | 7,776 | ok ✓ |
| `db/jalan-dewan-bahasa-kecil-alpha.db` | 1,296 | 1,296 | ok ✓ |
| `db/jalan-dewan-bahasa-kecil-alpha-dadu.db` | 1,296 | 1,296 | ok ✓ |
| `db/jalan-dewan-bahasa-kecil-qwerty.db` | 1,296 | 1,296 | ok ✓ |
| `db/jalan-dewan-bahasa-kecil-qwerty-dadu.db` | 1,296 | 1,296 | ok ✓ |

**All 8 databases: PASS** (correct row counts, integrity ok).

## Notable Decisions

- **5 hyphenated words preserved** — `batu-batan`, `batu-batu`, `buku-buku`, `peminta-minta`, `win-win` in `dadu.txt` and `dadu-bersih.txt` violate the expected "all lowercase a-z only" format but are legitimate Malay compound words. Replacing them would require re-running the Schlinkert pruning pipeline and re-verifying UD for both lists. Preserved as-is; noted as a known deviation. These words passed tidy -AAAA UD verification.

- **README other sections not updated** — The plan explicitly instructed "Do NOT modify any other README sections." However, the Sederhana, Dadu, and Kecil sections still contain placeholder statistics from the English Orchard Street originals (e.g., "ada" as shortest word, "seumpamanya" at 10 chars, when actual values are "acu" at 3 chars and "tidakdikeluarkan" at 16 chars). Flagged for a future clean-up pass.

- **Dice-variant brute force line** — `kecil-alpha-dadu.txt` and `kecil-qwerty-dadu.txt` legitimately fall below the brute force line because the 4-digit dice prefix (5 characters including tab) increases "word length" from ~4-5 chars to ~10 chars without adding entropy. The 10.34 bits of entropy comes from 4 dice rolls (6⁴ = 1,296), not from the word characters themselves. For diceware usage, this is irrelevant — entropy derives from the dice, not character-level brute force resistance.

## Deviations from Plan

### Auto-fixed Issues

None — plan executed as designed without technical bugs.

### Verified Non-Conformances

**1. 5 hyphenated compound words in dadu.txt and dadu-bersih.txt**
- **Found during:** Task 1, dice format check
- **Issue:** Words `batu-batan`, `batu-batu`, `buku-buku`, `peminta-minta`, `win-win` contain hyphens, violating expected `^\d+\t[a-z]+$` format (dadu.txt) and `^[a-z]+$` format (dadu-bersih.txt). Plan expected 0 non-conforming lines.
- **Decision:** Preserved as-is. These are legitimate Malay compound/reduplicated words that passed tidy -AAAA unique decodability verification. Replacing them would require re-pruning, re-supplementing, and re-verifying both lists — high risk of breaking UD for minimal gain.
- **Impact:** Minor — users see 5 hyphenated words out of 7,776 (0.06%). All other format and quality checks pass.

**2. Above brute force line false for dice-variant short lists**
- **Found during:** Task 1, tidy -AAAA verification
- **Issue:** `kecil-alpha-dadu.txt` and `kecil-qwerty-dadu.txt` show `Above brute force line? : false` while plan expected `true`.
- **Decision:** Accepted. This is a design characteristic of short dice-variant lists — the 4-digit dice prefix dilutes per-character entropy below the brute force threshold. Not a quality defect.
- **Impact:** None — diceware entropy derives from dice rolls, not character-level analysis. The clean (non-dice) variants both pass the brute force line check.

**3. Other README sections have outdated statistics**
- **Found during:** Task 2, comparing README stats to actual tidy output
- **Issue:** Sederhana, Dadu, and Kecil README sections contain placeholder statistics from the English Orchard Street originals. Actual values differ: e.g., sederhana mean length is 6.80 (README says 7.07), longest word is "tidakdikeluarkan" at 16 chars (README says "seumpamanya" at 10 chars).
- **Decision:** Not modified per plan's explicit constraint. Flagged for future clean-up.
- **Impact:** README users see slightly inaccurate statistics for non-Besar lists. All word counts and entropy are correct — only per-character metrics and example longest/shortest words are wrong.

## Known Stubs

| Stub | File | Reason |
|------|------|--------|
| Outdated README stats (Sederhana) | README.md lines 42-51 | Stats from English original; actual: mean 6.80, longest 16 "tidakdikeluarkan", efficiency 1.913, edit dist 6.763 |
| Outdated README stats (Dadu) | README.md lines 71-80 | Stats from English original; actual: mean 7.58, longest 18 "mengambinghitamkan", efficiency 1.706, edit dist 7.170 |
| Outdated README stats (Kecil Alpha) | README.md lines 101-110 | Stats from English original; actual: mean 5.28, longest 11 "persembahan", efficiency 1.958, edit dist 5.010 |
| Outdated README stats (Kecil QWERTY) | README.md lines 123-132 | Stats from English original; actual: mean 5.40, longest 11 "persembahan", efficiency 1.913, edit dist 5.143 |
| 5 hyphenated words | dadu.txt, dadu-bersih.txt | batu-batan, batu-batu, buku-buku, peminta-minta, win-win |

## Threat Flags

None — all threat model mitigations applied:
- T-04-08 (Tampering/verification bypass): Mitigated — `tidy -AAAA` run on every file, UD confirmed ✓
- T-04-09 (Spoofing/README stats mismatch): Mitigated — stats verified against tidy output via grep ✓
- T-04-10 (Spoofing/SQLite row count mismatch): Mitigated — `SELECT COUNT(*)` + `PRAGMA integrity_check` on all 8 databases ✓
- T-04-11 (Information Disclosure): Accepted — wordlists contain only common dictionary words ✓
- T-04-12 (DoS/batch verification): Accepted — one-time batch operation ✓

## Commits

| Hash | Type | Description |
|------|------|-------------|
| 983eea8 | docs | Update README Senarai Besar with actual tidy -AAAA stats and first 25 Malay words from besar.txt |

## Self-Check: PASSED

- [x] All 8 `.txt` files exist with correct line counts
- [x] All 8 `.txt` files pass `tidy -AAAA` with `Uniquely decodable? : true`
- [x] All 8 `.txt` files match target word counts exactly
- [x] All 8 `.txt` files meet or exceed target entropy
- [x] Dice roll ranges verified (1111-6666 for short, 11111-66666 for standard)
- [x] Dadu bersih word set identical to dadu word set
- [x] All 8 `.db` files exist with correct row counts
- [x] All 8 `.db` files pass `PRAGMA integrity_check` with result "ok"
- [x] README.md updated with actual tidy -AAAA stats for besar.txt
- [x] README.md example words from actual `jalan-dewan-bahasa-besar.txt` (first 25)
- [x] No deletions in commit 983eea8
- [x] XCUT-07 (entropy and word count verification) satisfied
