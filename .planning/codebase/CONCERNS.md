# Codebase Concerns

**Analysis Date:** 2026-06-30

## Tech Debt

**No Automated Tests or CI:**
- Issue: Quality relies entirely on manual verification. There are zero test files, no test runner configuration, and no CI pipeline (no `.github/` directory, no CI config files). Every wordlist change — word additions, removals, or substitutions — must be manually checked for correctness, uniqueness, decodability, and profanity.
- Files: Entire repository
- Impact: Silent regressions are possible. A single erroneous word insertion could break the uniquely-decodable property, reduce entropy, or introduce a profane word. These issues would only be found if a downstream user reports them.
- Fix approach: Add a CI pipeline (GitHub Actions) that runs basic validation on each wordlist — check sorted order, verify no duplicates, compute entropy, run Sardinas–Patterson to confirm unique decodability, scan against a profanity list.

**No Build or Release Automation:**
- Issue: There are no git tags, no release artifacts, and no automation to cut releases. The readme tells users to "download the lists as they are currently, download the latest tag/release, or fork this repository." However, `git tag --list` returns empty — no tags or releases have ever been created. Releases must be created manually.
- Files: Entire repository; `readme.markdown:10`
- Impact: Users cannot easily pin to a known-good version of the wordlists. Downstream consumers (Buttercup, Strongbox password managers) must pin to a specific commit hash rather than a semantic version.
- Fix approach: Create a release workflow (e.g., `git tag v1.0.0` + GitHub Release) with release notes documenting what changed in each wordlist. Use semver for versioning.

**No Versioning Strategy Documented:**
- Issue: The readme tells users to "download the lists as they are currently, download the latest tag/release" but there is no documented versioning scheme, no changelog, and no release notes. Downstream integrators (password managers like Buttercup and Strongbox) have no clear guidance on how to track or update their pinned wordlist versions.
- Files: `readme.markdown:10`; `faq.markdown:21-23`
- Impact: Each integration pins to whatever commit was current at integration time. It is unclear whether wordlist updates are backwards-compatible (same entropy, same word count) or breaking.
- Fix approach: Maintain a `CHANGELOG.md` describing each list's changes per version. Adopt a versioning convention (semver where major = incompatible wordlist structure changes).

## Security Considerations

**Uniquely-Decodable Property Has No Automated Guard:**
- Risk: The uniquely-decodable (UD) property is the central security differentiator of these wordlists (per the FAQ and readme). If a word change breaks the UD property, passphrases generated without delimiters could be ambiguously parsed, reducing effective entropy and potentially enabling targeted attacks. Currently there is no automated check — the maintainer must manually re-verify after every edit.
- Files: `readme.markdown:6`; `faq.markdown:34-40`; `lists/*.txt`
- Current mitigation: Manual verification by the maintainer using an external tool (Tidy/`tidy`).
- Recommendation: Bundle a lightweight UD verification script in the repository (a simple Rust or Python script that runs Sardinas–Patterson) and run it in CI on every push.

**No Automated Prefix-Word or Profanity Checks:**
- Risk: The readme states lists are "free of profane words, abbreviations, and British spellings." The FAQ distinguishes the Orchard Street lists from the EFF list by noting EFF includes words like "grope", "gonad", "ipad", etc. However, there is no automated screening — all profanity filtering is manual. The commit history shows frequent word cleanups (e.g., `removes word 'porosity'`, `remove awkward words 'peter' and 'peters'`, `remove confusing variants of the word 'acknowledge'`), suggesting the manual filter is imperfect.
- Files: `readme.markdown:7`; `faq.markdown:42-43`; git history
- Current mitigation: Manual review by the maintainer. Words are occasionally flagged and removed post-hoc.
- Recommendations: Add a CI step that checks each wordlist against a standard profanity/blocklist. Add a prefix-word check (list any word that is a prefix of another word in the same list) — even though the lists use Schlinkert pruning rather than prefix-free encoding for UD, knowing which prefix relationships exist is useful for auditing.

**License Compatibility with Upstream Data Sources:**
- Risk: The words are primarily derived from Google Books Ngram data (2012) and Wikipedia (June 2023). The project is licensed CC BY-SA 4.0. The readme notes that Wikipedia text was CC BY-SA 4.0 at the time of extraction. Google Books Ngram data licensing is less clear — the readme states the project has "no association with Google" and provides no Google-specific license note. If Google's ngram data has additional restrictions, downstream usage could be impacted.
- Files: `readme.markdown:156-161`
- Current mitigation: The readme includes a licensing section noting Wikipedia's CC BY-SA 4.0 license and disclaims association with Google/Wikipedia.
- Recommendations: Clarify the licensing status of Google Books Ngram-derived words explicitly. Add a `LICENSE` file (not just the readme mention) to make the CC BY-SA 4.0 terms unambiguous for downstream integrators.

## Performance Bottlenecks

