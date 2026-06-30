# Domain Pitfalls: Malay Wordlist Translation for Passphrase Generation

**Domain:** Bahasa Melayu passphrase wordlists (translation of Orchard Street Wordlists)
**Researched:** 2026-06-30
**Overall confidence:** MEDIUM

---

## Critical Pitfalls

Mistakes that would break security, require full rework, or render the wordlists unusable.

### Pitfall 1: Agglutinative Prefix Chaos Destroying Unique Decodability

**What goes wrong:** Malay is an **agglutinative language** with an extremely productive prefix system. After translation, entirely new prefix relationships emerge that did not exist in English, breaking the uniquely-decodable (UD) property that Sardinas–Patterson previously guaranteed.

**Why it happens:** The English lists were pruned for English prefix relationships (e.g., "boy" removed because "boyhood" existed). But a straight translation to Malay introduces a dense web of new prefix collisions from Malay's morphology:

| English Word | Malay Translation | Prefix Problem |
|---|---|---|
| teach | ajar | **Prefix of**: belajar, pelajar, pengajaran, diajar, pembelajaran, mempelajari, memperajari, terajar, diajari |
| eat | makan | **Prefix of**: makanan, pemakan, memakan, dimakan, termakan, memakani, pemakanan |
| book | buku | **Prefix of**: buku-buku, perbukuan, membukukan, kebukuan |
| run | lari | **Prefix of**: berlari, melarikan, pelarian, lari-lari, pelari, berlarian |
| clean | bersih | **Prefix of**: membersihkan, kebersihan, pembersih, pembersihan, dibersihkan |
| day | hari | **Prefix of**: harian, sehari, keseharian, perayaan-harian |
| hand | tangan | **Prefix of**: bertangan, menangani, penanganan, setangan, tangan-tangan |
| say | kata | **Prefix of**: berkata, mengata, mengatakan, perkataan, dikatakan, katakan |
| big | besar | **Prefix of**: membesar, kebesaran, pembesaran, sebesar, besarkan, membesarkan |
| good | baik | **Prefix of**: berbaik, memperbaiki, kebaikan, perbaikan, sebaik, dibaiki |
| water | air | **Prefix of**: air-air, berair, perairan, mengairi, pengairan, keairan |

**Consequences:** A list that was UD in English becomes **non-UD** after translation. Without re-pruning, passphrases become ambiguously parseable, silently reducing effective entropy. A user who generates "makananbesar" might have their passphrase parsed as "makanan+besar" or "makan+an+besar" or "makan+anbe+sar" — an attacker only needs one valid parse.

**Prevention:**
- Run Sardinas–Patterson **from scratch** on every translated list before release
- Do NOT assume the English pruning patterns carry over
- The Schlinkert pruning algorithm in Tidy is language-agnostic (operates on strings), so it should work with Malay — but test `tidy -K` with non-English input explicitly in a trial run with the alpha list first
- After pruning, verify not just UD but also that the remaining word count hits target (1296, 8192, 7776, 17576 words)

**Detection:** Run `tidy -AAAA` on the translated-but-unpruned list. If it reports "Uniquely decodable: false" (it almost certainly will), this pitfall has manifested.

**Phase:** All list translation phases (alpha → medium → diceware → long). Address immediately after each translation batch before marking a list as complete.

**Confidence:** HIGH — confirmed by the structure of Malay grammar (see Wikipedia Malay grammar: affixation is extremely productive with prefixes like meN-, ber-, ter-, per-, di-, ke-, se-).

---

### Pitfall 2: Short Function Words Creating Unfixable Prefix Chains

**What goes wrong:** Malay has an unusually high density of very short (2–3 letter) function words that overlap as prefixes of thousands of common words. These are impossible to remove without destroying usability, yet their presence forces the Schlinkert pruner to remove either the short word or huge cascades of longer words.

**Why it happens:** These short words are both standalone words AND productive affixes:

