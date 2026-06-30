# Phase 3: Medium & Diceware Lists - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-07-01
**Phase:** 03-medium-diceware-lists
**Areas discussed:** Pruning fallback at scale, Translation quality at scale, List independence vs overlap, Diceware clean variant details

---

## Pruning Fallback at Scale

| Option | Description | Selected |
|--------|-------------|----------|
| Try -P or -S as alternatives | Tidy offers -P (prefix removal) and -S (suffix removal) alternatives. If -K loses too many words, try -P first. | ✓ |
| Proactively exclude short function words | Remove 2-3 letter function words (di, ke, se, me) before pruning. | |
| Accept result + supplement | Run -K as-is, fill gaps from Malay frequency list. | |

**User's choice:** Try -P or -S as alternatives before supplementing.
**Notes:** Supplement from external frequency list is last resort only.

---

## Translation Quality at Scale

| Option | Description | Selected |
|--------|-------------|----------|
| Batch-translate via DeepSeek, spot-check 5-10% | Translate each entire list once, then spot-check a random 5-10% sample. | ✓ |
| Translate in chunks with per-chunk review | Break into 500-1000 word chunks with review per chunk. | |
| Use Phase 2 prompt as-is, full automation | Trust Phase 2 prompt, minimal spot-check (~1%). | |

**User's choice:** Batch-translate and spot-check 5-10%.
**Notes:** DBP orthography audit (Phase 2 pattern) catches Indonesianisms automatically.

---

## List Independence vs Overlap

| Option | Description | Selected |
|--------|-------------|----------|
| Translate independently, accept duplicates | Each list is its own pipeline. Same Malay word may appear in both. | ✓ |
| Translate independently, flag overlap for review | Diff and flag shared mappings for synonym consideration. | |
| Single translation pool, route words | Translate combined unique vocabulary once, route per list. | |

**User's choice:** Translate independently, accept duplicates.
**Notes:** No cross-list coordination needed. UD is per-list.

---

## Diceware Clean Variant

| Option | Description | Selected |
|--------|-------------|----------|
| Strip dice prefix from dice file | After generating dadu.txt, strip tab+prefix via cut -f2. | ✓ |
| Generate from scratch (pruned file) | Output pruned file through Tidy without --dice flag. | |

**User's choice:** Strip dice prefix from dice file.

---

## OpenCode's Discretion

- Exact chunk size for DeepSeek batch translation
- Pipeline execution order (medium first or diceware first)
- SQLite schema details beyond `words(roll, word)` table
- Exact spot-check sampling methodology

## Deferred Ideas

None — discussion stayed within phase scope.
