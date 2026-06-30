# Project Research Summary

**Project:** Jalan Dewan Bahasa Wordlists
**Domain:** Malay (Bahasa Melayu, Malaysia/DBP orthography) passphrase wordlist translation and pruning
**Researched:** 2026-06-30
**Confidence:** MEDIUM

## Executive Summary

Jalan Dewan Bahasa Wordlists is a **direct Malay translation** of the existing Orchard Street Wordlists — 7 English passphrase wordlists totaling ~46,500 words, each uniquely decodable. This is not a greenfield wordlist generation project; it is a **translation pipeline** that transforms each English wordlist into Standard Malay (Malaysia/DBP orthography) while preserving entropy targets, unique decodability, and identical integration with password managers (KeePassXC, Buttercup, Strongbox).

The recommended approach is a **semi-automated pipeline**: translate via DeepSeek v4 Pro → normalize with Tidy → deduplicate → Schlinkert-prune with Tidy (`-K`) → verify with Tidy (`-AAAA`) → format. Tidy (sts10/tidy v0.3.24+) is the non-negotiable core tool — it implements the Sardinas–Patterson algorithm, Schlinkert pruning, Unicode normalization, locale sorting, and dice-roll generation. Building from smallest list to largest (alpha at 1,296 words → long at 17,576 words) provides progressive risk validation.

The **three critical risks** are: (1) Malay's agglutinative morphology (meN-, ber-, ter-, di-, ke-, se- prefixes) introduces dense prefix collisions that didn't exist in English, potentially destroying unique decodability; (2) Malay's short function words ("di", "ke", "me", "se") create cascading prefix conflicts that may force Schlinkert pruning to remove hundreds of valid words, threatening list size targets; (3) DeepSeek v4 Pro's Malay translation quality is unverified — it may produce Indonesian variants, non-standard spellings, or unnatural word choices. All three risks are mitigated by the phased build order (smallest lists first), mandatory `tidy -AAAA` verification after each list, and manual review by a native Malay speaker before release.

## Key Findings

### Recommended Stack

The stack is **minimal** — this is a data transformation project, not a software application. The single essential tool is **Tidy**, which handles normalization, pruning, verification, and formatting. DeepSeek v4 Pro handles translation only. DBP dictionaries serve as validation references.

**Core technologies:**

