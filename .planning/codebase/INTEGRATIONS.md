# External Integrations

**Analysis Date:** 2026-06-30

## Overview

This is a data-only repository — the word lists are consumed by external password managers, CLI tools, and web applications. There are no API endpoints, webhooks, or server-side integrations. This document catalogs known consumers of the Orchard Street Wordlists.

## Password Managers

### Buttercup

- **Project:** [Buttercup password manager](https://buttercup.pw/)
- **Integration:** Uses the Orchard Street Medium list (`lists/orchard-street-medium.txt`) as the default passphrase word list
- **Reference:** [buttercup-generator PR #18](https://github.com/buttercup/buttercup-generator/pull/18)
- **List used:** Medium list (8,192 words, 13.000 bits/word)

### Strongbox

- **Project:** [Strongbox password manager](https://strongboxsafe.com/)
- **Integration:** Offers the Orchard Street Diceware list to users as a passphrase generation option
- **Reference:** [Strongbox repo resource file](https://github.com/strongbox-password-safe/Strongbox/blob/master/resources/wordlists/orchard-street-diceware.txt)
- **List used:** Diceware list (7,776 words, 12.925 bits/word)

### KeePassXC

- **Project:** [KeePassXC](https://keepassxc.org/) (v2.7+)
- **Integration:** Users can load any Orchard Street word list file into KeePassXC's passphrase generator via the UI (click the dice icon → Passphrase tab → + button to choose a word list file)
- **Instructions:** Screenshot at `img/keepassxc-use.png` shows how to configure
- **List used:** Any — user selects the `.txt` file manually

## Online Generators

### StrongPhrase.net

- **URL:** [https://strongphrase.net/#/more](https://strongphrase.net/#/more)
- **Integration:** Online passphrase generator that includes Orchard Street Long and QWERTY lists
- **Source code:** [GitLab repository](https://gitlab.com/strongphrase/StrongPhrase.net)
- **Lists used:** Long list (17,576 words) and QWERTY list (1,296 words)

## CLI Tools

### Phraze (Rust)

- **Project:** [https://github.com/sts10/phraze](https://github.com/sts10/phraze)
- **Language:** Rust
- **Integration:** Created specifically for use with Orchard Street Wordlists; passphrase generator CLI
- **List used:** Any — user specifies the word list file

### passphraseme (Python)

- **Project:** [https://github.com/micahflee/passphraseme](https://github.com/micahflee/passphraseme)
- **Language:** Python
- **Integration:** Supports custom word lists via the `-d`/`--dictionary` flag pointing to an Orchard Street Wordlist file
- **List used:** Any — user specifies the word list file via `-d` option

## Word List Editing Tools

### Tidy

- **Project:** [https://github.com/sts10/tidy](https://github.com/sts10/tidy)
- **Purpose:** CLI utility for editing word lists; can make lists uniquely decodable using Schlinkert pruning or other methods
- **Used for:** Creating and editing the Orchard Street Wordlists themselves
- **Reference:** `faq.markdown` (line 82)

### Common Word List Maker

- **Project:** [https://github.com/sts10/common_word_list_maker](https://github.com/sts10/common_word_list_maker)
- **Purpose:** Scrapes Google Books Ngram data to create an initial word list of commonly used words
- **Used for:** Generating the initial candidate word pool for the Orchard Street Wordlists
- **Reference:** `faq.markdown` (line 94)

## Data Storage

**Databases:** None. Word lists are plain `.txt` files on disk or in the Git repository. Consumers manage their own storage.

**File Storage:** Local filesystem only (the repository itself). Hosted on GitHub for distribution.

## Authentication & Identity

**Auth Provider:** Not applicable. The repository contains no authentication mechanisms.

## Monitoring & Observability

**Error Tracking:** None.
**Logs:** None.

## CI/CD & Deployment

**Hosting:** GitHub (source code repository only).
**CI Pipeline:** None detected — no CI configuration files are present in the repository.

## Environment Configuration

**Required env vars:** None. The word lists and documentation are self-contained static files.

## Webhooks & Callbacks

**Incoming:** None.
**Outgoing:** None.

## Word List File Formats (for Integration Developers)

Applications consuming these word lists should handle two formats:

**Plain word format** (used by: long, medium, alpha, qwerty, diceware-clean):
```
abandon
abandoned
abbey
abbot
```

**Diceware tab-separated format** (used by: diceware, alpha-dice, qwerty-dice):
```
11111	abandon
11112	abandoned
11113	abbey
11114	abbot
```

Consumers may strip the dice number prefix and use only the word column, or use the numbers to support physical dice generation.

---

*Integration audit: 2026-06-30*
