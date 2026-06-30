# Jalan Dewan Bahasa Wordlists

## What This Is

A direct translation of the Orchard Street Wordlists into Standard Malay (Bahasa Melayu, Malaysia orthography), providing Malay speakers with curated passphrase wordlists for strong, secure passphrases. Each English word is translated using DeepSeek v4 Pro, then normalization + Sardinas–Patterson + greedy Schlinkert pruning is applied to restore unique decodability tailored to Malay orthography and target entropy.

## Core Value

Malay speakers can generate secure, uniquely-decodable passphrases using common Malay words — without relying on English wordlists.

## Current State

**Shipped v1.0** — 8 wordlists + 8 SQLite databases, 25/25 requirements satisfied.

| List | Words | Entropy |
|------|-------|---------|
| Alpha + Alpha Dice | 1,296 | 10.340 bits |
| QWERTY + QWERTY Dice | 1,296 | 10.340 bits |
| Medium (sederhana) | 8,192 | 13.000 bits |
| Diceware (dadu) + Clean | 7,776 | 12.925 bits |
| Long (besar) | 17,576 | 14.101 bits |

## Current Milestone: v1.1 Web Passphrase Generator

**Goal:** A single HTML page using sql.js and the existing wordlist database to generate random Bahasa Melayu passphrases in-browser.

**Target features:**
- Load `jalan-dewan-bahasa-kecil-qwerty.db` via sql.js (WebAssembly SQLite)
- Generate random passphrases with configurable word count
- Dark-mode UI following DESIGN.md (indigo accent, glassy surfaces, system font)
- Pure JavaScript + single HTML file in `html/` directory — zero dependencies beyond sql.js CDN

## Requirements

### Validated

- [x] DOC-01: Malay README.md — v1.0
- [x] DOC-02: Malay FAQ.md — v1.0
- [x] ALFA-01: `jalan-dewan-bahasa-kecil-alpha.txt` (1,296 words, UD, 10.340 bits) — v1.0
- [x] ALFA-02: `kecil-alpha-dadu.txt` with 4-dice-roll prefixes — v1.0
- [x] QWER-01: `jalan-dewan-bahasa-kecil-qwerty.txt` (1,296 words, UD, 10.340 bits) — v1.0
- [x] QWER-02: `kecil-qwerty-dadu.txt` with 4-dice-roll prefixes — v1.0
- [x] SEDE-01: `jalan-dewan-bahasa-sederhana.txt` (8,192 words, UD, 13.000 bits) — v1.0
- [x] DADU-01: `jalan-dewan-bahasa-dadu.txt` (7,776 words, UD, 12.925 bits) — v1.0
- [x] DADU-02: `jalan-dewan-bahasa-dadu-bersih.txt` (7,776 words, same set) — v1.0
- [x] BESA-01: `jalan-dewan-bahasa-besar.txt` (17,576 words, UD, 14.101 bits) — v1.0
- [x] SQLI-01 through SQLI-08: All 8 SQLite databases — v1.0
- [x] XCUT-01: Translation deduplication — v1.0
- [x] XCUT-02: Normalization (NFC, lowercase, non-alphabetic removal) — v1.0
- [x] XCUT-03: Sardinas–Patterson unique decodability check — v1.0
- [x] XCUT-04: Greedy Schlinkert pruning — v1.0
- [x] XCUT-05: Profanity screening (33-entry curated Malay reject list) — v1.0
- [x] XCUT-06: DBP orthography audit (0 Indonesian violations) — v1.0
- [x] XCUT-07: Entropy and word count verification — v1.0

### Active

- [ ] **WEB-01**: HTML page at `html/index.html` loads the QWERTY wordlist SQLite database via sql.js and generates random passphrases from it
- [ ] **WEB-02**: Page follows DESIGN.md visual style (dark mode, #6366F1 primary accent, glassy card surfaces, system font)

### Out of Scope

- English wordlist maintenance — not a fork of the original, it's a translation
- Bilingual documentation — Malay-only
- Indonesian (KBBI) orthography — Standard Malay (Malaysia) only
- Automated translation tooling — translations done via DeepSeek v4 Pro, manually directed

## Context

Shipped v1.0 on 2026-07-01. The project produced 8 Malay wordlist `.txt` files and 8 SQLite `.db` files across 4 phases (10 plans). Pipeline proven: DeepSeek v4 Pro translation → Tidy normalize → dedup → Schlinkert prune → supplement → verify. All lists pass unique decodability, DBP orthography audit, and profanity screening.

Key learnings from v1.0:
- Malay agglutinative morphology causes heavy prefix collisions after translation — Schlinkert re-pruning is essential
- Many-to-one English→Malay translations cause significant word loss (up to 32% for diceware) — multi-round supplementation strategy proven
- DeepSeek API integration needs additional setup; in-chat heuristic translation was used as fallback
- README.md Sederhana/Dadu/Kecil sections still use English placeholder stats (only Besar updated)

## Constraints

- **License**: CC BY-SA 4.0 — derivative work must maintain same license
- **Orthography**: Standard Malay (Malaysia) — Dewan Bahasa dan Pustaka (DBP) spelling conventions
- **Unique decodability**: Every list must pass Sardinas–Patterson check after translation
- **Entropy**: Target bits/word matches or exceeds English originals
- **List structure**: Same file format as existing lists (one word per line, dice variants with tab-separated roll codes)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Direct translation + re-prune | Maintains Malay word authenticity while restoring unique decodability | ✓ Good — all 8 lists pass UD |
| Phased approach (small → medium → diceware → long) | Validate workflow on small lists before committing to large ones | ✓ Good — pipeline refined across phases |
| DeepSeek v4 Pro for translation | AI-assisted translation with manual oversight | ⚠️ Revisit — API not configured; used in-chat fallback |
| Standard Malay (Malaysia) orthography | DBP standard | ✓ Good — 0 Indonesian violations across all lists |
| Malay-only documentation | Malay-speaking audience | ✓ Good — README.md and FAQ.md complete |
| Tidy as sole processing tool | Normalization, dedup, pruning, verification all in one binary | ✓ Good — proven at 17,576 word scale |
| pipeline.sh as workflow docs | Executable reference for the 7-stage pipeline | ✓ Good — reusable across all phases |
| Supplementation strategy | Add common Malay words to fill gaps after dedup + pruning | ✓ Good — closed gaps up to 4,200 words |
| Profanity reject list reuse | 33-entry curated list from Phase 2 reused across all phases | ✓ Good — caught and fixed 11 words in Phase 3, 8 in Phase 4 |
| No cross-list merging | Each list translated and pruned independently | ✓ Good — preserves UD per list |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-07-01 after v1.0 milestone*
