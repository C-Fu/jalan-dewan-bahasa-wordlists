# Technology Stack — Malay Passphrase Wordlists

**Project:** Jalan Dewan Bahasa Wordlists
**Researched:** 2026-06-30
**Mode:** Ecosystem (subsequent — translation of existing English lists)
**Overall confidence:** HIGH for existing tooling (Tidy), MEDIUM for Malay-specific resources

## Key Constraint

This is a **translation project**, not a greenfield wordlist generation. The English lists exist (17,576 + 8,192 + 7,776 + 1,296 \* 2 + dice variants). The task is: translate each word to Standard Malay, then re-apply normalization, Sardinas–Patterson, and greedy Schlinkert pruning to restore unique decodability for Malay orthography.

This means **no need for word-frequency scraping tools** (Common Word List Maker, Wikipedia Word Frequency). Those were English creation tools. The Malay pipeline is: **translate → normalize → deduplicate → Schlinkert-prune → verify**.

---

## Recommended Stack

### Core Pruning & Verification Tool

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **[Tidy](https://github.com/sts10/tidy)** by sts10 | latest (v0.3.24+, Feb 2026) | Wordlist normalization, Schlinkert pruning, unique decodability verification, dice-roll prefix generation | Same tool used to create the original English lists. Already trusted in the ecosystem. Proven to handle non-English words. Author is the same maintainer as the original Orchard Street lists — most compatible choice. |

**Why Tidy is the right choice and alternatives considered:**

| Decision | Rationale |
|----------|-----------|
| **Use Tidy** (not a custom Python script) | Tidy already implements Sardinas–Patterson (`-AAAA` for audit, `-K` for Schlinkert pruning), Unicode normalization (`-z nfc/nfd/nfkc/nfkd`), locale-sensitive sorting (`--locale`), dice-roll prefix generation (`--dice`), and whittling (`--whittle-to`). Re-implementing any of these in Python would be wasted effort and introduce bugs. |
| **Use Tidy** (not Word List Auditor) | Tidy performs; Auditor only audits. We need to *create* the lists, not just check them. |
| **Use Tidy** (not manual editing) | A 17,576-word list cannot be manually Schlinkert-pruned. |

### Key Tidy flags needed for Malay processing

```bash
# Normalization + pruning pipeline for a translated list:
tidy \
  -z nfkd \                        # Unicode normalization (handles any decomposed chars)
  --locale ms \                    # Malay locale for alphabetic sorting
  -l \                             # lowercase all words
  -K \                             # Schlinkert prune (restore unique decodability)
  --whittle-to <N> \               # maintain exact target size matching English original
  --dice 6 \                       # for dice variants: prepend 5-dice rolls
  -o lists/jalan-dewan-bahasa-<name>.txt \
  translated-raw.txt
```

**Confidence:** HIGH — Tidy's documentation explicitly confirms non-English support (section "Using Tidy with non-English words and/or accented characters"), and it handles NFKD normalization which covers all Latin-script characters used in Malay.

### Translation Tool

| Technology | Purpose | Why |
|------------|---------|-----|
| **DeepSeek v4 Pro** (as specified in PROJECT.md) | Direct English→Malay word translation | Already selected by project owner. Use it for the word translation pass. |

**Important:** DeepSeek is used for *translation only*. The wordlist processing pipeline (cleaning, sorting, UD verification, dice-roll formatting) is entirely handled by Tidy.

### DBP / Orthography Reference (Validation Only)

| Resource | Purpose | Why |
|----------|---------|-----|
| **[Kamus Dewan Edisi Keempat](https://huggingface.co/datasets/mesolitica/kamus-dewan)** (HuggingFace) | Reference for DBP-standard Malay spellings | This is the authoritative dictionary of Standard Malay (Malaysia) as defined by Dewan Bahasa dan Pustaka. Available as structured JSON. Use to verify translated words follow DBP orthography, not Indonesian (KBBI) variants. |
| **[PRPM DBP Online](https://prpm.dbp.gov.my/)** | Spot-check individual words | DBP's online portal for quick lookups. Use for manual sanity checks on ambiguous translations. |
| **[DBP Crawl Dataset](https://huggingface.co/datasets/mesolitica/dbp)** (78,670 words) | Large Malay word list from DBP definitions | Crawled from prpm.dbp.gov.my. Provides a baseline Malay word collection for reference. Useful for identifying non-Malay words that may slip through translation. |

**These are REFERENCE only** — the project translates the English lists directly, it does not create new Malay lists from corpora.

### Malay NLP (Not Needed — Overkill)

**Do NOT use:**

| Library | Why Avoid |
|---------|-----------|
| **[Malaya](https://pypi.org/project/malaya/)** (PyPI, v5.1.1) | Full NLP toolkit (sentiment, NER, parsing) built on PyTorch. 2.6 MB + deep learning dependencies. Completely unnecessary for passphrase wordlist generation. The project translates words, it doesn't analyze Malay text structure. |
| **Malay stemmers** (Sastrawi, etc.) | Malay is agglutinative (pe-, me-, -kan, -i, per-, ber-, ter- affixes). Stemmers try to reduce words to root forms. That's the wrong direction for passphrase lists — we want natural, common Malay words, not roots. The English lists use inflected forms ("abandoned", "abandoning", "abandonment") and the Malay translations should do the same. |
| **Wikipedia Word Frequency tool** | Only supports English and the 19 languages listed in its repo. Malay (ms) is not included. Even if it were, this project translates, not generates from frequency data. |

---

## Malay Orthography Considerations for Processing

### What Tidy Handles Automatically

1. **Unicode normalization** (`-z nfkd`): Malay uses basic Latin script (A-Z, a-z). However, loanwords from Arabic may use characters like `kh`, `gh`, `sy`. Tidy's NFKD normalization handles these correctly.

2. **Locale sorting** (`--locale ms`): Tidy uses Rust's locale system. Malay (`ms`) or Malay/Malaysia (`ms-MY`) locale sorts `e`, `i`, `u` before diacritical variants. This matters for alphabetization in the alpha list.

3. **Lowercasing** (`-l`): Standard Malay orthography is case-sensitive only for proper nouns. Wordlist words should be lowercase.

### What Needs Manual Attention in Translation

1. **E/ə ambiguity**: Malay orthography uses `e` for both `/ə/` (schwa, as in *e*mas) and `/e/` (as in *e*kor). This is a spelling ambiguity that Tidy can't resolve. Translation must ensure consistent Malay orthography.

2. **Affixation and prefix conflicts**: Malay has productive affixation:
   - Prefixes: *meN-*, *peN-*, *ber-*, *ter-*, *per-*, *di-*, *ke-*, *se-*
   - Suffixes: *-kan*, *-i*, *-an*, *-nya*
   - Example: *tulis* (write) → *menulis* (writes), *penulis* (writer), *tulisan* (writing), *menuliskan* (write for), *penulisan* (writing process)
   
   The Schlinkert pruning handles prefix/suffix conflicts algorithmically (same as English), but the translation pass should prefer words that won't be pruned into uselessness. For example, translating "writes" → "menulis" and "writing" (noun) → "tulisan" is fine because Schlinkert will handle any prefix overlap with other words. But if 20% of words end up as affixed variants of the same root, Schlinkert will eat them.

3. **Mixed Malay/Indonesian spellings**: Some words differ between Malaysia (DBP) and Indonesia (KBBI):
   - *negeri* (MS) vs *negara* (standard Indonesian for 'state')
   - *kerana* (MS) vs *karena* (Indonesian)
   - *televisyen* (MS) vs *televisi* (Indonesian)
   - *komputer* (MS) vs *komputer* (same in both)
   
   Translation must insist on Standard Malay (Malaysia) spellings.

---

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Locale | `--locale ms` | Not specifying a locale | Tidy defaults to system LANG, then en-US. Malay sorting order differs from English (e.g., diacritics, digraphs like `kh`, `ng`, `ny`, `sy`). Without `--locale ms`, the alpha list will sort incorrectly for Malay users. |
| Normalization | `-z nfkd` | No normalization | Without normalization, visually identical characters (e.g., composed vs decomposed `e` with acute accent) could appear as separate words, breaking entropy calculations and unique decodability. |
| Pruning | Tidy `-K` | Prefix-only (`-P`) or suffix-only (`-S`) pruning | The English originals used Schlinkert pruning (Sardinas–Patterson algorithm). It removes fewer words than prefix-only pruning while achieving the same unique-decodability. Malay's agglutinative nature makes prefix-only pruning especially destructive. |
| Word generation | Translate English lists | Scrape Malay corpora | The project goal is specifically a translation that maintains the same entropy targets. Scraping Malay corpora would produce different word sets with different frequency characteristics, making it impossible to match the original entropy/\#words targets. |

---

## Dependencies

**Runtime:** None — the wordlists are consumed directly by password managers, just like the English originals.

**Tooling (local/CI):**
- Rust toolchain (to build/install Tidy) — only needed if building from source
- Tidy binary — [pre-built releases available](https://github.com/sts10/tidy/releases) for macOS, Linux, Windows
- Python 3.x (optional) — only if writing custom validation scripts (not strictly needed)

**Storage:** The original English lists total ~46,500 lines. Malay lists will be comparable in size. Trivial even for the smallest CI runners.

---

## Installation

### Install Tidy

```bash
# Option A: Download pre-built binary
curl -L https://github.com/sts10/tidy/releases/latest/download/tidy-x86_64-linux -o tidy
chmod +x tidy
mv tidy ~/.local/bin/

# Option B: Build from source (requires Rust)
cargo install --git https://github.com/sts10/tidy --locked --branch main
```

### Verify Tidy works with Malay

```bash
# Test locale support
tidy --list-locales 2>&1 | grep -i ms
# Expected output includes "ms" or "ms-MY"
```

---

## Sources

| Source | Type | Confidence |
|--------|------|------------|
| [sts10/tidy README](https://github.com/sts10/tidy) — non-English word support section | Official documentation | HIGH |
| [sts10/tidy releases](https://github.com/sts10/tidy/releases) — v0.3.24 (Feb 2026) | Official releases | HIGH |
| [Kamus Dewan Edisi Keempat](https://huggingface.co/datasets/mesolitica/kamus-dewan) — DBP dictionary dataset | Structured data | MEDIUM (dataset may not cover all neologisms) |
| [PRPM DBP Online](https://prpm.dbp.gov.my/) — Official DBP portal | Official reference | HIGH for orthography |
| [Malaysian Dataset](https://github.com/malaysia-ai/malaysian-dataset) — `Malays.dic.txt` (24,550 words) | Community corpus | MEDIUM (may include Indonesian variants) |
| [Malaysian Dataset DBP crawl](https://github.com/malaysia-ai/malaysian-dataset/tree/master/dictionary/dbp) — 78,670 words from DBP definitions | Community corpus | MEDIUM-HIGH (sourced from DBP website) |
| [Malaysian Dataset Ngram](https://github.com/malaysia-ai/malaysian-dataset/tree/master/dictionary/ngram) — Malay Wikipedia unigrams | Community corpus | MEDIUM (not verified for spelling) |
| [Wikipedia - Malay orthography](https://en.wikipedia.org/wiki/Malay_orthography) | Reference | HIGH for historical context |
| Malaya PyPI (v5.1.1, 2024) | Package index | HIGH that it exists, but NOT recommended for this project |

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Tidy capability for Malay | **HIGH** | Confirmed from official docs: Unicode normalization, locale support, Schlinkert pruning are all language-agnostic. |
| DBP resource availability | **MEDIUM** | Kamus Dewan dataset exists but may not cover modern vocabulary. The DBP crawl (78,670 words) is comprehensive for a reference. |
| Malay locale in Tidy | **MEDIUM** | Tidy delegates locale to Rust's `locale` system. Rust supports `ms` locale via ICU — should work on most systems but may need `ms-MY` on some. Need to verify with `--list-locales`. |
| Existing Malay passphrase lists | **HIGH — nothing exists** | No existing Malay diceware or passphrase wordlists found on GitHub or elsewhere. This project fills a genuine gap. |

## What to NOT Use

1. **Common Word List Maker / Wikipedia Word Frequency** — These are English wordlist creation tools. This project translates, not creates from scratch.
2. **Malaya NLP library** — Unnecessary ML dependencies for a text-processing task. Tidy handles everything needed.
3. **Malay stemmers (Sastrawi, etc.)** — Wrong direction; we need natural Malay words, not root forms.
4. **Jawi script resources** — Project scope is Standard Malay in Latin (Rumi) script, not Jawi.
5. **KBBI (Indonesian dictionary) resources** — Project specifies Standard Malay (Malaysia) orthography only. Indonesian has different spelling for some words.
