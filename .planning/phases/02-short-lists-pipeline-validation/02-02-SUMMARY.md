---
phase: 02
plan: 02
status: complete
completed: 2026-06-30
---

# Plan 02-02: QWERTY List Translation & Pipeline

## What Was Built

Translated 1,296 English QWERTY-list words into Standard Malay (DBP orthography), ran the full pipeline, generated the dice variant, and created 2 SQLite databases. The QWERTY list is the companion short list running in parallel with alpha.

## Results

| File | Words | Status |
|------|-------|--------|
| `lists/jalan-dewan-bahasa-kecil-qwerty.txt` | 1,296 | ✓ Uniquely decodable |
| `lists/jalan-dewan-bahasa-kecil-qwerty-dadu.txt` | 1,296 | ✓ Dice format (1111-6666) |
| `db/jalan-dewan-bahasa-kecil-qwerty.db` | 1,296 rows | ✓ Integrity ok |
| `db/jalan-dewan-bahasa-kecil-qwerty-dadu.db` | 1,296 rows | ✓ Integrity ok |

## Pipeline Stats

- Raw translations: 1,296 lines
- After normalization: 1,039 unique (257 duplicates)
- After deduplication: 1,039 unique
- Supplementary words added: ~457
- After Schlinkert pruning: 988 → supplemented → 1,404 → whittled to 1,296
- Entropy: 10.340 bits/word (log₂(1296))

## Notable Decisions

- Used model in-chat translation (no DeepSeek API key available)
- Added ~457 supplementary common Malay words to reach target
- Whittled to exactly 1,296 using tidy `--whittle-to`
- Profanity-cleaned (babi, gua, tahi removed in Plan 02-03)
- Shared 953 words with alpha list (73% overlap — both draw from common Malay vocabulary)

## Self-Check: PASSED

- tidy -AAAA: Uniquely decodable? : true
- Word count: 1,296
- Entropy: 10.340 bits
- Dice format: 1111-6666, tab-separated
- SQLite integrity: ok
