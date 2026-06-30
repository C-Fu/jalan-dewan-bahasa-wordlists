# Coding Conventions

**Analysis Date:** 2026-06-30

## Naming Patterns

**Files:**
- Pattern: `orchard-street-{type}(-dice)?.txt` — lowercase with hyphens
- Examples: `orchard-street-long.txt`, `orchard-street-diceware.txt`, `orchard-street-alpha-dice.txt`
- Exception: Documentation files use `.markdown` extension (not `.md`), e.g. `readme.markdown`, `faq.markdown`

**Functions/Variables/Types:**
- Not applicable — project contains no code files (no `.ts`, `.js`, `.py`, `.rs`, `.rb`, `.sh`, `.yml`, `.json`, `.toml` files exist)

## Documentation Format

**Markdown flavor:** GitHub-Flavored Markdown (GFM)
- Extension: `.markdown` (not `.md`)
- Files: `readme.markdown`, `faq.markdown`

**Features used:**
- Headers with `#`, `##`, `###`
- Code blocks with triple backticks and language identifiers (`text`, `txt`)
- Inline code spans with single backticks
- Hyperlinks to files within repo: `[Orchard Street Long List](lists/orchard-street-long.txt)`
- Hyperlinks to external URLs: `[the EFF's guide](https://www.eff.org/dice)`
- HTML inline (e.g. license badge `<a rel="license"...>`)
- Images: `![alt text](img/keepassxc-use.png)`
- Ordered/unordered lists
- Superscript: `<sup>13</sup>`

## List File Format

### Plain word lists (non-diceware)

**Format:** Plain text file, one word per line, no trailing whitespace.

```text
abandon
abandoned
abandoning
abandonment
abatement
abbey
abbot
```

**Files using this format:**
- `lists/orchard-street-long.txt` (17,576 words)
- `lists/orchard-street-medium.txt` (8,192 words)
- `lists/orchard-street-alpha.txt` (1,296 words)
- `lists/orchard-street-qwerty.txt` (1,296 words)
- `lists/orchard-street-diceware-clean.txt` (7,776 words)

**Encoding:** Assumed UTF-8 (no BOM), plain ASCII characters.

### Diceware-format lists

**Format:** Plain text file, one word per line. Each line starts with a dice roll number (5 digits for 7,776-word list, 4 digits for 1,296-word lists), followed by a **tab character**, then the word.

```text
11111	abandon
11112	abandoned
11113	abbey
11114	abbot
11115	abdomen
11116	abdominal
11121	abilities
```

**Files using this format:**
- `lists/orchard-street-diceware.txt` (7,776 words, 5-digit dice rolls: 11111–66666)
- `lists/orchard-street-alpha-dice.txt` (1,296 words, 4-digit dice rolls: 1111–6666)
- `lists/orchard-street-qwerty-dice.txt` (1,296 words, 4-digit dice rolls: 1111–6666)

**Dice roll numbering scheme:**
- 5-digit roll (4 lists): each digit 1–6, range `11111`–`66666` = 6^5 = 7,776 unique combinations
- 4-digit roll (2 lists): each digit 1–6, range `1111`–`6666` = 6^4 = 1,296 unique combinations

## Word Quality Conventions

**Case:** All words are **lowercase** only.

**Characters:** Only standard English alphabet letters (a–z). No:
- Punctuation (periods, commas, apostrophes, etc.)
- Hyphens or dashes
- Digits or numbers
- Diacritics or accented characters
- Spaces within words

**Spelling:** American English only. No British spellings (e.g. "colour", "centre", "organise" are excluded).
- See git commit `9fb0d2c` which removes words like "marvellous" per the British English criterion.

**Profanity:** No profane words allowed. Confirmed manually and via community review.

**Abbreviations:** No abbreviations (e.g. "comm" removed in commit `3b58fce`).

**Awkward/niche words:** Removed via manual curation — examples from git history:
- `bdfc367`: removed "birthplace", "please", "racecourse"
- `d57d0b6`: removed "hank"
- `bb0439d`: removed "hepatic" (niche medical term)
- `ab9dfa8`: removed confusing variants of "acknowledge"
- `fad306b`: removed "peter" and "peters"
- `b96bcbd`: removed "aqueous"
- `4fd015f`: removed "porosity"

## Uniquely Decodable Convention

