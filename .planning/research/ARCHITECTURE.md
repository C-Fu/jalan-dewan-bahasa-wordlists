# Architecture: Malay Translation Pipeline

**Researched:** 2026-06-30
**Domain:** Bahasa Melayu passphrase wordlist translation and pruning
**Existing system:** Orchard Street Wordlists (English) — flat data/documentation repository, no code

---

## System Overview

The architecture extends the existing **flat data/documentation** model into a **semi-automated pipeline** that transforms English wordlists into Standard Malay (Malaysia/DBP) wordlists while preserving unique decodability and entropy targets. The pipeline is **not a software application** — it is a sequence of manual and tool-assisted steps, each editing or processing plain text files.

### Key Insight

Unique decodability is **language-dependent**. A set of English words that is uniquely decodable does *not* imply that their Malay translations will also be uniquely decodable — prefix and suffix relationships change dramatically across languages. For example:

| English | Malay | Issue |
|---------|-------|-------|
| `book` → `buku` | `bookshelf` → `rak buku` | Two-word translation (space problem) |
| `door` → `pintu` | `doorway` → `pintu gerbang` | The translation may itself contain `pintu` |
| `ant` → `semut` | `semantics` → `semantik` | `semut` is prefix of `semantik` in Malay |

Therefore **every translated list must be re-pruned** using Schlinkert pruning against Malay orthography.

---

