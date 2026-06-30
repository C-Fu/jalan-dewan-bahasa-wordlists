# Jalan Dewan Bahasa Wordlists

## What This Is

A direct translation of the Orchard Street Wordlists into Standard Malay (Bahasa Melayu, Malaysia orthography), providing Malay speakers with curated passphrase wordlists for strong, secure passphrases. Each English word is translated using DeepSeek v4 Pro, then normalization + Sardinas–Patterson + greedy Schlinkert pruning is applied to restore unique decodability tailored to Malay orthography and target entropy.

## Core Value

Malay speakers can generate secure, uniquely-decodable passphrases using common Malay words — without relying on English wordlists.

## Requirements

### Validated

- 8 English wordlists for passphrase generation exist (long, medium, diceware, alpha, qwerty with dice variants) — existing
- Documentation (readme.markdown, faq.markdown) exists in English — existing
- All English lists are uniquely decodable (Sardinas–Patterson / Schlinkert pruning) — existing
- CC BY-SA 4.0 license applied — existing
- Lists are plain text, one word per line — existing
- Dice variants use tab-separated roll codes — existing

### Active

- [ ] Translate readme.markdown and faq.markdown to Malay-only (`README.md`, `FAQ.md` in Malay)
- [ ] Translate alpha short list → `lists/jalan-dewan-bahasa-kecil-alpha.txt` (+ dice variant)
- [ ] Translate QWERTY short list → `lists/jalan-dewan-bahasa-kecil-qwerty.txt` (+ dice variant)
- [ ] Translate medium list → `lists/jalan-dewan-bahasa-sederhana.txt`
- [ ] Translate diceware list → `lists/jalan-dewan-bahasa-dadu.txt` (+ clean variant `bersih`)
- [ ] Translate long list → `lists/jalan-dewan-bahasa-besar.txt`
- [ ] Apply normalization + Sardinas–Patterson + greedy Schlinkert pruning to each list for Malay orthography
- [ ] Verify unique decodability of each translated list
- [ ] Verify target entropy per list is preserved

### Out of Scope

- English wordlist maintenance — not a fork of the original, it's a translation
- Bilingual documentation — Malay-only
- Indonesian (KBBI) orthography — Standard Malay (Malaysia) only
- Automated translation tooling — translations done via DeepSeek v4 Pro, manually directed

## Context

This is a brownfield project based on the existing Orchard Street Wordlists repository. The codebase has been analysed: `.planning/codebase/` contains 7 documents (STACK, ARCHITECTURE, STRUCTURE, CONVENTIONS, TESTING, INTEGRATIONS, CONCERNS). Translation is phased from smallest lists (alpha/qwerty) to largest (long), with readme/FAQ translated first.

## Constraints

- **License**: CC BY-SA 4.0 — derivative work must maintain same license
- **Orthography**: Standard Malay (Malaysia) — Dewan Bahasa dan Pustaka (DBP) spelling conventions
- **Unique decodability**: Every list must pass Sardinas–Patterson check after translation
- **Entropy**: Target bits/word must match or exceed English originals (long ~14.1, medium 13.0, diceware ~12.9, short ~10.3)
- **List structure**: Same file format as existing lists (one word per line, dice variants with tab-separated roll codes)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Direct translation + re-prune | Maintains Malay word authenticity while restoring unique decodability | — Pending |
| Phased approach (small → medium → diceware → long) | Validate workflow on small lists before committing to large ones | — Pending |
| DeepSeek v4 Pro for translation | User-specified AI translation tool | — Pending |
| Standard Malay (Malaysia) orthography | User specified DBP standard | — Pending |
| Malay-only documentation | Malay-speaking audience | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-06-30 after initialization*