| Word | Length | Role | Prefix of... |
|------|--------|------|-------------|
| di | 2 | "at/in" + passive prefix | 1000+ words: dimakan, dibaca, ditulis, dibeli, diambil, diberikan... |
| ke | 2 | "to" + prefix | 500+ words: keluar, kekal, kedua, beberapa, kepada, keadaan... |
| ber | 3 | stative/habitual prefix | 500+ words: berjalan, berlari, berbicara, bekerja, berhenti... |
| ter | 3 | accidental prefix | 300+ words: terambil, terjatuh, terbaca, terkenal, terbang... |
| per | 3 | causative prefix | 200+ words: pergi, perkara, perlu, perbaharui, perbaiki... |
| se | 2 | "one/whole" prefix | 300+ words: sebuah, seorang, selamat, sejak, sebelum... |
| pe | 2 | actor prefix | 300+ words: pekerja, pelajar, pemain, penulis, pembeli... |
| me | 2 | agent focus prefix | 500+ words: melihat, membaca, menulis, memasak... |
| men | 3 | variant of me- prefix | 400+ words: menjadi, menanti, mendaki, mencari... |
| meng | 4 | variant of me- prefix | 200+ words: mengambil, mengatakan, mengajar... |
| dan | 3 | "and" | 20+ words: danau, dandan, dana, tandan... |
| ada | 3 | "have/there is" | 10+ words: adakah, adanya, berada, keadaan, mengada... |
| air | 3 | "water" | 10+ words: air-air, berair, perairan, mengairi... |
| api | 3 | "fire" | 5+ words: apian, berapi, mengapi, peapi... |
| ini | 3 | "this" | 0 (unique, safe) |
| itu | 3 | "that" | 5+ words: itulah, situ, itu-itu |

**Consequences:**
- The Schlinkert pruner may need to delete "di" and "ke" from the short lists entirely, losing high-frequency common words
- OR the pruner may cascade-delete hundreds of valid derived words that start with those prefixes
- The short list (1296 words) may fail to reach target word count if too many words are pruned
- The long list (17576 words) is vulnerable: if "me" needs removal, hundreds of "me-" words follow

**Prevention:**
- For short lists (alpha, qwerty, 1296 words): consider proactively excluding all 2-letter words and most 3-letter function words to prevent cascade deletion. Target minimum word length of 4 for short lists.
- For longer lists: allow Tidy's Schlinkert pruning to handle it, but verify the final counts
- Consider maintaining `di` and `ke` variants as wordlist entries only if the Schlinkert pruner can handle them without collateral damage
- Test on alpha list first (smallest, fastest iteration) before scaling to larger lists

**Warning signs during translation:**
- After pruning the alpha list, word count drops below 1296
- The pruner removes >20% of translated words
- Common Malay words like "di", "ke", "dan", "ada" are absent from the final pruned list

**Phase:** Alpha list translation (Phase 3) — this is the canary in the coal mine. If the short list can't reach 1296 words, the workflow needs revision.

**Confidence:** HIGH — confirmed by Malay grammar (Wikipedia: Malay has prefixes meN-, ber-, ter-, per-, di-, ke-, se- which are both bound morphemes and can overlap with word-initial syllables).

---

### Pitfall 3: Reduplication Breaking Unique Decodability

**What goes wrong:** Malay productively forms words via full reduplication (e.g., "buku-buku" = books, "hati-hati" = careful). If both the base form and the reduplicated form appear in the list, the base form is a prefix of the reduplicated form, creating a UD violation.

**Why it happens:** Reduplication is a core Malay word formation process (kata ganda). Common reduplicated words appear naturally in frequency-ranked wordlists:

| Base | Reduplicated | Meaning (Redup) | UD Impact |
|------|-------------|-----------------|-----------|
| buku | buku-buku | books | "buku" is prefix of "buku-buku" |
| hati | hati-hati | careful | "hati" is prefix of "hati-hati" |
| jalan | jalan-jalan | stroll | "jalan" is prefix of "jalan-jalan" |
| mata | mata-mata | spy | "mata" is prefix of "mata-mata" |
| anak | anak-anak | children | "anak" is prefix of "anak-anak" |
| orang | orang-orang | people | "orang" is prefix of "orang-orang" |
| kuda | kuda-kuda | trestle/horse | "kuda" is prefix of "kuda-kuda" |
| kupu | kupu-kupu | butterfly | "kupu" is uncommon alone |
| laba | laba-laba | spider | "laba" means profit |
| rama | rama-rama | butterfly/moth | "rama" uncommon alone |
| sayur | sayur-mayur | vegetables | Rhythmic reduplication variant |