## High-Level Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PIPELINE PER ENGLISH WORD LIST                         │
│                                                                           │
│  English list ──► Translation ──► Normalization ──► Dedup ──► Pruning    │
│  (source)        (DeepSeek V4)    (DBP rules)      (merge    (Tidy -K)   │
│                                                    repeats)              │
│                                                                           │
│       Pruned list ──► Verification ──► Formatting ──► Final .txt files   │
│       (uniquely       (Tidy -AAAA,    (sort, dice    (plain + dice       │
│        decodable)      entropy check)  generation)    variants)           │
│                                                                           │
└─────────────────────────────────────────────────────────────────────────┘
```

Each of the 5 English lists flows through this pipeline independently. There is **no cross-list merging** (doing so would almost certainly break unique decodability, per the existing architecture constraint).

---

## Component Boundaries

### 1. Translation Component

| Aspect | Detail |
|--------|--------|
| **Input** | English wordlist (e.g. `orchard-street-alpha.txt` — 1296 lines, one word per line) |
| **Process** | Human-directed DeepSeek v4 Pro translation: each English word → best Standard Malay equivalent |
| **Output** | Raw Malay wordlist (same number of lines, one Malay word or short phrase per line) |
| **Tooling** | DeepSeek v4 Pro (user-specified), edited manually |
| **Location** | Scratch files outside `lists/` (`.gitignore`-ed) |
| **Character set** | Standard Malay Rumi script (Latin), DBP orthography |

#### Translation Rules

| Rule | Reason | Source Priority |
|------|--------|-----------------|
| Use shortest common Malay equivalent | Long phrases reduce usability for passphrase entry | DBP dictionary |
| Prefer single-word translations | Two-word translations like `rak buku` defeat the one-word-per-line format | Passphrase usability |
| Reject profane / culturally inappropriate words | Same convention as English lists | Manual review |
| Keep untranslatable English words only as last resort | Goal is Malay-only lists | Fallback only |
| Note: one Malay word may translate multiple English words | Many-to-one mapping causes duplicates (handled in Dedup) | Pipeline invariant |

#### Translation Edge Cases

| Case | Example | Handling |
|------|---------|----------|
| **Multi-word English** | `bookshelf` | Find single Malay equivalent (`rak buku` → reject, use `almari` if appropriate, or keep `rak_buku` → no, must be single word) |
| **Same Malay for different English words** | `big`/`large` → `besar` | Keep only one instance (handled in Dedup) |
| **No Malay equivalent** | `cappuccino` | Use Malayized borrowing (`kapucino` per DBP) |
| **False friends** | `actual` ≠ `aktual` in Malay | Must verify against DBP standard |
| **Inflection** | Malay is largely uninflected | Simpler than English — fewer morphological variants |

---

### 2. Normalization Component

| Aspect | Detail |
|--------|--------|
| **Input** | Raw Malay wordlist from Translation |
| **Process** | Standardize orthography, lowercase, strip diacritics (if needed), handle whitespace |
| **Output** | Normalized Malay wordlist |
| **Tooling** | Manual or scripted (`sed`/`awk` or Tidy `-l -z nfc`) |

#### Normalization Rules

| Rule | Rationale |
|------|-----------|
| Lowercase all words | Same convention as English lists (Tidy `-l`) |
| Apply NFC Unicode normalization | Prevents duplicate-looking words with decomposed diacritics (Tidy `-z nfc`) |
| Remove leading/trailing whitespace | Tidy does this automatically |
| Replace curly quotes with straight | Tidy `-q` flag |
| Strip non-alphabetic characters | Malay words only — no digits, no punctuation (Tidy `--remove-nonalphabetic`) |

#### Diacritics Policy

**Decision needed:** Malay Rumi script in Malaysia typically does not use diacritics in everyday writing (unlike Indonesian which uses `é`). DBP orthography is strictly a-z. However, some proper Malay spellings may use diacritics.

| Option | Implication |
|--------|-------------|
| Strip all diacritics (a-z only) | Max compatibility with password managers; aligns with existing a-z convention |
| Keep DBP diacritics | More orthographically correct; may break some tool consumers |

**Recommendation:** Strip to a-z to match the existing convention and ensure KeePassXC/Buttercup compatibility. The PROJECT.md specifies DBP spelling conventions, but DBP's standard orthography for everyday words already uses plain a-z.

---

### 3. Deduplication Component

| Aspect | Detail |
|--------|--------|
| **Input** | Normalized Malay wordlist |
| **Process** | Identify Malay words that appear more than once (multiple English → same Malay); keep one, note the loss |
| **Output** | Deduplicated Malay wordlist (length ≤ original English list length) |
| **Tooling** | `sort -u` or Tidy (auto-removes duplicates) |

#### Impact Assessment

| List | English Size | Estimated Malay Size After Dedup | Entropy Impact |
|------|-------------|----------------------------------|----------------|
| Alpha | 1296 | 1250–1290 (low collision) | Minimal (~10.3 → ~10.29 bits) |
| QWERTY | 1296 | 1250–1290 (low collision) | Minimal |
| Medium | 8192 | 7900–8190 | Minimal |
| Diceware | 7776 | 7500–7750 | Minimal |
| Long | 17576 | 17000–17500 | Minimal |

The highest collision risk is among synonyms (e.g. `big`/`large` → `besar`, `buy`/`purchase` → `beli`). Short lists (alpha, qwerty) may avoid this because they already select one variant. The medium/diceware/long lists will have more collisions.

**Mitigation if too many duplicates:** Replace duplicated words with Malay synonyms (rich synonym set in Malay), but this may require re-running Schlinkert pruning.

---

### 4. Schlinkert Pruning Component

This is the **critical architectural component** that restores unique decodability after translation.

| Aspect | Detail |
|--------|--------|
| **Input** | Deduplicated Malay wordlist |
| **Process** | Run Sardinas–Patterson algorithm iteratively; remove minimum words to eliminate ambiguity |
| **Output** | Pruned Malay wordlist (uniquely decodable) |
| **Tooling** | `tidy -K --schlinkert-prune -z nfc --locale ms-MY` |
| **How it works** | 1. Compute "dangling suffixes" between all word pairs<br>2. Check if any dangling suffix exists as a word in the list<br>3. If yes: list is NOT uniquely decodable<br>4. Remove offending words (those in the intersection of the suffix set and the word set)<br>5. Run again — try both forward and reversed word forms; keep whichever preserves more words |

#### Why Malay-specific Pruning is Necessary

Malay has different word formation patterns than English:

| Pattern | Examples | UD Risk |
|---------|----------|---------|
| Prefixes `meN-`, `ber-`, `ter-`, `di-`, `ke-` | `baca` / `membaca`, `lari` / `berlari` | HIGH — `baca` is prefix of `membaca` |
| Suffixes `-an`, `-kan`, `-i` | `makan` / `makanan`, `tulis` / `tuliskan` | HIGH — `makan` is prefix of `makanan` |
| Reduplication | `buku` / `buku-buku` | MEDIUM — hyphen issue |
| Compounds | `air mata`, `tanggung jawab` | LOW — two-word compounds unlikely in wordlist |
| Loanwords | `universiti`, `kualiti`, `komputer` | LOW — generally unique |

The Schlinkert pruning algorithm handles these automatically — no manual prefix analysis needed. This is the **key reason** we use Schlinkert pruning over simple prefix-stripping.

#### Expected Word Loss

| List | English Size | Expected Post-Prune Size (Malay) | Confidence |
|------|-------------|----------------------------------|------------|
| Alpha | 1296 | 1200–1280 | LOW — depends on Malay prefix relations |
| QWERTY | 1296 | 1200–1280 | LOW |
| Medium | 8192 | 7600–8100 | LOW |
| Diceware | 7776 | 7200–7700 | LOW |
| Long | 17576 | 16000–17400 | LOW |

**If loss is too severe:** Consider Tidy's `-P` (prefix removal) or `-S` (suffix removal) as alternatives — Schlinkert pruning should preserve *more* words than either, but if it produces anomalous results (as with BIPS39 where `-P` preserved more), try the alternative.

#### Tidy Command for Pruning

```bash
# Run the Malay list through Tidy with Schlinkert pruning
tidy -K \
     -l \                  # lowercase
     -z nfc \              # NFC Unicode normalization
     --locale ms-MY \      # Malay (Malaysia) locale for sorting
     -o pruned_list.txt \  # output
     deduped_list.txt      # input

