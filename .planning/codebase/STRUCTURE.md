# Codebase Structure

**Analysis Date:** 2026-06-30

## Directory Layout

```
jalan-dewan-bahasa-wordlists/
├── lists/                          # 8 curated word list files
│   ├── orchard-street-long.txt     # 17576 words, ~14.1 bits/word
│   ├── orchard-street-medium.txt   # 8192 words, 13 bits/word
│   ├── orchard-street-diceware.txt # 7776 words, with 5-dice prefixes
│   ├── orchard-street-diceware-clean.txt  # 7776 words, no prefixes
│   ├── orchard-street-alpha.txt    # 1296 words, alphabetical layout
│   ├── orchard-street-alpha-dice.txt     # 1296 words, 4-dice prefixes
│   ├── orchard-street-qwerty.txt   # 1296 words, QWERTY layout
│   └── orchard-street-qwerty-dice.txt    # 1296 words, 4-dice prefixes
├── img/
│   └── keepassxc-use.png           # Screenshot: KeePassXC wordlist setup
├── .planning/                      # Development planning artifacts
│   └── codebase/                   # Codebase mapping documents
├── .git/                           # Version control (VCS)
├── readme.markdown                 # Project overview and list descriptions
├── faq.markdown                    # Frequently asked questions
└── .gitignore                      # Git ignore rules
```

## Directory Purposes

**`lists/`:**
- Purpose: All curated passphrase word list files
- Contains: 8 `.txt` files, one word per line
- Key files:
  - `orchard-street-long.txt`: The largest list — 17576 words, ~14.1 bits/word, mean word length 7.98 chars
  - `orchard-street-medium.txt`: 8192 words (2<sup>13</sup>), exactly 13 bits/word, used by Buttercup password manager
  - `orchard-street-diceware.txt`: 7776 words (for 5 dice rolls), with tab-separated dice-roll prefix codes (e.g., "11111\tabandon")
  - `orchard-street-diceware-clean.txt`: Same 7776 words as the diceware list, but without dice-roll prefixes — for tools that parse bare words
  - `orchard-street-alpha.txt`: 1296 words optimized for alphabetical on-screen keyboards; 4.12 mean character length
  - `orchard-street-alpha-dice.txt`: Alpha list with 4-dice-roll prefix codes (e.g., "1111\tabbot")
  - `orchard-street-qwerty.txt`: 1296 words optimized for QWERTY on-screen keyboards; 4.24 mean character length
  - `orchard-street-qwerty-dice.txt`: QWERTY list with 4-dice-roll prefix codes (e.g., "1111\taccess")

**`img/`:**
- Purpose: Screenshots and images for documentation
- Contains: 1 PNG screenshot
- Key files: `img/keepassxc-use.png` — demonstrates how to load a custom word list in KeePassXC

**`.planning/`:**
- Purpose: Development planning artifacts and codebase maps
- Contains: Codebase analysis documents
- Key files: `.planning/codebase/` — architecture and structure documents consumed by `/gsd-plan-phase` and `/gsd-execute-phase`

## Key File Locations

**Entry Points:**
- `readme.markdown`: Primary entry point for human visitors (GitHub landing page)

**Documentation:**
- `readme.markdown`: Project overview, word list table, usage guidance, licensing
- `faq.markdown`: Answers to common questions (dice usage, password manager integration, unique decodability, creation process)

**Core Data:**
- `lists/orchard-street-long.txt`: Highest-entropy word list
- `lists/orchard-street-medium.txt`: Binary-optimized word list
- `lists/orchard-street-diceware.txt`: Dice-compatible list (primary)
- `lists/orchard-street-diceware-clean.txt`: Dice-compatible list (no prefixes)
- `lists/orchard-street-alpha.txt`: Short list for alphabetical keyboards
- `lists/orchard-street-alpha-dice.txt`: Short list with dice prefixes
- `lists/orchard-street-qwerty.txt`: Short list for QWERTY keyboards
- `lists/orchard-street-qwerty-dice.txt`: Short list with dice prefixes

**Configuration:**
- `.gitignore`: Ignores `blended-source.txt`, `words-to-blend-in.txt`, `justfile`, `.justfile` (build/scratch files from word list creation tooling)

## Naming Conventions

**Files:**
- Word lists: `orchard-street-{type}[-variant].txt` — lowercase with hyphens; `type` is `long`, `medium`, `diceware`, `alpha`, or `qwerty`; `variant` is `-dice` or `-clean`
- Documentation: `*.markdown` (explicit `.markdown` extension, not `.md`)
- Images: `kebab-case-description.png`

**Directories:**
- Single lowercase word: `lists/`, `img/`, `.planning/`, `.git/`

## Where to Add New Code

**New Word List Variant:**
- Implementation: `lists/orchard-street-{name}[-variant].txt`
- The file must be one-word-per-line plain text
- If a dice variant: prefix each line with `{dice-code}\t{word}`
- Must be verified uniquely decodable before committing

**New FAQ Entry:**
- Edit: `faq.markdown`
- Add a new H3 section (`### Question`) with answer below

**New Documentation:**
- Project-level docs: repo root as `{name}.markdown`
- Images: `img/{name}.png`

**New Planning Artifacts:**
- Codebase maps: `.planning/codebase/{DOCUMENT}.md`

## Special Directories

**`.git/`:**
- Purpose: Git version control data
- Generated: Yes (by `git init` / `git clone`)
- Committed: No (VCS internal directory)

**`.planning/`:**
- Purpose: GSD planning artifacts used during development
- Generated: Yes (by GSD tools)
- Committed: Yes (shared across development sessions)

---

*Structure analysis: 2026-06-30*