**Consequences:**
- Schlinkert pruning must remove either the base or the reduplicated form
- If "hati" is removed, the list loses a common word
- If "hati-hati" is removed, the list loses a distinct compound meaning
- Reduplicated forms are generally longer than 3 characters, so they're natural fits for passphrase lists

**Prevention:**
- After translation, scan for reduplication patterns (hyphenated duplicates) and decide deliberately per pair
- In general: prefer keeping the **base form** and dropping the reduplicated form, since the reduplicated form is usually just a grammatical variant (most reduplications mark plurality or continuous action, not distinct concepts)
- **Exception**: some reduplicated forms have distinct meanings (kupu-kupu ≠ any meaning of kupu alone; laba-laba ≠ profit; mata-mata ≠ generic "eye"). These are worth keeping if possible
- Run prefix-word check (-P in Tidy) to surface all base-reduplication conflicts

**Warning signs:** Any word with a hyphen in a Malay wordlist is suspect. Any word repeated with "-" between copies.

**Phase:** All list translation phases. Address during pruning by running `tidy -AAAA` and checking the "Free of prefix words?" attribute.

**Confidence:** HIGH — based on Malay grammar (reduplication is a core word formation process described in Wikipedia Malay grammar).

---

### Pitfall 4: Indonesian vs Malaysian Orthography Confusion

**What goes wrong:** The project targets Standard Malay (Malaysia, DBP orthography), but AI translation tools and online dictionaries often default to Indonesian spellings. Mixed orthography produces inconsistency and potentially duplicate words.

**Why it happens:** The languages share ~80% vocabulary but differ in spelling conventions:

| Malaysian (DBP) | Indonesian (EYD) | Notes |
|----------------|-----------------|-------|
| kerana | karena | "because" |
| sistem | sistim (old) / sistem | "system" — now mostly harmonized |
| universiti | universitas | "university" — different root word |
| negeri | negara | "state/country" |
| kereta | mobil | "car" — different word entirely |
| bas | bus | "bus" — spelling difference |
| kualiti | kualitas | "quality" — different suffix (-iti vs -itas) |
| kuantiti | kuantitas | "quantity" — different suffix |
| aktiviti | aktivitas | "activity" — different suffix |
| mega- | mega- | Same |
| pasca- | pasca- | Same |
| Jumaat | Jumat | "Friday" |
| Isnin | Senin | "Monday" |
| Selasa | Selasa | Same |
| Ogos | Agustus | "August" |
| Disember | Desember | "December" |
| November | Nopember | "November" |
| makmal | laboratorium | "laboratory" |
| gereja | gereja | "church" — same spelling, different pronunciation |
| fail | berkas | "file" |
| fokus | fokus | "focus" — same |
| teknikal | teknis | "technical" |

**Consequences:**
- Inconsistent spelling within a list breaks alphabetical sorting (different words for same concept)
- User confusion: Malay speakers may question the authority of a list that uses Indonesian spellings
- License violation risk: the project explicitly scopes out Indonesian orthography

**Prevention:**
- After translation, run a Malaysian/Indonesian spelling consistency check against a reference list
- Key patterns to verify:
  - Words ending in -iti (Malaysian) should NOT end in -itas (Indonesian)
  - Words with "j" should not use Dutch-influenced "dj" (archaic)
  - Words starting with "ka" vs "ke" often differ between standards
- Use DBP's online dictionary (prpm.dbp.gov.my) as the spelling authority for verification
- Check loanword suffixes: Malaysian uses -asi/-si, Indonesian sometimes uses -isasi
- Create a list of known Malaysian/Indonesian divergences and grep the translated wordlists for Indonesian forms

**Warning signs:** Words like "karena", "universitas", "kualitas", "nopember", "agustus" appearing in the list.

**Phase:** Quality assurance after each translation phase, before pruning.

**Confidence:** MEDIUM — the core spelling differences are documented, but a comprehensive divergence list for the full ~17k word vocabulary would need to be compiled.

---

### Pitfall 5: Profanity and Cultural Sensitivity — Malay-Specific Screening Needed

