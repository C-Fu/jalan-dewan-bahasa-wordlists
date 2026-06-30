# Testing Patterns

**Analysis Date:** 2026-06-30

## Test Framework

**No automated test framework detected.** The project contains:
- No test runner configuration (no `jest.config.*`, `vitest.config.*`, `pytest.ini`, etc.)
- No test files (no `*.test.*`, `*.spec.*`, `test_*.py`, etc.)
- No CI/CD pipeline (no `.github/`, `.travis.yml`, etc.)
- No build scripts, makefiles, or task runner configs
- No code files of any kind

**This is a wordlist data project**, not a software project. Quality is verified through manual processes and tooling run by the maintainer, not through automated test suites.

## Quality Properties

Each wordlist is verified against a set of quality properties. These properties are documented per list in `readme.markdown` and verified during the list creation process using external tools.

### Per-List Quality Metrics

The following properties are documented for every list in `readme.markdown`:

| Property | Description | Verified For |
|----------|-------------|-------------|
| List length | Word count | All 8 lists |
| Mean word length | Average characters per word | All 8 lists |
| Length of shortest word | Minimum word length in characters | All 8 lists |
| Length of longest word | Maximum word length in characters | All 8 lists |
| Free of prefix words? | Whether any word is a prefix of another | All 8 lists (all return `false`) |
| Uniquely decodable? | Whether list passes Sardinas–Patterson check | All 8 lists (all return `true`) |
| Entropy per word | log₂(list length) in bits | All 8 lists |
| Efficiency per character | Entropy ÷ mean word length | All 8 lists |
| Above brute force line? | Whether list exceeds brute-force threshold | All 8 lists (all return `true`) |
| Mean edit distance | Average Levenshtein distance between word pairs | All 8 lists |

### Quality Metrics by List

| List | Words | Bits/word | Mean length | Mean edit dist |
|------|-------|-----------|-------------|----------------|
| `orchard-street-long.txt` | 17,576 | 14.101 | 7.98 chars | 7.915 |
| `orchard-street-medium.txt` | 8,192 | 13.000 | 7.07 chars | 6.966 |
| `orchard-street-diceware.txt` | 7,776 | 12.925 | 7.05 chars | 6.954 |
| `orchard-street-diceware-clean.txt` | 7,776 | 12.925 | 7.05 chars | 6.954 |
| `orchard-street-alpha.txt` | 1,296 | 10.340 | 4.12 chars | 4.043 |
| `orchard-street-alpha-dice.txt` | 1,296 | 10.340 | 4.12 chars | 4.043 |
| `orchard-street-qwerty.txt` | 1,296 | 10.340 | 4.24 chars | 4.170 |
| `orchard-street-qwerty-dice.txt` | 1,296 | 10.340 | 4.24 chars | 4.170 |

## Quality Verification Methods

### 1. Uniquely Decodable Verification

