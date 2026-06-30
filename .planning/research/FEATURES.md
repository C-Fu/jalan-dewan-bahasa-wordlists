# Feature Landscape: Malay Passphrase Wordlists

**Domain:** Malay (Bahasa Melayu, Malaysia orthography) passphrase wordlists — a direct translation of the Orchard Street Wordlists
**Researched:** 2026-06-30

## Table Stakes

Features users expect from any passphrase wordlist. Missing these = the wordlist is unusable for passphrase generation.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Unique decodability** (Sardinas–Patterson) | Passphrases can be concatenated without delimiters. Without this, users need separators between words, reducing entropy and adding cognitive load. This is the defining feature of Orchard Street lists and must carry over. | HIGH | Requires running Sardinas–Patterson + Schlinkert pruning per list. Must pass `Free of prefix words? : false` + `Uniquely decodable? : true`. |
| **Common, recognizable Malay words** | Users must be able to read, spell, and recall each word. Niche/archaic Malay words ("berinda", "sewaktu", "lembaga" outside common usage) defeat memorability. Original list used Wikipedia + Google Books frequency data; translated words inherit English commonality but must be common in Malay too. | MEDIUM | Translation must prioritize common Malay equivalents. If a common English word has a rare Malay translation, consider a synonym. Requires manual pass. |
| **No profanity** | The original English list explicitly bans profanity. Malay has distinct profane words ("bodoh", "babi", "celaka", "sial", "puki", "kontol", etc.) plus culturally sensitive religious terms. Any profanity in a passphrase list is unacceptable. | MEDIUM | Need a curated Malay profanity reject list. Cannot reuse English reject list directly — cultural taboos differ. Religious profanity uniquely important in Malay context. |
| **Standard Malay (DBP) orthography** | Users expect post-1972 spelling reform (Ejaan Rumi Baharu). No Za'aba-era spellings, no Indonesian variants (KBBI). E.g., "sistem" not "sistim", "universiti" not "universitas", "teksi" not "taksi" (specific differences). | LOW | Direct translation should produce DBP-standard. Verify loanwords especially. |
| **Lowercase only** | Original convention: all words lowercase. Malay proper nouns (usually capitalised) must be lowercased on the list. | LOW | Trivial with Tidy `-l` flag. |
| **No abbreviations** | Original explicitly bans abbreviations. Malay SMS-short forms ("dg" for "dengan", "utk" for "untuk", "kp" for "kepada", "yg" for "yang", "dgn" for "dengan") must not appear. | LOW | Abbreviations typically don't exist in English source words, but translations should expand any that slip through. |
| **One word per line, plain text** | Standard for all passphrase generators (KeePassXC, Buttercup, Strongbox, etc.). Must be pure plain text, no metadata, no formatting. UTF-8 with no BOM. | LOW | Check after generation. |
| **Target entropy preserved** | Users expect specific entropy per list: long ~14.1 bits/word, medium 13.0, diceware ~12.9, short ~10.3. Translated lists must hit these targets. | HIGH | Entropy = log2(list_length). Exact word counts must be preserved: long=17,576, medium=8,192, diceware=7,776, short=1,296. If pruning removes too many words, need a fallback. |
| **No punctuation, digits, or spaces** | Original convention: alphabetical characters only. Malay words with apostrophes (e.g., "Kaabah" — properly "Kaabah" but some spellings use apostrophe), hyphens ("Jawi" — no hyphen generally), or numbers must be excluded. Malay loanwords sometimes have apostrophes. | LOW | Check each word. Malay tends to avoid apostrophes in standard orthography ("Quran" not "Qur'an" in MS spelling). |
| **Minimum 3-character words** | Original US English lists enforce 3-char minimum. Malay's shortest common words are "di", "ke", "ia", "dan", "ini", "itu" (2 chars). All wordlists at 7,776+ words cross the "brute force line" only with minimum-3. With 2-char words, the list would need to be ≤676 words to stay above the brute force line. | MEDIUM | This is a real tension in Malay. Many core function words are 2 letters. Lists at 1,296+ words MUST use minimum-3. Needs explicit justification in documentation. |
| **Diceware format variant** | Password managers (Strongbox) and CLI tools (phraze, passphraseme) expect tab-separated roll-code format for physical dice users. | LOW | Use Tidy `--dice 6` on the pruned plain list to generate dice variant. |
| **CC BY-SA 4.0 license** | Original license. Must carry over. All documentation must acknowledge. | LOW | License file and attribution text are trivially transplantable. |
| **Malay documentation** | The point of this project is serving Malay speakers. README and FAQ must be in Malay (not English). | LOW | Translation of readme.markdown and faq.markdown. |

## Differentiators