**Wordlists Are Static Text Files Only:**
- Problem: The only way to consume these wordlists is to download the entire text file or fork the repository. There is no API, no CDN, no programmatic access. For a password manager bootstrapping itself, this means either bundling the entire file (46,504 lines across all lists) or fetching from GitHub raw.
- Files: `lists/*.txt`
- Cause: The project is intentionally simple — just wordlists and docs.
- Improvement path: This is a low concern given the project's scope, but if usage grows, consider publishing to a package registry (npm, cargo) or providing a minimal API. Not a bottleneck today.

## Fragile Areas

**Single-Maintainer Risk:**
- Files: Entire repository
- Why fragile: Sam Schlinkert (`sts10`) is the sole committer across all 124 commits. Repository has no CODE_OF_CONDUCT, no CONTRIBUTING guide, and no issue templates — lowering the barrier for community contributions also means no structured process for handover. If the maintainer becomes unavailable, there is no documented process for triage, release, or continued curation.
- Test coverage: N/A — no tests at all.
- Safe modification: Any change should be validated against the UD property and profanity-cleaned. The external tool `tidy` is required for UD validation.

**Wordlist Data Freshness:**
- Files: `lists/*.txt`
- Why fragile: The word sources are snapshots — Google Books Ngram data is from 2012; Wikipedia data was pulled in June 2023. There is no automated freshness check or schedule for re-pulling data. Language evolves; new common words (e.g., "covid", "zoom", "tiktok") may be absent from the lists, while outdated words remain. The long list claims to be "made up of common words found in (English) Wikipedia and Google Books" but cannot reflect current usage without manual re-runs of the external scraping tools.
- Test coverage: None.
- Safe modification: To refresh, the maintainer must re-run `common_word_list_maker` (for Google Books) and `wikipedia-word-frequency` (for Wikipedia), then re-apply the Schlinkert pruning process using `tidy`.

## Dependencies at Risk

**External Tooling for List Generation:**
- Risk: The wordlists are produced via three external tools owned by the same maintainer: `common_word_list_maker` (Google Books scraper), `wikipedia-word-frequency` (IlyaSemenov), and `tidy` (Schlinkert pruning). If any of these tools become unmaintained or incompatible with current language runtimes, the ability to regenerate or update the lists is impaired.
- Impact: Cannot refresh word data or fix UD issues without alternative tooling.
- Migration plan: The validation logic (UD checking) is well-defined by the Sardinas–Patterson algorithm and could be re-implemented in ~100 lines of Python. The scraping tools are harder to replace but the underlying data sources (Google Ngrams, Wikipedia dumps) are stable.

**Google Books Ngram 2012 Data:**
- Risk: The ngram data is from 2012 — over 14 years old at time of analysis. Google provides newer ngram datasets (2019), but the tooling (`common_word_list_maker`) targets the 2012 snapshot. The lists are therefore biased toward vocabulary pre-2012.
- Impact: Newer common words are systematically excluded.
- Migration plan: Update `common_word_list_maker` to support the 2019 ngram dataset, then regenerate.

## Missing Critical Features

**No Changelog:**
- Problem: When words are added or removed (60 commits touching wordlists), there is no changelog explaining which lists changed, which words were affected, and why. Downstream integrators (Buttercup, Strongbox) cannot easily assess whether an update is safe.
- Blocks: Informed adoption of updates by downstream projects.

**No Bundle Validation Script:**
- Problem: The repository contains only raw text files. Users who want to verify wordlist integrity (sortedness, no duplicates, UD property) have no bundled tool to do so — they must install `tidy` separately.
- Blocks: Trustless verification by end users.

## Test Coverage Gaps

**Untested Area — Everything:**
- What's not tested: Every aspect of the wordlists — word uniqueness, alphabetical sorting, entropy calculations, unique decodability, profanity absence, prefix-word relationships, dice-roll mapping correctness.
- Files: `lists/*.txt`
- Risk: Any manual editing mistake (e.g., a duplicate word, an out-of-order insertion, a word that breaks UD) goes undetected until a downstream user reports it. The commit history shows multiple ad-hoc cleanup commits (`swap out 'herald' for 'noun'`, `removes niche medical word 'hepatic'`, `remove awkward word 'hank'`), suggesting the manual process catches issues only after they've been introduced.
- Priority: High

**Untested Area — Tooling Integration:**
- What's not tested: The correctness of the pipeline that produced the lists (Google Books scraper → Wikipedia frequency extractor → Tidy pruning) is not validated. If a new version of `tidy` changes its pruning behavior, there is no test to detect unexpected drift in the output lists.
- Files: External repositories (`sts10/common_word_list_maker`, `IlyaSemenov/wikipedia-word-frequency`, `sts10/tidy`)
- Risk: Silent drift in list quality if upstream tools change.
- Priority: Medium

---

*Concerns audit: 2026-06-30*