All Orchard Street Wordlists are **uniquely decodable** — meaning no sequence of words from a given list can be parsed in more than one way. This allows passphrases to be concatenated without delimiters (e.g. "thrillerconcernclearedevidencestretchapple").

**Key difference from other wordlists:** Unlike the EFF long list (which achieves unique decodability by banning all prefix words), Orchard Street lists **do** contain prefix words. They achieve unique decodability through a different mechanism.

### Schlinkert Pruning

Schlinkert pruning is the process used to make all Orchard Street wordlists uniquely decodable. It is based on the [Sardinas–Patterson algorithm](https://en.wikipedia.org/wiki/Sardinas%E2%80%93Patterson_algorithm), which determines whether a code (set of codewords/words) is uniquely decodable.

**The process:**
1. Start with a candidate word list (sourced from Wikipedia and Google Books, filtered for quality).
2. Run the Sardinas–Patterson algorithm to detect whether the list is uniquely decodable.
3. If not uniquely decodable, identify the problematic word(s) causing ambiguity.
4. Remove the minimum number of words needed to resolve the ambiguity.
5. Repeat until the list passes the unique decodability check.

**Schlinkert pruning vs. prefix-free (no-prefix) approach:**
- Prefix-free approach: Remove any word that is a prefix of another word in the list. This is simpler but more destructive (removes more words).
- Schlinkert pruning: Removes only the words that actually cause decoding ambiguity, preserving more words in the list.
- Result: Orchard Street lists have `Free of prefix words? : false` but `Uniquely decodable? : true` — meaning they contain prefix words but are still uniquely decodable because the context of the concatenation prevents actual ambiguity.

**Reference:** [Blog post describing Schlinkert pruning in detail](https://sts10.github.io/2022/08/12/efficiently-pruning-until-uniquely-decodable.html)

**Tool used:** [Tidy](https://github.com/sts10/tidy) — a CLI utility for editing word lists, which implements the Schlinkert pruning algorithm.

## Word Sources

**Primary sources:**
1. [Google Books Ngram data](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) (2012 data) — scraped via [Common Word List Maker](https://github.com/sts10/common_word_list_maker)
2. [Wikipedia word frequency project](https://github.com/IlyaSemenov/wikipedia-word-frequency/) (data taken June 2023)

**Processing toolchain:**
1. `common_word_list_maker` — extracts commonly used words from Google Books Ngram data
2. `wikipedia-word-frequency` — gathers word frequencies from Wikipedia articles
3. `Tidy` — prunes lists to be uniquely decodable via Schlinkert pruning

## Licensing

**License:** [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/)

**Reasoning:** Wikipedia text is licensed under CC BY-SA 4.0, and since words were sourced from Wikipedia, the derived wordlists use the same license.

**Attribution:** Sources cited in `readme.markdown` (lines 157–161). Project has no association with Google, Wikipedia, or the Wikipedia frequency project creators.

## Git Configuration

**`.gitignore` contents (at repo root):**
```
blended-source.txt
words-to-blend-in.txt
justfile
.justfile
```

- Ignores build/processing artifacts (`blended-source.txt`, `words-to-blend-in.txt`)
- Ignores task runner config (`justfile`, `.justfile`) for [`just`](https://github.com/casey/just) command runner
- No language-specific ignores (no `node_modules/`, `.venv/`, `target/`, etc.)
- No `.env` or secrets files present in the repository

## Change Management

**Process:** Word list changes are made via manual commits by the maintainer. The git history shows a pattern of:
1. Removing problematic/awkward words (most common)
2. Improving documentation
3. Accepting community contributions via Issues and PRs

**No automated CI/CD pipelines** detected (no `.github/`, `.travis.yml`, or other CI configs).

## No Coding Conventions

This project contains **zero code files**. There are no:
- Source files (`.ts`, `.js`, `.py`, `.rs`, `.rb`, `.go`, etc.)
- Configuration files (`.json`, `.yml`, `.toml`, `.ini`, etc.)
- Build scripts or tooling
- Test files or test frameworks
- Linting or formatting configurations

Therefore, the following convention categories are **not applicable**:
- Naming conventions for functions, variables, or types
- Code style (formatting, linting)
- Import organization
- Error handling patterns
- Logging patterns
- Comment/JSDoc conventions
- Function design (size, parameters, return values)
- Module design (exports, barrel files)
- TypeScript/JavaScript patterns

---

*Convention analysis: 2026-06-30*
