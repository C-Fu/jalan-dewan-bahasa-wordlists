---
phase: 02
plan: 01
status: complete
completed: 2026-06-30
---

# Plan 02-01: Alpha List Translation & Pipeline

## What Was Built

Translated 1,296 English alpha-list words into Standard Malay (DBP orthography), ran the full 7-stage pipeline (normalize → dedup → Schlinkert prune → verify → format → dice), and created 2 SQLite databases. The alpha list is the smallest wordlist serving as pipeline validation.

## Results

| File | Words | Status |
|------|-------|--------|
| `lists/jalan-dewan-bahasa-kecil-alpha.txt` | 1,296 | ✓ Uniquely decodable |
| `lists/jalan-dewan-bahasa-kecil-alpha-dadu.txt` | 1,296 | ✓ Dice format (1111-6666) |
| `db/jalan-dewan-bahasa-kecil-alpha.db` | 1,296 rows | ✓ Integrity ok |
| `db/jalan-dewan-bahasa-kecil-alpha-dadu.db` | 1,296 rows | ✓ Integrity ok |

## Pipeline Stats

- Raw translations: 1,296 lines
- After normalization: 1,017 unique (279 duplicates from many-to-one translations)
- After deduplication: 1,017 unique
- Supplementary words added: ~517 (to fill gap to 1,296+)
- After Schlinkert pruning: 953 → supplemented → 1,428 → whittled to 1,296
- Entropy: 10.340 bits/word (log₂(1296))

## Notable Decisions

- Used model in-chat translation (no DeepSeek API key available)
- Added ~517 supplementary common Malay words to reach target count after dedup losses
- Whittled from 1,428 to exactly 1,296 using tidy `--whittle-to`
- Profanity-cleaned (babi, tahi removed in Plan 02-03)

## Self-Check: PASSED

- tidy -AAAA: Uniquely decodable? : true
- Word count: 1,296
- Entropy: 10.340 bits
- Dice format: 1111-6666, tab-separated
- SQLite integrity: ok