Features that distinguish Jalan Dewan Bahasa from alternative approaches (e.g., a hypothetical Malay Diceware, or using English lists for Malay speakers).

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Schlinkert pruning (not prefix-free)** | Malay has pervasive prefixing morphology (me-, mem-, men-, meng-, memper-, ber-, ter-, pe-, per-, ke-, se-, di-, etc.). A naive prefix-free approach would remove thousands of valid common words. Schlinkert pruning preserves prefix-containing words that don't actually cause decoding ambiguity, yielding a richer list. | HIGH | This is the key technical differentiator. Original English had same approach; Malay makes it even more valuable. Tidy's `--schlinkert-prune` (`-K`) flag implements this but is marked experimental — may need manual approach. |
| **Malay-morphology-aware pruning** | Beyond generic Schlinkert, understanding Malay agglutination patterns (overlapping prefixes and suffixes, circumfixes like me-/-kan, ber-/-an) can guide what to prioritize keeping vs cutting when pruning is necessary. | MEDIUM | Not automated in Tidy — requires manual judgment. The translation itself may produce fewer homograph problems than English because Malay spelling is more phonemic. |
| **DBP-specified loanword treatment** | Malaysia-specific loanword spelling differs from both English source and Indonesian. Correct handling of "teknologi", "kualiti", "universiti", "komputer", "strategi", "demokrasi", etc. signals authority and quality. | MEDIUM | DeepSeek v4 Pro must be instructed to use Malaysian DBP spelling for loanwords, not English or Indonesian forms. |
| **Five list sizes matching existing ecosystem** | Password managers already integrate with Orchard Street. Malay users can switch to Jalan Dewan Bahasa without changing tools. Same word counts = same entropy calculations. Same file format = drop-in replacement. | LOW | Direct 1:1 mapping of list sizes and formats. |
| **Malay-optimized short lists (Alpha & QWERTY)** | Short lists optimized for TV/game-console keyboards. Malay's frequent digraphs (ng, ny, sy, kh) interact differently with these layouts — e.g., on Alpha layout, "t" and "u" are far apart (line 1 → line 4), which affects click count. | MEDIUM | Original English short lists already optimized for layout. Malay translations may produce longer words (e.g., "keanggotaan" = 12 chars vs "membership" = 9 chars) unless word-length constraints are enforced. |
| **Uniquely decodable without delimiters** | "amanatkementerianpemberitahuan" = "a mandate from the ministry announcement" — uniquely decodability removes need for separators. Malay's suffix/prefix density makes this theoretically more challenging but equally valuable. | HIGH | This is the hardest feature. Malay suffixes (-kan, -i, -an, -nya, -ku, -mu) overlap heavily with word beginnings. E.g., "makan" + "kan" = "makankan" (could parse as "makan" + "kan" or "ma" + "kankan" — Schlinkert pruning finds true ambiguities). |
| **Phase approach: smallest lists first** | Build and validate machine on short lists (alpha/qwerty = 1,296 words) before tackling medium (8,192), diceware (7,776), and long (17,576). Catches orthography/profanity/pruning issues early with low cost. | LOW | A project methodology differentiator, not a product one. Still worth documenting. |

## Anti-Features

Features to explicitly NOT build. These would dilute quality or confuse users.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| **Indonesian (KBBI) orthography** | Indonesian and Malaysian spelling differ in systematic ways: loanword treatment (kualiti/kualitas, universiti/universitas, negerionegara), shared words with different meanings (kerana/karena, awallhulu, etc.), and different frequency patterns. Mixing them creates confusing, non-standard lists. | Pure Standard Malay (Malaysia) only. Verify loanwords use -iti/-logi/-si/-(i)si endings per DBP. |
| **Bilingual English/Malay lists** | Users would need to know which language list was used for the passphrase. Also defeats the purpose of a Malay-only list. | Keep English and Malay lists separate. Malay documentation references only Malay lists. |
| **Jawi script version** | Jawi uses Arabic script with different character encoding, right-to-left layout, and vowel representation. Adds massive complexity for no passphrase usability benefit. Password managers expect plain Latin text. | Rumi (Latin) only. |
| **Dialectal or regional Malay** | Kelantanese, Terengganu, Kedahan, Sarawakian, Sabahan dialects have distinct vocabulary that Standard Malay speakers may not recognize (e.g., Kelantan "tah" vs standard "tidak", "gapo" vs standard "apa"). Using them reduces the usable audience. | Only Standard Malay as defined by DBP. The English source words don't map to dialects anyway. |
| **Overly short words (1-2 characters)** | Words like "di" (at), "ke" (to), "ia" (it/she/he), "dan" (and), "ini" (this), "itu" (that) are common but cause: (a) brute force line violations for lists >676 words, (b) prefix decoding ambiguity, (c) trivial guessing. | Minimum 3 characters. Acceptable tradeoff: sacrifice a handful of common 2-letter function words for security of the entire list. |
| **Fused/compound Malay words with hyphens** | Some Malay compound words use hyphens in certain contexts (e.g., "pasca-merdeka", "anti-kemerdekaan"). But modern DBP norm is to write compounds as single words or separate. Hyphens complicate decoding. | All words must be single alphabetic tokens, no hyphens. Compound words like "setiausaha", "tanggungjawab", "warganegara" should be single words. |
| **Scraping Malay text sources directly** | Building a fresh Malay wordlist from Malay Wikipedia or Malay books (instead of translating English) would produce a different set of problems: different curation burden, no alignment with existing Orchard Street quality, unknown uniqueness properties, and no provenance guarantee. | Direct translation from curated English lists is safer and faster. The English lists are already known to be high-quality, common words. |
| **Word frequency annotations** | Some list formats include frequency or ranking metadata. This adds file size and confusion. Password generators only need the word itself (or word + dice roll). | Plain one-word-per-line format. Dice variants use tab-separated rolls. |

