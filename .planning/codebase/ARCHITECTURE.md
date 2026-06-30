<!-- refreshed: 2026-06-30 -->
# Architecture

**Analysis Date:** 2026-06-30

## System Overview

This is a **data/documentation project** — not a software project. There is no application code, no build system, no runtime, and no automated tests. The entire repo is a flat collection of curated word-list text files and supporting documentation (markdown + images).

```text
┌───────────────────────────────────────────────────────────┐
│                     Word List Taxonomy                      │
│                                                             │
│  ┌─ Long (17576 words, ~14.1 bits/word)                   │
│  │   └─ orchard-street-long.txt                            │
│  ├─ Medium (8192 words, 13 bits/word)                      │
│  │   └─ orchard-street-medium.txt                          │
│  ├─ Diceware (7776 words, ~12.9 bits/word)                 │
│  │   ├─ orchard-street-diceware.txt   (with dice prefixes)  │
│  │   └─ orchard-street-diceware-clean.txt  (no prefixes)    │
│  └─ Short (1296 words, ~10.3 bits/word)                   │
│      ├─ Alpha: α.txt / α-dice.txt                          │
│      └─ QWERTY: qwerty.txt / qwerty-dice.txt               │
└───────────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────────┐
│                  Documentation Layer                        │
│  readme.markdown   faq.markdown   img/keepassxc-use.png    │
└───────────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────────┐
│                  Version Control (.git/)                    │
│  Tracks all changes to lists and docs, provides releases   │
└───────────────────────────────────────────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| Long List | Highest-entropy / longest words for strong passphrases | `lists/orchard-street-long.txt` |
| Medium List | 2<sup>13</sup> words, binary-optimized entropy calculation | `lists/orchard-street-medium.txt` |
| Diceware List | 5-dice-roll compatible wordlist with prefix codes | `lists/orchard-street-diceware.txt` |
| Diceware Clean | Same words as diceware, no dice-roll prefixes | `lists/orchard-street-diceware-clean.txt` |
| Alpha Short List | Optimized for alphabetical on-screen keyboards | `lists/orchard-street-alpha.txt` |
| Alpha Dice | Alpha list with 4-dice-roll prefixes | `lists/orchard-street-alpha-dice.txt` |
| QWERTY Short List | Optimized for QWERTY on-screen keyboards | `lists/orchard-street-qwerty.txt` |
| QWERTY Dice | QWERTY list with 4-dice-roll prefixes | `lists/orchard-street-qwerty-dice.txt` |
| Readme | Project overview, list descriptions, licensing | `readme.markdown` |
| FAQ | Frequently asked questions | `faq.markdown` |
| KeePassXC screenshot | Guide image for wordlist setup | `img/keepassxc-use.png` |

## Pattern Overview

**Overall:** Flat data/documentation repository.

**Key Characteristics:**
- All word lists are plain `.txt` files with one word per line
- No code, no build step, no package manager
- Every list is **uniquely decodable** (verified via Sardinas–Patterson algorithm / Schlinkert pruning)
- Lists exist in pairs: with and without dice-roll prefixes
- Documentation is markdown-only (no site generator, no HTML)
- Version control (.git) is the sole development tool

## Layers

**Word List Data Layer:**
- Purpose: Contains the curated passphrase word lists
- Location: `lists/`
- Contains: 8 `.txt` files
- Depends on: Nothing
- Used by: Password managers (KeePassXC, Strongbox, Buttercup), CLI tools (phraze, passphraseme), and human readers

**Documentation Layer:**
- Purpose: Explains the word lists, usage, and creation process
- Location: repo root + `img/`
- Contains: `readme.markdown`, `faq.markdown`, `img/keepassxc-use.png`
- Depends on: Nothing
- Used by: Human consumers of the project

## Data Flow

### Primary Request Path (Human reads a word list)

1. User opens file from `lists/` in text editor or password manager
2. File is read line-by-line — each line is either a bare word or a `dice-roll-prefix\tword` pair
3. User selects words randomly from the list to construct a passphrase

### Secondary Path (Password Manager Integration)

1. Password manager (KeePassXC, Strongbox, Buttercup) loads `.txt` file as a custom word list
2. Application parses the file to build an in-memory word array
3. Passphrase generator draws words uniformly at random from the array

**State Management:** Stateless — the lists are static text files.

## Key Abstractions

**Word List:**
- Purpose: A curated collection of English words suitable for passphrase generation
- Examples: `lists/orchard-street-long.txt`, `lists/orchard-street-medium.txt`
- Pattern: Plain text file, one word per line, sorted alphabetically within each file
- Properties: Uniquely decodable, free of profanity, no abbreviations, no British spellings

**Dice Variant:**
- Purpose: A word list with tab-separated dice-roll codes prepended to each word
- Examples: `lists/orchard-street-diceware.txt`, `lists/orchard-street-alpha-dice.txt`, `lists/orchard-street-qwerty-dice.txt`
- Pattern: `dice-code\tword` where dice-code is a number (5 digits for 7776-word lists, 4 digits for 1296-word lists)
- Use: Enables passphrase generation using physical dice

**Unique Decodability:**
- Purpose: Ensures that concatenated words from a list can be uniquely segmented back into the original words, allowing passphrases to omit separators
- Implementation: Achieved via Schlinkert pruning (an iterative process based on the Sardinas–Patterson algorithm)
- Impact: Passphrases like "thrillerconcernclearedevidencestretchapple" are safe to use without delimiters

## Entry Points

**Repository Root:**
- Location: `readme.markdown`
- Triggers: Human reads the GitHub repository page
- Responsibilities: Introduces the project, summarizes each list, links to FAQ, states the license

**Word Lists:**
- Location: `lists/orchard-street-*.txt`
- Triggers: Password manager imports file; CLI tool reads file; human downloads for offline use
- Responsibilities: Provide words for passphrase generation

## Architectural Constraints

- **No code:** This project intentionally contains no source code, automation, or tests. All creation tooling lives in external repositories (common_word_list_maker, tidy, wikipedia-word-frequency).
- **Static data only:** Word lists are never programmatically generated or transformed at build time. They are committed as-is.
- **Unique decodability:** Every list must pass the Sardinas–Patterson unique-decodability check. This imposes constraints on which words can coexist in a single list.
- **No list merging:** Lists must not be combined, as the union would almost certainly lose unique decodability.
- **Line format:** Dice variants use a single tab character between the dice code and the word. Non-dice variants contain only the word, with no separator characters.

## Anti-Patterns

### Combining lists

**What happens:** A consumer concatenates two or more Orchard Street word lists into one large list.
**Why it's wrong:** The combined list is almost certainly not uniquely decodable, breaking the core property that makes passphrase separators optional.
**Do this instead:** Use a single list as-is, or use an external tool like [Tidy](https://github.com/sts10/tidy) to re-verify and re-establish unique decodability after any edits.

### Forgetting which variant to use

**What happens:** A user copies `orchard-street-diceware.txt` (with prefixes) into a password manager that expects bare words.
**Why it's wrong:** The password manager will treat the dice-roll codes as part of the word (e.g., "11111\tabandon"), generating passphrases with embedded numbers and tabs.
**Do this instead:** Use the `-clean` suffix variant (`orchard-street-diceware-clean.txt`) or one of the non-dice short lists when the tool does not understand prefix notation.

## Error Handling

**Strategy:** Not applicable — there is no code to produce errors.

## Cross-Cutting Concerns

**Licensing:** All files are licensed under CC BY-SA 4.0. Word data derives from Google Books Ngram (2012) and Wikipedia (June 2023 dump via IlyaSemenov/wikipedia-word-frequency).

**Versioning:** GitHub releases/tags provide snapshots. The repo note in `readme.markdown` advises users to download a tagged release if they want a static copy.

---

*Architecture analysis: 2026-06-30*