# Also try with -P or -S if -K loses too many words
tidy -P -l -z nfc --locale ms-MY -o pruned_list.txt deduped_list.txt
```

---

### 5. Verification Component

| Aspect | Detail |
|--------|--------|
| **Input** | Pruned Malay wordlist |
| **Process** | Check unique decodability, count words, compute entropy, verify against targets |
| **Output** | Verification report + confirmed-ready list |
| **Tooling** | `tidy -AAAA -z nfc --locale ms-MY` produces full attribute report |

#### Verification Checks

| Check | Command/Tool | Pass/Fail |
|-------|-------------|-----------|
| Uniquely decodable? | `tidy -AAAA` → `Uniquely decodable? : true` | **REQUIRED** |
| Word count ≥ target | Manual count vs expected | Must meet or exceed |
| Entropy ≥ English original | `tidy -AAAA` → `Entropy per word` | Must meet or exceed |
| No duplicates | Tidy auto-removes (check `List length` = unique count) | **REQUIRED** |
| No profanity | Manual review + reject list | **REQUIRED** |
| Alphabetically sorted | Tidy does by default | Expected |
| Dice codes correct (dice variant) | Verify format: `digits\tword` | **REQUIRED** |

#### Entropy Targets (from PROJECT.md)

| List | English Bits/Word | Target Malay Bits/Word |
|------|-------------------|----------------------|
| Long (besar) | ~14.1 | ≥ 14.1 |
| Medium (sederhana) | 13.0 | ≥ 13.0 |
| Diceware (dadu) | ~12.9 | ≥ 12.9 |
| Alpha/kecil-alpha | ~10.3 | ≥ 10.3 |
| QWERTY/kecil-qwerty | ~10.3 | ≥ 10.3 |

If deduplication + pruning reduces word count too much (< target), need to find replacement words and re-run pruning.

---

### 6. Formatting Component

| Aspect | Detail |
|--------|--------|
| **Input** | Verified clean Malay wordlist |
| **Process** | Sort, generate dice variant with tab-separated roll codes |
| **Output** | `jalan-dewan-bahasa-{type}.txt` + `jalan-dewan-bahasa-{type}-dadu.txt` |
| **Tooling** | `tidy --dice 6 -o output.txt input.txt` for dice variant |

#### Output File Mapping

| English Source | Malay Plain File | Malay Dice File | Dice Roll Length |
|---------------|------------------|-----------------|------------------|
| alpha.txt | `jalan-dewan-bahasa-kecil-alpha.txt` | `jalan-dewan-bahasa-kecil-alpha-dadu.txt` | 4 digits (6^4 = 1296) |
| qwerty.txt | `jalan-dewan-bahasa-kecil-qwerty.txt` | `jalan-dewan-bahasa-kecil-qwerty-dadu.txt` | 4 digits (6^4 = 1296) |
| medium.txt | `jalan-dewan-bahasa-sederhana.txt` | (no dice variant needed) | N/A |
| diceware.txt | `jalan-dewan-bahasa-dadu-bersih.txt` | `jalan-dewan-bahasa-dadu.txt` | 5 digits (6^5 = 7776) |
| long.txt | `jalan-dewan-bahasa-besar.txt` | (no dice variant needed) | N/A |

#### Formatting Rules

- Plain files: one word per line, lowercase, sorted alphabetically
- Dice files: `digits\tword`, sorted by dice code (which Tidy does automatically with `--dice`), tab-separated
- All files: UTF-8 without BOM, no trailing whitespace, final newline
- Dice codes: 4 digits for 1296-word lists, 5 digits for 7776-word lists (matching English convention)

---

## Independent Workstream: Documentation

Documentation translation is an **independent workstream** that has no data dependencies on the wordlist pipeline, but its file references must match the final output filenames.

```
┌──────────────────────┐
│ Documentation Lane    │
│                       │
│ English readme/FAQ    │
│     │                 │
│     ▼                 │
│ Malay README.md       │
│ Malay FAQ.md          │
│     │                 │
│     ▼                 │
│ Link to translated    │
│ wordlist files        │
└──────────────────────┘
```

**Convention decision:** The existing project uses `.markdown` extension for documentation. PROJECT.md says Malay versions should be `README.md` and `FAQ.md`. Recommend using `.md` for Malay files to differentiate from English originals, unless there's a reason to stay consistent with `.markdown`.

---

## Data Flow Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        MASTER DATA FLOW                               │
│                                                                        │
│  For each of 5 English lists (alpha, qwerty, medium, diceware, long): │
│                                                                        │
│  Source: lists/orchard-street-{type}.txt                               │
│     │                                                                   │
│     ▼ scratch/raw-{type}.txt                                           │
│  [TRANSLATION: DeepSeek v4 → human editing]                            │
│     │                                                                   │
│     ▼ scratch/normalized-{type}.txt                                    │
│  [NORMALIZATION: Tidy -l -z nfc --locale ms-MY]                        │
│     │                                                                   │
│     ▼ scratch/deduped-{type}.txt                                       │
│  [DEDUP: Tidy auto or sort -u, verify manually]                        │
│     │                                                                   │
│     ▼ scratch/pruned-{type}.txt                                        │
│  [PRUNING: Tidy -K -l -z nfc --locale ms-MY]                          │
│     │                                                                   │
│     ▼ [VERIFICATION: Tidy -AAAA, manual checks]                        │
│     │                                                                   │
│     ├──► lists/jalan-dewan-bahasa-{type}.txt                           │
│     │    (clean, sorted, verified)                                      │
│     │                                                                   │
│     └──► lists/jalan-dewan-bahasa-{type}-dadu.txt                      │
│          (dice variant, if exists)                                      │
│                                                                        │
│  Documentation lane (independent):                                     │
│     readme.markdown ──► README.md                                      │
│     faq.markdown   ──► FAQ.md                                          │
└──────────────────────────────────────────────────────────────────────┘
```