## Feature Dependencies

```
English wordlist files (existing)
    → Translate each word to Malay via DeepSeek v4 Pro
        → Malay profanity / dialect / Indonesian reject lists curated
        → Manual verification pass (spelling, commonness, DBP compliance)
            → Normalize (lowercase, trim, UTF-8 NFC)
                → Sardinas–Patterson check
                    → If NOT uniquely decodable: Schlinkert pruning
                        → Re-check unique decodability
                            → Verify entropy / list count still matches target
                                → If count lost: adjust / re-evaluate
                                    → Diceware variant generation (Tidy --dice 6)
                                        → Word-level quality check (min length, no non-alpha)
                                            → Final attribute report (entropy, mean length, edit distance, etc.)
                                                → Malay documentation (README.md, FAQ.md)
```

### Phase Dependency Map

```
Phase 1: README/FAQ translation (no wordlist dependencies)
    → Phase 2: Alpha + Alpha-Dice lists (smallest, validate pipeline)
        → Phase 3: QWERTY + QWERTY-Dice lists (parallel to Alpha structure)
            → Phase 4: Medium list
                → Phase 5: Diceware + Diceware-Clean lists
                    → Phase 6: Long list (largest, greatest pruning risk)
```

Constraint: Each phase produces files that can be independently adopted by password managers. The phases follow increasing word count so that pruning validation gets harder gradually.

## MVP Recommendation

The MVP is not about minimal features — this is a well-scoped translation project with crisp boundaries. Every list is "must have" for completeness. The recommended build order optimizes for risk reduction:

**Prioritize:**

1. **Alpha list (1,296 words)** + dice variant — smallest, fastest validation of the entire translation-to-pruning pipeline. Catches profanity, orthography, and encoding issues cheaply.
2. **QWERTY list (1,296 words)** + dice variant — validates short-list patterns on second layout. If processing pipeline is automated, these two can run in parallel.
3. **Medium list (8,192 words)** — first medium-size test. Validates that Schlinkert pruning scales to moderate list sizes.
4. **Diceware list (7,776 words)** + clean variant — the most common entry point for password manager users (matches EFF long list size). High-value.
5. **Long list (17,576 words)** — most pruning risk. Deferred until pipeline is proven on smaller lists.

**Defer:** Nothing in scope. The project is all 7 wordlists (alpha, alpha-dice, qwerty, qwerty-dice, medium, diceware, diceware-clean, long). No features beyond documentation and lists are planned.

## Sources

- Orchard Street Wordlists `readme.markdown` and `faq.markdown` — feature descriptions, attributes, qualifiers
- Orchard Street CONVENTIONS.md — list format, quality conventions, unique decodability approach
- Orchard Street INTEGRATIONS.md — consumer expectations (KeePassXC, Buttercup, Strongbox, phraze)
- EFF wordlist design principles (Bonneau 2016) — word selection criteria, usability goals
- [Tidy README](https://github.com/sts10/tidy) — Schlinkert pruning, wordlist attributes, Unicode normalization, locale sorting
- [WLA README](https://github.com/sts10/wla) — word list audit attributes, brute force line calculation
- [Wikipedia: Malay orthography](https://en.wikipedia.org/wiki/Malay_orthography) — DBP standard, digraphs, loanword treatment, 1972 spelling reform
- [Malay phonemic orthography](https://en.wikipedia.org/wiki/Malay_orthography#Letter_names_and_pronunciations) — e/e schwa ambiguity, digraph status (ng, ny, sy, kh)