**What goes wrong:** A direct translation of English profanity filters is insufficient. Malay has culturally-specific taboo words, religious sensitivities, and politeness registers that a translated English blocklist won't catch.

**Why it happens:** Profanity is culturally specific. English lists of "bad words" won't contain Malay profanity. Additionally:

1. **Religious sensitivity:** Malay society is predominantly Muslim. Words related to pork (babi), alcohol (arak), apostasy (murtad), or blasphemy (kufur) may be sensitive even if not technically profane
2. **Body parts:** Malay words for genitalia (faraj, zakar, puki, pantat, tetek, konek) must be excluded — same as English, but the specific words differ
3. **Animal insults:** "Anjing" (dog) and "babi" (pig) are common Malay insults
4. **Royalty-related:** Words involving monarchs (raja, sultan, Yang di-Pertuan Agong) aren't profane but may be culturally inappropriate in a passphrase context
5. **Politeness registers:** Malay distinguishes formal from coarse language. Words like "aku" (I, informal) vs "saya" (I, formal) aren't profane, but extremely crude variants like "gua" (Hokkien-derived slang), "koi" (dialect) might be inappropriate for some users
6. **False friends:** "Kotor" = dirty (fine), but "butoh" = performative dance (not a word in standard Malay, but could be confused)
7. **Slang:** Modern Malay slang (e.g., "bapak" for father — neutral, but "bapak kau" becomes insulting) evolves quickly

**Consequences:**
- Including profane Malay words destroys user trust — the English lists specifically advertise being "free of profane words"
- Culturally insensitive words could offend users, making the project unusable for its target audience
- A translated blocklist will miss Malay-specific profanity