**State management:** No runtime state — all intermediate files are either `.gitignore`-ed scratch files or committed final outputs. The pipeline is "cold start" each time.

---

## Build Order and Dependencies

### Phase Structure

```
Phase 0: Documentation                   ─── No dependencies
  ├── Translate README.md                ─── Must reference final filenames
  └── Translate FAQ.md                        (can use placeholder names, update later)

Phase 1: Kecil-alpha (1296 words)        ─── Validates pipeline on smallest list
  ├── Translate alpha list               ─── Establishes translation patterns
  ├── Normalize + Dedup                  ─── Establishes normalization rules
  ├── Schlinkert prune                   ─── Proves pruning works for Malay
  └── Verify + Format                    ─── Proves entropy targets achievable

Phase 2: Kecil-qwerty (1296 words)      ─── Same size, different vocabulary
  ├── Translate qwerty list              ─── Reuses patterns from Phase 1
  ├── Normalize + Dedup + Prune          ─── Confidence: same approach
  └── Verify + Format

Phase 3: Sederhana (8192 words)          ─── Scales to medium size
  ├── Translate medium list              ─── More vocabulary diversity
  ├── Normalize + Dedup + Prune          ─── Pruning time will be longer
  └── Verify + Format

Phase 4: Dadu (7776 words)              ─── Diceware + clean variant
  ├── Translate diceware list
  ├── Normalize + Dedup + Prune
  ├── Verify + Format (clean)
  └── Generate dice variant (bersih)

Phase 5: Besar (17576 words)            ─── Largest list, last
  ├── Translate long list
  ├── Normalize + Dedup + Prune          ─── Pruning may take hours/days
  └── Verify + Format

Phase 6: Final Cross-Verification        ─── Verify all lists together
  ├── Re-verify unique decodability of all 5 lists
  ├── Re-check entropy targets
  ├── Update README.md with final filenames
  └── Update .gitignore for scratch artifacts
```

