# Technology Stack

**Analysis Date:** 2026-06-30

## Project Nature

**Type:** Data / documentation project — curated word lists for passphrase generation, plus supporting documentation.

This project contains no executable code, no build system, and no runtime dependencies. It is a pure data repository consisting of eight plain-text word list files and two Markdown documentation files.

## File Formats

| Format | Use | Files |
|--------|-----|-------|
| `.txt` | Plain-text word lists (one word per line; diceware variants use tab-separated `dice-number\tword` format) | `lists/orchard-street-*.txt` (8 files) |
| `.markdown` | Project documentation | `readme.markdown`, `faq.markdown` |
| `.png` | Instructional screenshot | `img/keepassxc-use.png` |
| `.gitignore` | Git ignore rules | `.gitignore` |

## Word List Details

All eight word lists live in the `lists/` directory:

| File | Words | Format | Entropy/Word |
|------|-------|--------|-------------|
| `lists/orchard-street-long.txt` | 17,576 | plain words | 14.101 bits |
| `lists/orchard-street-medium.txt` | 8,192 | plain words | 13.000 bits |
| `lists/orchard-street-diceware.txt` | 7,776 | dice number + tab + word | 12.925 bits |
| `lists/orchard-street-diceware-clean.txt` | 7,776 | plain words (no dice numbers) | 12.925 bits |
| `lists/orchard-street-alpha.txt` | 1,296 | plain words (alphabetical keyboard optimized) | 10.340 bits |
| `lists/orchard-street-qwerty.txt` | 1,296 | plain words (QWERTY keyboard optimized) | 10.340 bits |
| `lists/orchard-street-alpha-dice.txt` | 1,296 | dice number + tab + word | 10.340 bits |
| `lists/orchard-street-qwerty-dice.txt` | 1,296 | dice number + tab + word | 10.340 bits |

## Data Sources

**Primary sources:**
- [Google Books Ngram data](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) (2012 dataset) — source of word frequency data used by Common Word List Maker
- [Wikipedia word frequency project](https://github.com/IlyaSemenov/wikipedia-word-frequency/) by Ilya Semenov — word frequencies pulled from English Wikipedia (data fetched June 2023)

**Selection criteria:**
- Common English words only
- Excludes profane words, abbreviations, and British spellings
- All lists are uniquely decodable (safe to combine words without delimiters)

## Programming Languages

**None detected.** The project is a data repository. There are no source code files in any programming language.

## Build System / Package Management

**None detected.** No build scripts, no `package.json`, `Cargo.toml`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Makefile`, `justfile`, or similar configuration files exist in the repository. (A `.justfile` entry is listed in `.gitignore` but is not committed.)

## CI/CD

**None detected.** No CI pipeline configuration files (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, etc.) are present.

## Dependencies

**Runtime dependencies:** None. The word lists are self-contained plain-text files with no external dependencies.

## Tools Used in Creation (not committed to repo)

These are CLI tools referenced in `faq.markdown` that were used to create or edit the word lists. They are not part of the repository itself:

- **Common Word List Maker** (`https://github.com/sts10/common_word_list_maker`) — scrapes Google Books Ngram data to generate initial candidate word lists
- **Wikipedia word frequency generator** (`https://github.com/IlyaSemenov/wikipedia-word-frequency`) — gathers word frequencies from Wikipedia articles
- **Tidy** (`https://github.com/sts10/tidy`) — CLI utility for editing word lists, used to apply Schlinkert pruning
- **Schlinkert pruning algorithm** — custom process based on the Sardinas–Patterson algorithm that makes word lists uniquely decodable

## Platform Requirements

**Development:** Any system capable of reading plain text files and Markdown.

**Production:** None — the word lists are consumed directly by third-party password managers, CLI tools, and web apps at their own discretion.

## Configuration

**Environment:** Not applicable — there is no runtime environment to configure.

**Project configuration:**
- `.gitignore` — excludes `blended-source.txt`, `words-to-blend-in.txt`, `justfile`, `.justfile`

## License

**CC BY-SA 4.0** — Creative Commons Attribution-ShareAlike 4.0 International License.

---

*Stack analysis: 2026-06-30*