**Prevention:**
- Compile a Malay-specific profanity blocklist from:
  - [LDNOOBW list](https://github.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words) (has Malay/Indonesian: `ms` and `id`)
  - Community sources (Malaysian online forums, content moderation lists)
  - Manual review by a Malay speaker
- Screen for: genital terms, sexual acts, excreta, animal insults, religious taboos
- Pay special attention to words that are innocent in English but problematic in Malay (e.g., "puki" — nothing in English, offensive in Malay)
- Run `tidy -r malay_profane_words.txt` on each translated list
- For the "bersih" (clean) variant of the diceware list, apply stricter filtering — remove words that might be merely culturally insensitive rather than technically profane

**Warning signs:** Any word that a Malay speaker would hesitate to say in polite conversation.

**Phase:** Before each list is finalized. The blocklist should be compiled in Phase 1 (research/setup) before translations begin.

**Confidence:** MEDIUM — the strategy is sound, but a comprehensive Malay profanity list needs to be assembled. No authoritative public source is known.

---

### Pitfall 6: Entropy Per Word Drift After Translation

**What goes wrong:** After translation + pruning, the target lists may not achieve the same bits/word entropy as the English originals, silently reducing security.

**Why it happens:**
- **Word length inflation:** Malay agglutinative words are longer on average than English. A short English word like "set" might translate to "set" (same) or "kumpulan" (longer). If the pruner removes more short Malay words (due to prefix conflicts), the average word length increases, but also the total word count may drop
- **Non-standard list sizes:** If the Schlinkert pruner removes too many words, the list won't reach the target size (1296, 8192, 7776, or 17576). Pruning to a non-standard size changes entropy:
  - 1296 → 10.34 bits/word
  - 8192 → 13.00 bits/word
  - 7776 → 12.93 bits/word
  - 17576 → 14.10 bits/word
- **Over-pruning risk:** Aggressive pruning to restore UD could leave the list too small, forcing the use of reject lists or re-translation
- **Minimum word length:** The English lists have shortest words of 3 characters. Malay 2-letter words (di, ke, se) may need removal to achieve UD, potentially raising minimum length to 4+ and changing the "brute force line" calculation

**Entropy targets from English originals:**

| List | Target Size | Bits/Word | Min Word Len | English Shortest |
|------|------------|-----------|-------------|-----------------|
| alpha / qwerty | 1,296 | 10.34 | 3 | "add" |
| medium | 8,192 | 13.00 | 3 | "add" |
| diceware | 7,776 | 12.93 | 3 | "add" |
| long | 17,576 | 14.10 | 3 | "add" |

**Consequences:**
- A smaller-than-target list silently reduces security (user thinks they get 12.9 bits/word but gets only 12.0)
- The project constraint requires entropy to "match or exceed English originals"
- Downstream users (Buttercup, Strongbox) rely on specific list sizes for entropy calculations

**Prevention:**
- After translation + pruning, ALWAYS verify:
  - Word count matches target exactly
  - `tidy -AAA` output shows entropy per word matching or exceeding the target
  - "Above brute force line" reports true
- If word count falls short:
  - Try adjusting the input word order (frequency-sorted input to whittle-to)
  - Consider manual word substitutions (swap problematic words with synonyms)
  - In extreme cases, supplement from a secondary Malay word frequency list
- If the 17576 long list cannot be reached, consider reducing the target (e.g., 7776 or 8192) and documenting the change for downstream consumers
- Run `tidy -m 3` to enforce minimum 3-character words (to match English behavior) but evaluate whether 3 is feasible given pitfall #2

**Phase:** Verification step after each list translation + pruning.

**Confidence:** HIGH — entropy calculation is deterministic. The risk is whether the target count can be achieved, not whether we can measure it.

---

### Pitfall 7: AI Translation Producing Non-Standard or Unnatural Malay

**What goes wrong:** DeepSeek v4 Pro may produce literal, stilted, or incorrect Malay translations that don't match actual Malaysian usage. The wordlists would then contain words that Malaysian speakers don't recognize or use.

**Why it happens:**
- **AI hallucination in low-resource contexts:** While Malay is not low-resource, subtle errors can creep in
- **Singapore/Brunei influence:** AI may mix Malay variants from different regions
- **Archaic/formal register:** AI may default to literary Malay rather than common usage
- **Indonesian bias:** Many online Malay resources tilt Indonesian; AI training data may reflect this
- **False friends:** Words that share form but differ in meaning:

| English | Malay (correct) | Wrong Translation | Issue |
|---------|----------------|-------------------|-------|
| actually | sebenarnya | sebenarnya (correct) | — |
| sensitive | sensitif | *sensitiv (not a word) | Non-existent word |
| appointment | temujanji | *apoinmen (English loanword, non-standard) | Not used in Malay |
| library | perpustakaan | *library (English) | Non-Malay |
| supermarket | pasar raya | *supermarket (English loan, but accepted) | Acceptable but may need decision |
| bicycle | basikal | *basikal (correct) | — |
| hospital | hospital | *hospital (correct) | — |

**Consequences:**
- Lists contain words that native speakers don't recognize, reducing usability
- Credibility of the project is undermined ("why does this list have words no Malaysian uses?")
- May introduce Indonesianisms (see Pitfall #4)

**Prevention:**
- After translation, have each list reviewed by a native Malaysian Malay speaker before finalization
- Cross-reference translations against DBP's Kamus Dewan (prpm.dbp.gov.my) for contentious words
- Check for obvious AI artifacts: overly formal words, rare inflected forms, inconsistent affixation
- Spot-check the medium list (8192 words) — if it passes, the larger lists derived from similar translation patterns should too
- Create a "known translation issues" log to track recurring problems (e.g., "DeepSeek consistently translates 'track' as 'trek' instead of 'landasan'")

**Warning signs:** Words that a Malaysian speaker finds unfamiliar, non-standard English loanwords that have native Malay alternatives.

**Phase:** Quality assurance during/after each translation. The alpha list is the best test case since it's small.

**Confidence:** LOW → MEDIUM (depends on DeepSeek v4 quality for Malay, which has not been verified). Requires empirical testing.

---

### Pitfall 8: Homophone Confusion in Malay Passphrases

**What goes wrong:** Malay has homophones (words with same pronunciation, different spelling) that could confuse users trying to recall passphrases from memory or dictation.

**Why it happens:** Malay orthography's E/e ambiguity creates many homophone pairs:

| Word 1 | Word 2 | Pronunciation | Issue |
|--------|--------|--------------|-------|
| emas (gold) | émas (?) | Both /əmas/ | Same pronunciation, different meaning — but é is not used in modern orthography |
| perang (war) | perang (brown) | Same spelling | Same spelling, different meaning — indistinguishable in writing |
| seri (series/radiance) | seri (draw/tie) | Same spelling | Homograph (different meaning, same spelling) — not a passphrase problem but confusing |
| masa (time) | masa (mass?) | Same | Homograph |
| bulan (moon/month) | bulan (month?) | Same | Homograph — not a problem since meaning is irrelevant to passphrase recall |

Unlike English "sun/son" or "right/write", Malay has **fewer homophones but more homographs** due to the E/e ambiguity. For passphrase *typing*, homographs don't matter (same spelling). For passphrase *recall*, homographs don't matter either (same word to remember).

**The real risk:** The `e` vs `é` distinction in Indonesian orthography creates homophones in Malaysian spelling (which doesn't mark schwa). But since the wordlist only uses written form, this is not a practical concern.

**Malay-specific homophone pairs worth noting:**
- sangka (to suspect) / sangka (same) — homograph only
- sementara (temporary/while) — no common homophone
- akhir (end) vs akir? — non-standard spelling variant

**Consequences:** Minimal. Malay has fewer homophone pairs in standard orthography than English. The Tidy `--homophones` option can be used if pairs are identified, but the risk is low.

**Prevention:**
- Run Tidy's homophone check if a Malay homophone list is available
- Do NOT prioritize this — the English lists didn't filter homophones heavily either (Schlinkert pruning handles some overlap)
- The main English list FAQ explicitly states homophones are not a major concern (FAQ doesn't mention them)

**Phase:** Optional — can be addressed in a quality pass but not blocking.

**Confidence:** HIGH — Malay has fewer homophones than English. Not a priority.

---

### Pitfall 9: "Kata Nafi" and Short Particles Creating Security Overlap

**What goes wrong:** Malay negation and modal particles (tidak, bukan, jangan, belum, sudah, pernah, akan, sedang, telah) are short, common words that may create unpredictable attack surfaces through their frequency and overlap patterns.

**Why it happens:** These function words are:
- Very common in actual usage (high frequency = users may assume they're in the list)
- Short (3-5 characters)
- May serve as prefixes or overlap with longer derived forms

| Particle | Length | Meaning | Possible Conflicts |
|----------|--------|---------|-------------------|
| tidak | 5 | no/not | Prefix of: tidakkah, tidaknya (rare) |
| bukan | 5 | is not | Prefix of: bukankah, bukanlah |
| jangan | 6 | do not | Prefix of: janganlah, jangankan |
| belum | 5 | not yet | Prefix of: belumlah (rare) |
| sudah | 5 | already | Prefix of: sudahkah, sudahnya |
| pernah | 6 | ever | Prefix of: pernahkah |
| akan | 4 | will | Prefix of: akanlah, akankah (rare) |
| sedang | 6 | while/currently | Prefix of: sedangkan |
| telah | 5 | already (past) | Safe (no common derivations) |
| ada | 3 | have/there is | See Pitfall #2 — many overlaps |
| ialah | 5 | is/are | Safe |
| yaitu | 5 | that is | Safe |
| sahaja | 6 | only/just | Safe |

**Consequences:** These words are relatively safe from a UD perspective (most have few or no derived forms). The risk is more about word choice: if the list includes both "tidak" and "tidaklah", the suffix creates a UD issue. But "tidaklah" is not a common word and would likely not appear.

**Prevention:**
- Standard Sardinas–Patterson + Schlinkert pruning handles any suffix overlap naturally
- No special handling needed beyond the normal pruning workflow
- If a negation word like "tidak" is pruned, consider replacing it with a synonym (e.g., bukan, tiada) to maintain vocabulary richness

**Phase:** Handled automatically by pruning. No special phase needed.

**Confidence:** HIGH — standard pruning addresses this. Not a special concern.

---

### Pitfall 10: Word-Length Inversion on Short Lists

**What goes wrong:** The short lists (alpha, qwerty) are optimized for typing on restricted keyboards (smart TV remotes, game controllers). If the Malay translations are significantly longer than the English originals, the typing-optimization benefit is lost.

**Why it happens:** The English short lists average ~4.2 characters/word with max ~7-8 characters. Malay words can be longer due to agglutination:

| English (alpha) | Malay Translation | Length Δ |
|----------------|------------------|----------|
| add | tambah | +2 |
| ago | lalu | 0 |
| aim | sasar | +2 |
| apt | sesuai | +3 |
| arm | lengan | +2 |
| art | seni | 0 |
| ask | tanya | +1 |
| ate | makan | 0 |
| atom | atom | 0 |
| bay | teluk | +2 |
| bet | yakin | +1 |
| bow | busur | +2 |
| bud | tunas | +2 |
| bus | bas | 0 |
| cad | calon | +2 |
| can | boleh | +2 |
| cap | topi | +1 |
| car | kereta | +3 |
| cog | gigi | +1 |
| cop | polis | +2 |

Many common English 3-letter words translate to 4+ character Malay words. The short list may grow beyond the 7-character maximum of the original.

**Consequences:**
- Users entering passphrases on smart TV remotes click more per word
- The "efficiency per character" metric drops
- The alpha/QWERTY layout optimization becomes less effective if words are long enough to need scrolling
- If word count falls below 1296 AND words are longer, the short list is strictly worse than the English original

**Prevention:**
- For short-lists translation: prefer shorter synonyms when available
  - e.g., "makan" (4) instead of "memakan" (7) for "eat"
  - "beli" (4) instead of "membeli" (7) for "buy"
  - "lihat" (5) instead of "melihat" (7) for "see"
- Enforce maximum word length matching English (7 for alpha, 8 for QWERTY)
- If a short English word has no short Malay equivalent, consider substituting a different word with a short Malay translation
- The short list should be the MOST carefully manually curated of all lists — it has the tightest constraints

**Analogy:** The EFF short list prioritizes memorable words over short words. The Orchard Street short lists prioritize short words for typing. Malay's agglutination fights against this goal.

**Phase:** Alpha list translation specifically. If the alpha list succeeds (reaches 1296 words with appropriate lengths), the QWERTY list should follow the same pattern.

**Confidence:** MEDIUM — the word-length inflation is directional (Malay words tend to be longer), but the magnitude depends on which words are selected.

---

## Moderate Pitfalls

### Pitfall 11: Tidy Tool Encoding / Non-ASCII Handling

**What goes wrong:** Tidy's documentation notes it has not been heavily tested with non-English languages. Malay uses only standard Latin ASCII letters (no diacritics), so this is low risk, but edge cases exist.

**Details:** Standard Malay uses only A-Z (no accented characters), so encoding issues are minimal. However:
- Apostrophes in Arabic loanwords (e.g., "Ka'bah", "Al-Qur'an", "ma'mur") may cause sorting or delimiter issues
- Tidy recommends `-z nfc` normalization for non-English lists
- The `--locale ms` or `--locale ms-MY` should be specified for correct Malay alphabetical sorting

**Phase:** Setup/configuration of the translation pipeline.

### Pitfall 12: Missing "Bersih" (Clean) Variant Specifics

**What goes wrong:** The English project has a "clean" variant of the diceware list that omits dice roll numbers. The Malay project must also produce this variant, but may forget.

**Details:** This is a straightforward copy of the word list without the tab-separated dice roll prefix. Low risk of being forgotten if tracked in the requirements.

**Phase:** After diceware list is finalized.

### Pitfall 13: Translation of Abbreviations and Acronyms

**What goes wrong:** The English lists explicitly exclude abbreviations. Translating an abbreviation may produce a full word in Malay, losing the abbreviation filtering.

**Details:** English abbreviations like "app", "apt", "asp" are short words that would be translated differently in Malay. For example:
- "app" → "aplikasi" (not an abbreviation in Malay)
- "apt" → "sesuai" (adjective)
- This is actually BENEFICIAL — the Malay words are more natural

However, some English abbreviations are loanwords in Malay:
- "TV" → "televisyen" (full word)
- "info" → "maklumat" (preferred) or "info" (colloquial)
- Avoid the abbreviated loanword if a full Malay word exists

**Phase:** Translation phase; ensure the system prompt to DeepSeek specifies "translate abbreviations as full words where appropriate."

---

## Minor Pitfalls

### Pitfall 14: Sorting Order in Malay

Malay alphabetical order is identical to English (A-Z, no special characters). The `--locale ms-MY` flag in Tidy is helpful but not critical — the default sort order produces correct results since both use the Latin alphabet without diacritics.

### Pitfall 15: Documentation Translation Consistency

The readme.markdown and FAQ.markdown translations must use consistent Malay terminology for technical terms:
- "passphrase" → "frasa laluan" (or "kata laluan" — decide and stick to one)
- "entropy" → "entropi"
- "uniquely decodable" → "boleh dinyahkod secara unik" (or define once)
- "wordlist" → "senarai perkataan"
- "Schlinkert pruning" → "pemangkasan Schlinkert" (proper name, keep as-is)
- "Sardinas–Patterson algorithm" → "algoritma Sardinas–Patterson"

**Phase:** Documentation translation phase.

### Pitfall 16: Over-Reliance on AI Without Manual Verification

The CONCERNS.md noted the original English lists have single-maintainer risk and no tests. The Malay translation adds AI-generated risks on top. Without ANY manual verification by a Malay speaker, the list quality is entirely dependent on DeepSeek v4's Malay capabilities, which are unverified.

**Phase:** Need a verification step involving a native/fluent Malay speaker before release.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Phase 2: Readme/FAQ translation | Pitfall 15 (doc terminology consistency) | Decide Malay glossary upfront |
| **Phase 3: Alpha list translation** | **Pitfall 2** (short word prefix cascade), **Pitfall 10** (word length inflation) | **Highest risk phase.** Test Schlinkert pruning on alpha first; verify target size of 1296 achievable. The alpha list is the canary. |
| Phase 4: QWERTY list translation | Same as Phase 3 (same size list) | Apply same workflow as alpha |
| Phase 5: Medium list translation | Pitfall 1 (prefix chaos), Pitfall 6 (entropy drift) | Larger list = more prefix collisions but more words to spare |
| Phase 6: Diceware list translation | Pitfall 1, Pitfall 6, Pitfall 12 (bersih variant) | Standard workflow should be established by now |
| Phase 7: Long list translation | Pitfall 1, Pitfall 6 | Largest list, most vulnerable to cascade deletions. Monitor word count closely. |
| All phases | Pitfall 4 (Indo vs Malay spelling) | Run spelling audit per phase |
| All phases | Pitfall 5 (Malay profanity) | Require profanity scan per phase |
| All phases | Pitfall 7 (AI translation quality) | Spot-check translations per phase |

---

## Sources

- Malay orthography — Wikipedia (accessed 2026-06-30) — HIGH confidence
- Malay grammar — Wikipedia (accessed 2026-06-30) — HIGH confidence (affixation, reduplication)
- Tidy documentation and README — github.com/sts10/tidy — HIGH confidence (tool capabilities for non-English)
- Orchard Street Wordlists README, FAQ, CONCERNS.md — local project files — HIGH confidence
- EFF Wordlist blog post (Bonneau, 2016) — eff.org — HIGH confidence (passphrase wordlist design principles)
- Omniglot Malay page — omniglot.com — MEDIUM confidence (pronunciation notes)
- DBP Online Dictionary (Kamus Dewan) — prpm.dbp.gov.my — referenced as authority (not directly accessed)
- LDNOOBW profanity lists — github.com/LDNOOBW (Malay/Indonesian) — MEDIUM confidence (community-maintained)

---

## Research Flags for Validation

1. **DeepSeek v4 Pro Malay quality** — UNKNOWN. The entire project depends on this. Must be validated with a test translation of the alpha list (1296 words) before committing to the full pipeline.

2. **Schlinkert pruning on Malay input** — Tidy's `-K` option claims to work with any language (string-based, no language awareness). This should work, but the Tidy README explicitly says "I've not tested Tidy with other languages all that much." Test with a small Malay sample before running on full lists.

3. **Malay profanity list completeness** — The LDNOOBW list for `ms` (Malay) exists but its comprehensiveness is unverified. Expect to find gaps.

4. **Malay word frequency baseline** — If Schlinkert pruning + translation can't reach target list sizes, a Malay word frequency list (from Wikipedia ms or DBP) may be needed as a supplementation source. Availability of such a list is unknown.

5. **Entropy targets achievable?** — Whether 17576 Malay words can pass UD pruning and still hit the long list target is the biggest open question. The alpha list (1296) is a good proxy: if the short list works, the long list probably will too with aggressive enough word sourcing.