### Dependency Graph

```
Phase 0 ──► Phase 1 ──► Phase 2 ──► Phase 3 ──► Phase 4 ──► Phase 5 ──► Phase 6
  │            │            │            │            │            │
  │            └────No cross-list dependencies────────────┘            │
  │                                                                     │
  └─────────────Documentation finalization depends on Phase 1-5────────┘
                         (file names, word counts)
```

**Key insight:** Each wordlist phase is **independent**. They don't share data. The ordering is purely about risk management — prove the pipeline on small lists first.

---

## Tidy Configuration for Malay Lists

### Recommended Tidy Profile

```bash
# A typical translation→pruning run for a Malay word list

# Step 1: Normalize after translation
tidy -l \
     -z nfc \
     --locale ms-MY \
     --remove-nonalphabetic \
     -o scratch/normalized-{type}.txt \
     scratch/raw-{type}.txt

# Step 2: Schlinkert prune
tidy -K \
     -l \
     -z nfc \
     --locale ms-MY \
     -o scratch/pruned-{type}.txt \
     scratch/normalized-{type}.txt

# Step 3: Verify
tidy -AAAA \
     -z nfc \
     --locale ms-MY \
     scratch/pruned-{type}.txt

# Step 4: Generate dice variant (for diceware/alpha/qwerty)
tidy --dice 6 \
     -z nfc \
     --locale ms-MY \
     -o lists/jalan-dewan-bahasa-{type}-dadu.txt \
     scratch/pruned-{type}.txt

# Step 5: Output clean sorted list
tidy -z nfc \
     --locale ms-MY \
     -o lists/jalan-dewan-bahasa-{type}.txt \
     scratch/pruned-{type}.txt
```

### Key Flags Explained for Malay

| Flag | Purpose for Malay |
|------|-------------------|
| `-z nfc` | NFC normalization prevents duplicate detection failures from decomposed Unicode characters (e.g., `é` as U+00E9 vs U+0065 U+0301) |
| `--locale ms-MY` | Ensures Malay-appropriate alphabetical sort order (e.g., `c` after `b`, not treated as English) |
| `--remove-nonalphabetic` | Strips any stray punctuation or digits — Malay words should be a-z only |
| `-l` | Lowercase — all Malay words lowercase like English originals |
| `-K` | Schlinkert prune — the core algorithm; may take hours on long list |
| `--dice 6` | Generates 4- or 5-digit dice codes based on list length |