- **[Tidy](https://github.com/sts10/tidy) v0.3.24+**: Wordlist normalization (`-z nfkd`), Schlinkert pruning (`-K`), unique decodability verification (`-AAAA`), dice-roll prefix generation (`--dice 6`), locale-sensitive sorting (`--locale ms`), whittling (`--whittle-to`). Same tool used to create the original English lists. Tidy's operations are string-based and language-agnostic, confirmed to work with non-English words in official docs.
- **DeepSeek v4 Pro** (as specified in PROJECT.md): English→Malay word translation. Used for translation only — all post-processing is handled by Tidy. Quality for Malay is UNVERIFIED and needs empirical testing.
- **[Kamus Dewan Edisi Keempat](https://huggingface.co/datasets/mesolitica/kamus-dewan)** (HuggingFace): DBP-standard dictionary reference for validating Malay spellings, especially loanwords (universiti vs universitas, kualiti vs kualiti, sistem vs sistim).
- **[PRPM DBP Online](https://prpm.dbp.gov.my/)**: Official DBP portal for spot-checking individual ambiguous translations.

**Critical configuration for Malay:**
```bash
tidy -z nfkd --locale ms -l -K --whittle-to <N> --dice 6
```

**Avoid (overkill):** Malaya NLP library, Malay stemmers (Sastrawi), Wikipedia Word Frequency tool, Jawi script resources, KBBI (Indonesian dictionary).

### Expected Features

**Must have (table stakes):**
- **Unique decodability** via Sardinas–Patterson + Schlinkert pruning — pass `Uniquely decodable? : true`
- **Common, recognizable Malay words** — users must read/spell/recall each word; niche or archaic Malay defeats memorability
- **No profanity** — requires curated Malay profanity reject list (not transliterated from English); religious sensitivity matters
- **Standard Malay (DBP) orthography** — post-1972 Ejaan Rumi Baharu; no Za'aba-era or Indonesian (KBBI) variants
- **Target entropy preserved** — exact word counts: alpha/qwerty=1,296, medium=8,192, diceware=7,776, long=17,576
- **Minimum 3-character words** — Malay 2-letter function words (di, ke, ia) create UD and brute-force-line violations
- **Lowercase only, plain text, UTF-8 no BOM, alphabetical**
- **Diceware format variant** — tab-separated roll codes for physical dice users
- **CC BY-SA 4.0 license** with attribution
- **Malay documentation** — README and FAQ in Malay, not English

**Should have (differentiators):**
- **Schlinkert pruning (not prefix-free)** — Malay's pervasive prefixing morphology makes Schlinkert dramatically more valuable than for English; prefix-only pruning would destroy thousands of valid common words
- **DBP-specified loanword treatment** — Malaysia-specific spelling of loanwords (teknologi, kualiti, universiti, strategi, demokrasi) signals authority and quality
- **Malay-optimized short lists** — Malay's frequent digraphs (ng, ny, sy, kh) interact differently with Alpha/QWERTY keyboard layouts; word-length inflation from agglutination must be manually curbed
- **Smallest-lists-first phase approach** — build alpha (1,296) → qwerty (1,296) → medium (8,192) → diceware (7,776) → long (17,576) to catch issues early with minimal cost

**Defer (out of scope):**
- Bilingual English/Malay lists (defeats purpose of a Malay-only list)
- Jawi script version (massive complexity, no passphrase usability benefit)
- Dialectal or regional Malay (Kelantanese, Terengganu, etc.)
- Word frequency annotations (unnecessary metadata)
- Scraping Malay text sources (direct translation is safer and faster)
- Homophone filtering (Malay has fewer homophones than English; not a priority)

### Architecture Approach

The architecture is a **semi-automated pipeline** extending the existing flat data/documentation model. Each of the 5 English lists flows through the pipeline **independently** — there is no cross-list merging (would break unique decodability). The pipeline is stateless: intermediate scratch files are `.gitignore`-ed, only final committed `.txt` files and documentation are tracked.

**Major components:**
1. **Translation** — Input: English wordlist. Process: Human-directed DeepSeek v4 Pro, one English word → best Standard Malay equivalent. Output: raw Malay wordlist. Rules: prefer shortest common equivalent, prefer single-word translations, reject profane words, keep untranslatable English as last resort.
2. **Normalization** — Tidy `-l -z nfc --locale ms-MY --remove-nonalphabetic`. Lowercases, NFC-normalizes Unicode, strips non-alphabetic characters. Diacritics policy: strip to a-z for max password-manager compatibility.
3. **Deduplication** — Multiple English words → same Malay word (e.g., big/large → besar). Tidy auto-removes; expected loss is minimal (1-5%).
4. **Schlinkert Pruning** — The critical component. Tidy `-K` runs Sardinas–Patterson iteratively to restore unique decodability. Malay's agglutinative prefixes (meN-, ber-, ter-, di-, ke-, se-) and suffixes (-kan, -i, -an, -nya) create entirely new UD-violating relationships not present in English. Expected word loss is LOW confidence — alpha may lose 1-8%, long may lose 5-10%.
5. **Verification** — Tidy `-AAAA` confirms `Uniquely decodable? : true`, word count ≥ target, entropy ≥ English original.
6. **Formatting** — Tidy `--dice 6` generates dice variants for alpha, qwerty, and diceware lists. Output filenames: `jalan-dewan-bahasa-{type}.txt` + `jalan-dewan-bahasa-{type}-dadu.txt`.

**Independent workstream:** Documentation translation (README.md, FAQ.md) has no data dependencies on the wordlist pipeline but must reference final filenames.

### Critical Pitfalls

1. **Agglutinative Prefix Chaos (Critical)** — Malay's productive prefix system (meN-, ber-, ter-, pe-, per-, di-, ke-, se-) creates dense new prefix relationships after translation that didn't exist in English. Every translated list MUST be re-pruned from scratch — the English pruning patterns do not carry over. Prevention: Run `tidy -AAAA` on every translated list before release; never assume English UD carries over.

2. **Short Function Word Prefix Cascade (Critical)** — Malay has high-density 2-3 letter function words (di, ke, se, me, pe, ber, ter) that are simultaneously standalone words AND productive prefixes affecting hundreds of words each. Both "di" and "ke" overlap 500-1000+ words. The Schlinkert pruner may be forced to delete these common words or cascade-delete hundreds of derived words. Prevention: For short lists (alpha, qwerty), proactively exclude 2-letter words and consider minimum 4-char target. Test on alpha first as canary.

3. **Indonesian vs Malaysian Orthography (Critical)** — AI translation tools and online dictionaries default to Indonesian (KBBI) spellings. Critical differences: -iti vs -itas (kualiti/kualitas), negeri vs negara, kerana vs karena, universiti vs universitas, bas vs bus, Jumaat vs Jumat, Ogos vs Agustus. Prevention: After translation, grep for Indonesian forms (-itas, karena, universitas, nopember, agustus). Use prpm.dbp.gov.my as spelling authority for ambiguous words.

4. **Profanity and Cultural Sensitivity (Critical)** — English profanity filters are insufficient for Malay. Malay has culturally-specific taboos: religious sensitivity (babi/pork, arak/alcohol, murtad/apostasy, kufur/blasphemy), animal insults (anjing/dog, babi/pig), royalty-related words, politeness registers (aku vs gua vs saya). Prevention: Compile Malay-specific profanity blocklist from LDNOOBW + community sources + manual review by a Malaysian speaker. Screen each list before finalization.

5. **Entropy Drift After Pruning (Critical)** — Translation + Schlinkert pruning may reduce word counts below target sizes, silently reducing security. Malay agglutination tends to produce longer words, and prefix conflicts remove more words. Prevention: Always verify `tidy -AAAA` shows correct entropy per word, word count matches target exactly, and "Above brute force line" reports true. If short, try Tidy's `-P` (prefix) or `-S` (suffix) alternatives to `-K`. Worst case: supplement from Malay word frequency list.

6. **AI Translation Quality (Moderate)** — DeepSeek v4 Pro's Malay quality is unverified. May produce Indonesian-biased, stilted, archaic, or non-standard Malay. Prevention: Require manual review by a native Malaysian speaker. Cross-reference contentious words against Kamus Dewan. Spot-check medium list (8,192 words) as representative sample.

7. **Reduplication Breaking UD (Moderate)** — Malay productively forms words via full reduplication (buku-buku, hati-hati, jalan-jalan). If both base and reduplicated form exist in a list, the base is a prefix of the reduplicated form → UD violation. Prevention: Scan for hyphenated reduplications after translation. Prefer keeping base form; only keep reduplication when it has a distinct meaning (kupu-kupu ≠ kupu, laba-laba ≠ laba).

8. **Word-Length Inflation on Short Lists (Moderate)** — Malay agglutination makes translated words 1-3 characters longer on average than English originals (add → tambah, car → kereta, apt → sesuai). Short lists (alpha/qwerty) optimized for typing on restricted keyboards lose efficiency. Prevention: For short lists, prefer shorter Malay synonyms; enforce max word length matching English originals (7 for alpha, 8 for qwerty).

## Implications for Roadmap

Based on combined research, the following phase structure is recommended. The ordering is driven by risk management — each phase proves the pipeline on progressively larger lists, with the alpha list as the critical canary.

### Phase 0: Documentation Translation
**Rationale:** No data dependencies — can proceed immediately and in parallel with all other phases. Must be started early because it establishes Malay terminology conventions used throughout the project.
**Delivers:** `README.md` (Malay), `FAQ.md` (Malay)
**Addresses:** FEATURES table stakes — Malay documentation
**Avoids:** PITFALLS #15 (doc terminology consistency) — establish glossary upfront (frasa laluan, entropi, boleh dinyahkod secara unik, senarai perkataan)
**Research flag:** Not needed — straightforward translation with established glossary.

### Phase 1: Alpha List (kecil-alpha, 1,296 words) + dice variant
**Rationale:** Smallest list = fastest iteration. Validates the entire translation-to-pruning pipeline at minimal cost. This phase answers the three biggest unknowns: (1) Can Malay prefixes be pruned without destroying the list? (2) Does Tidy's `-K` work correctly with Malay strings? (3) Is DeepSeek's Malay translation quality acceptable? The alpha list is the "canary in the coal mine" — if it fails, the approach needs revision.
**Delivers:** `jalan-dewan-bahasa-kecil-alpha.txt`, `jalan-dewan-bahasa-kecil-alpha-dadu.txt`
**Addresses:** FEATURES table stakes (UD, entropy, min length, no profanity, DBP orthography) + differentiator (Schlinkert pruning validated for Malay)
**Avoids:** PITFALLS #1 (prefix chaos — caught early on small list), PITFALLS #2 (short word cascade — alpha is the test case), PITFALLS #6 (entropy drift — verified on smallest scale), PITFALLS #10 (word length inflation — alpha has tightest constraints)
**Research flags:** **HIGH NEED for research-phase.** Must test DeepSeek v4 Pro Malay quality with a ~50-word sample. Must test Tidy `-K` with Malay input on a small sample. Must compile initial Malay profanity blocklist. If alpha can't reach 1,296 words after pruning, develop fallback strategy (synonym substitution, frequency-list supplementation).

### Phase 2: QWERTY List (kecil-qwerty, 1,296 words) + dice variant
**Rationale:** Same size as alpha with different vocabulary. Once the pipeline is proven on alpha, qwerty follows the exact same workflow. Reuses all Phase 1 learnings.
**Delivers:** `jalan-dewan-bahasa-kecil-qwerty.txt`, `jalan-dewan-bahasa-kecil-qwerty-dadu.txt`
**Addresses:** Same table stakes and differentiators as Phase 1
**Avoids:** Same pitfalls as Phase 1 (now with proven mitigations)
**Research flag:** Low — standard patterns from Phase 1 carry over.

### Phase 3: Medium List (sederhana, 8,192 words)
**Rationale:** First medium-size test (6× larger than alpha). Validates that Schlinkert pruning scales to moderate list sizes and that the word-length fallback strategy (3-char minimum vs 2-char exclusion) holds at scale.
**Delivers:** `jalan-dewan-bahasa-sederhana.txt`
**Addresses:** FEATURES table stakes at medium scale + differentiator (DBP loanword treatment across larger vocabulary)
**Avoids:** PITFALLS #4 (Indonesian vs Malaysian — more loanwords appear at this scale), PITFALLS #6 (entropy drift — verify medium-size targets), PITFALLS #7 (AI quality — spot-check 8k translations)
**Research flag:** Medium — may need to research Malay synonym lists for replacement strategy if too many collisions.

### Phase 4: Diceware + Clean Lists (dadu, 7,776 words)
**Rationale:** The most common list for password manager users (matches EFF long list size). High value. Pipeline should be well-established by this phase.
**Delivers:** `jalan-dewan-bahasa-dadu.txt`, `jalan-dewan-bahasa-dadu-bersih.txt`
**Addresses:** FEATURES table stakes for diceware + differentiator (Malay-optimized clean variant)
**Avoids:** PITFALLS #12 (clean variant — easily forgotten, track explicitly)
**Research flag:** Low — pipeline and patterns established by Phase 3.

### Phase 5: Long List (besar, 17,576 words)
**Rationale:** Largest list, most vulnerable to cascade deletions from agglutinative prefix conflicts. Deferred until pipeline is proven on smaller lists. This is the highest-risk phase for entropy drift (Pitfall #6).
**Delivers:** `jalan-dewan-bahasa-besar.txt`
**Addresses:** FEATURES table stakes at full scale
**Avoids:** All critical pitfalls at maximum scale. Especially PITFALLS #1 (prefix chaos — largest surface area), PITFALLS #2 (short word cascade — most vulnerable), PITFALLS #6 (entropy drift — hardest to hit 17,576 target)
**Research flags:** **HIGH need for research-phase.** If 17,576 target can't be met, need fallback plan: (a) supplement from Malay frequency list (Wikipedia ms unigrams), (b) reduce target and document for downstream consumers, (c) use Tidy `-P` or `-S` as alternative to `-K`.

### Phase 6: Final Cross-Verification
**Rationale:** After all lists are individually verified, do a final holistic check: re-verify unique decodability of all 7 files, confirm entropy targets, update README with final filenames, clean up `.gitignore`.
**Delivers:** Verified release-ready repository
**Research flag:** Not needed — pure verification.

### Phase Ordering Rationale

- **Smallest-first de-risks the pipeline:** The alpha list (1,296 words) is where the three biggest unknowns get resolved. If Malay agglutination makes pruning impossible, or DeepSeek quality is unacceptable, or 1,296 words can't be reached — this is discovered in hours, not weeks.
- **Independent list processing:** Each list translates and prunes independently (no cross-list merging), so phases are naturally parallelizable — but doing them sequentially minimizes rework when process refinements from earlier phases apply to later, larger lists.
- **Documentation is independent:** Phase 0 runs alongside all others. Its only dependency is knowing final filenames (update at Phase 6).
- **Risk escalates with list size:** The long list (17,576) has 13.6× more words than alpha. Each prefix collision has a larger cascade surface. Deferring it to Phase 5 means 4 phases of pipeline hardening before tackling the hardest problem.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 1 (Alpha):** HIGH — DeepSeek v4 Pro Malay quality is UNKNOWN; Tidy `-K` on non-English input is UNVERIFIED (despite docs claiming it works); Malay profanity list needs assembly; short word prefix cascade is untested on real Malay data
- **Phase 5 (Long):** HIGH — whether 17,576 Malay words can pass UD pruning and hit target is the biggest open question. Alpha list success is a proxy but not definitive.

Phases with standard patterns (skip research-phase):
- **Phase 0 (Documentation):** Straightforward translation task with established glossary
- **Phase 2 (QWERTY):** Same size as alpha, reuses same pipeline
- **Phase 4 (Diceware):** Pipeline well-established by Phase 3
- **Phase 6 (Verification):** Pure verification, no new research needed

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | **HIGH** | Tidy is well-documented, language-agnostic, and confirmed by official README for non-English use. No alternatives come close. DeepSeek v4 Pro is project-specified. |
| Features | **HIGH** | Direct translation of well-specified English feature set. Every feature has clear "why" and "success criterion." Malay-specific additions (profanity, DBP compliance) are well-scoped. |
| Architecture | **MEDIUM-HIGH** | Pipeline model is well-understood and documented. Key uncertainty: whether Tidy `-K` behaves identically for Malay strings as English. The Tidy README caveat ("not tested much with other languages") gives pause. Also: deduplication loss estimates are guesses (5 of 22 English synonym groups → same Malay is possible). |
| Pitfalls | **MEDIUM** | High confidence in identifying the pitfalls (Malay grammar is well-documented). Medium confidence in mitigation completeness. Specific concerns: (1) DeepSeek v4 Pro's Malay quality is unmeasured — the entire project quality depends on this. (2) Malay profanity list completeness is unverified. (3) The word-length-inflation magnitude depends on translation choices and is hard to predict. |

**Overall confidence:** MEDIUM

### Gaps to Address

- **DeepSeek v4 Pro Malay quality (HIGH PRIORITY):** The entire project's word quality depends on DeepSeek's ability to produce natural, DBP-standard Malay translations. This must be empirically tested with a ~50-word sample from the alpha list during Phase 1 planning before committing to the full pipeline. If quality is poor, consider: (a) improved system prompts (explicit DBP instructions, Indonesian avoidance), (b) human post-editing budget, (c) alternative translation engine.
- **Malay profanity blocklist (HIGH PRIORITY):** No authoritative public source for Malay profanity known. Strategy: compile from LDNOOBW `ms` list, Malaysian community sources, and manual review. Needs assembly during Phase 1 preparation.
- **Word count targets after pruning (MEDIUM PRIORITY):** The expected post-prune sizes for each list are LOW confidence guesses. The alpha list (1,296 words) is the first real data point. If alpha cannot reach 1,296, a fallback plan for all larger lists is needed before proceeding to Phase 3.
- **Tidy locale support for `ms`/`ms-MY` on target CI systems (LOW PRIORITY):** Tidy delegates to Rust's locale system. Must verify `--list-locales` includes `ms` or `ms-MY` on the actual build/CI system. If not, default sort (identical for a-z) is acceptable but suboptimal.

## Sources

### Primary (HIGH confidence)
- [sts10/tidy README](https://github.com/sts10/tidy) — Non-English word support, Schlinkert pruning, normalization, dice generation
- [sts10/tidy releases](https://github.com/sts10/tidy/releases) — v0.3.24 (Feb 2026), pre-built binaries
- [Schlinkert Pruning Blog Post](https://sts10.github.io/2022/08/12/efficiently-pruning-until-uniquely-decodable.html) — Algorithm documentation (sts10)
- [Orchard Street Wordlists](.) — Existing readme.markdown, faq.markdown, CONVENTIONS.md, INTEGRATIONS.md, CONCERNS.md — local project files
- Wikipedia: [Malay orthography](https://en.wikipedia.org/wiki/Malay_orthography), [Malay grammar](https://en.wikipedia.org/wiki/Malay_grammar) — Affixation, reduplication, DBP standard

### Secondary (MEDIUM confidence)
- [Kamus Dewan Edisi Keempat](https://huggingface.co/datasets/mesolitica/kamus-dewan) — DBP dictionary dataset (HuggingFace)
- [DBP Crawl Dataset](https://huggingface.co/datasets/mesolitica/dbp) — 78,670 Malay words from prpm.dbp.gov.my
- [Malaysian Dataset](https://github.com/malaysia-ai/malaysian-dataset) — Malays.dic.txt (24,550 words), DBP crawl, Ngram data
- [PRPM DBP Online](https://prpm.dbp.gov.my/) — Official DBP portal for orthography validation
- [LDNOOBW profanity lists](https://github.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words) — Malay (`ms`) and Indonesian (`id`) profanity lists (community-maintained)
- [WLA README](https://github.com/sts10/wla) — Word list auditor attributes, brute force line calculation

### Tertiary (LOW confidence — needs validation)
- DeepSeek v4 Pro Malay translation quality — **UNVERIFIED**, must be tested empirically
- Malay word frequency list for potential supplementation — existence and quality unknown
- Comprehensive list of Malaysian/Indonesian spelling divergences for full ~17k vocabulary — partial list in research, not exhaustive
- Tidy `-K` behavior with non-English input — documented as untested by maintainer

---
*Research completed: 2026-06-30*
*Ready for roadmap: yes*