**Method:** [Sardinas–Patterson algorithm](https://en.wikipedia.org/wiki/Sardinas%E2%80%93Patterson_algorithm)

**Tool:** [Tidy](https://github.com/sts10/tidy), specifically its Schlinkert pruning mode.

**Process:**
1. Run Sardinas–Patterson on the candidate wordlist.
2. If the algorithm detects ambiguous decodings, identify the problematic word(s).
3. Remove the minimum number of words necessary to eliminate ambiguity.
4. Re-run the algorithm until the list passes.
5. All 8 lists currently pass as uniquely decodable.

**Result:** All lists show `Free of prefix words? : false` but `Uniquely decodable? : true`. This is by design — Schlinkert pruning preserves prefix words that don't cause actual ambiguity.

### 2. Profanity Check

**Method:** Manual review plus community reporting.

**Evidence:**
- The `readme.markdown` states "Free of profane words" as a key property.
- Community members can report profane words via Issues. No open profanity issues exist.
- Word sources (Wikipedia articles, Google Books) naturally exclude most profanity.

### 3. Abbreviation and British Spelling Check

**Method:** Manual curation and community review.

**Evidence from git history:**
- `3b58fce`: removed abbreviation "comm" from lists
- Git log pattern: words are periodically reviewed and swapped/removed when identified as inappropriate

### 4. Brute Force Line Check

**Method:** The "Above brute force line?" metric checks whether the list's average efficiency per character exceeds a known threshold for brute-force resistance. All 8 lists pass this check.

### 5. Mean Edit Distance

**Method:** Calculates the average [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) between all pairs of words in each list. Higher values indicate that words are more visually distinct from each other, reducing the chance of misreading or mistyping.

### 6. Prefix Word Check

**Method:** Checks whether any word in a list is a prefix of another word in the same list. All lists contain prefix words (returns `false` for "Free of prefix words?"), but are still uniquely decodable due to Schlinkert pruning.

## Word Source Verification

**Sources are documented in `readme.markdown`** (lines 157–161):
- [Google Books Ngram data](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) (2012 data) — acquired via [Common Word List Maker](https://github.com/sts10/common_word_list_maker)
- [Wikipedia word frequency project](https://github.com/IlyaSemenov/wikipedia-word-frequency/) by Ilya Semenov — data taken June 2023

**License compliance:** Wikipedia text is licensed under CC BY-SA 4.0, so the wordlists are also licensed CC BY-SA 4.0, with attribution provided.

## Community Validation

**No formal testing process** — quality is maintained through:

1. **Issue tracking:** Users can report problems via GitHub Issues. Issues have historically led to word removals (e.g., profanity, awkward words, British spellings).

2. **Pull requests:** Community members can submit PRs to improve the lists or documentation.

3. **Transparent git history:** Every word addition, removal, or swap is recorded in the commit history with explanatory messages, e.g.:
   - `bdfc367 swap out a few words: birthplace, please, racecourse`
   - `d57d0b6 remove awkward word 'hank' from 3 lists`
   - `fad306b remove awkward words 'peter' and 'peters' from all lists`

## Production Validation

The wordlists are validated through real-world usage:

- **Buttercup password manager** uses `orchard-street-medium.txt` as its passphrase word list ([PR #18](https://github.com/buttercup/buttercup-generator/pull/18))
- **Strongbox password manager** offers `orchard-street-diceware.txt` to users ([Strongbox source](https://github.com/strongbox-password-safe/Strongbox/blob/master/resources/wordlists/orchard-street-diceware.txt))
- **Phraze CLI** ([sts10/phraze](https://github.com/sts10/phraze)) generates passphrases from Orchard Street lists
- **StrongPhrase.net** ([online generator](https://strongphrase.net/#/more)) uses Long and QWERTY lists

## Quality Checklist for New Lists or Modifications

When adding a new wordlist or modifying an existing one, the maintainer should verify:

- [ ] List is uniquely decodable (run Sardinas–Patterson / Tidy)
- [ ] No profane words present (manual review)
- [ ] No abbreviations present
- [ ] No British spellings present (American English only)
- [ ] All words are lowercase, a–z only, no punctuation or hyphens
- [ ] Entropy per word calculated and documented (log₂ of list length)
- [ ] Mean word length calculated and documented
- [ ] Mean edit distance calculated and documented
- [ ] List passes brute force line check
- [ ] Word sources are documented
- [ ] License is CC BY-SA 4.0 with attribution
- [ ] If diceware format: dice roll numbers are sequential and correct for the list length

## Coverage

**Requirements:** None enforced (no test coverage target).

**Actual coverage:** Quality properties are verified for every list at creation/modification time, but there are no automated regression tests to catch regressions after edits.

## Test Types

**Unit Tests:** Not applicable — no code in project.

**Integration Tests:** Not applicable — no code in project.

**E2E Tests:** Not applicable — no code in project.

**Manual smoke tests:** Each list's quality metrics (entropy, mean length, unique decodability) are recomputed by the maintainer whenever the list is modified.

---

*Testing analysis: 2026-06-30*