---

## Scratch File Management

Intermediate pipeline artifacts should be stored outside version control to avoid clutter.

| File | Location | Tracked? |
|------|----------|----------|
| Raw translation output | `scratch/raw-{type}.txt` | No (gitignore) |
| Normalized list | `scratch/normalized-{type}.txt` | No (gitignore) |
| Deduplicated list | `scratch/deduped-{type}.txt` | No (gitignore) |
| Pruned list (pre-verification) | `scratch/pruned-{type}.txt` | No (gitignore) |
| Final clean list | `lists/jalan-dewan-bahasa-{type}.txt` | **Yes** |
| Final dice variant | `lists/jalan-dewan-bahasa-{type}-dadu.txt` | **Yes** |
| Documentation | `README.md`, `FAQ.md` | **Yes** |

**Update `.gitignore`:** Add `scratch/` directory to the existing gitignore patterns.

```gitignore
# Scratch files from translation pipeline
scratch/

# Existing ignores (keep)
blended-source.txt
words-to-blend-in.txt
justfile
.justfile
```

---

## Architecture Constraints (Inherited + New)

### Inherited from Existing Architecture

| Constraint | Why It Applies to Malay |
|------------|------------------------|
| **No code** | Repo remains a data/documentation project; pipeline tooling is external |
| **Static data only** | Lists are committed as-is, never generated at build time |
| **Unique decodability** | Every Malay must pass Sardinas–Patterson check |
| **No list merging** | Combining Malay lists would break UD — same as English |
| **Line format** | Plain: one word per line; Dice: `digits\tword` |
| **a-z characters only** | DBP orthography is a-z; diacritics removed for compatibility |

### New Constraints (Translation Pipeline)

| Constraint | Rationale |
|------------|-----------|
| **No two-word translations** | `air mata` would require spaces; passphrase generators expect single words |
| **One-to-keep mapping** | Multiple English → same Malay: keep one, discard rest |
| **Translation is manual/human-directed** | No automated MT pipeline — DeepSeek v4 Pro output must be reviewed |
| **Scratch files outside VCS** | Keep repository clean; only final lists committed |
| **Preserve original entropy targets** | Malay lists must match or exceed English bits/word |

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Many English words → same Malay word | Medium | Reduced entropy | Replace duplicates with Malay synonyms; re-prune |
| Malay morphology causes excessive pruning loss | Medium | Word count drops below target | Try both `-K`, `-P`, `-S`; pick best result |
| Tidy doesn't handle Malay locale correctly | Low | Wrong sort order | Verify sort manually; test with sample |
| Two-word translations unavoidable | Medium | Some English concepts have no single Malay word | Reject those words; find alternatives |
| Diacritics break password manager compatibility | Low | Words not recognized | Strip all diacritics to a-z |
| Schlinkert pruning on 17576-word list takes too long | Medium | Delays completion | Run overnight; blog post says it may take hours/days |

---

## Sources

- **Tidy README** (sts10/tidy, v0.3.24, February 2026): https://github.com/sts10/tidy — HIGH confidence
- **Schlinkert Pruning Blog Post** (sts10, August 2022): https://sts10.github.io/2022/08/12/efficiently-pruning-until-uniquely-decodable.html — HIGH confidence
- **Existing architecture analysis**: `.planning/codebase/ARCHITECTURE.md` — HIGH confidence
- **Project requirements**: `.planning/PROJECT.md` — HIGH confidence
- **Existing conventions**: `.planning/codebase/CONVENTIONS.md` — HIGH confidence
- **Sardinas–Patterson algorithm**: https://en.wikipedia.org/wiki/Sardinas%E2%80%93Patterson_algorithm — HIGH confidence
- **Dewan Bahasa dan Pustaka orthography**: General knowledge; DBP spelling conventions for Standard Malay — MEDIUM confidence (should be verified against official DBP reference if orthographic edge cases arise)

---

*Architecture analysis for Malay translation pipeline: 2026-06-30*
