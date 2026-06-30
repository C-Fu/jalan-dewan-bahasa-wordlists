# Jalan Dewan Bahasa Wordlists — Project Guide

## GSD Workflow

This project follows GSD (Get Shit Done) workflow for structured execution.

### Active Documents

| Artifact | Location |
|----------|----------|
| Project Context | `.planning/PROJECT.md` |
| Config | `.planning/config.json` |
| Requirements | `.planning/REQUIREMENTS.md` |
| Roadmap | `.planning/ROADMAP.md` |
| State | `.planning/STATE.md` |
| Codebase Map | `.planning/codebase/` |
| Research | `.planning/research/` |

### Current State

See `.planning/STATE.md` for current phase and progress.

### Workflow Commands

- `/gsd-plan-phase <N>` — Plan a phase
- `/gsd-execute-phase <N>` — Execute a planned phase
- `/gsd-transition` — Mark phase complete, move to next
- `/gsd-progress` — Show project progress

### Mode

- **Workflow mode**: YOLO (auto-approve)
- **Granularity**: Coarse (4 phases)
- **Execution**: Parallel where possible

### Phase Overview

| Phase | Name | Status |
|-------|------|--------|
| 1 | Foundation & Documentation | Pending |
| 2 | Short Lists & Pipeline Validation | Pending |
| 3 | Medium & Diceware Lists | Pending |
| 4 | Long List & Final Verification | Pending |

## Requirements

17 v1 requirements across 7 categories. See `.planning/REQUIREMENTS.md` for details.

## Project Summary

Direct translation of Orchard Street Wordlists into Standard Malay (Malaysia). Pipeline: DeepSeek v4 Pro translation → Dedup → Normalize (Tidy) → Schlinkert Pruning (Tidy -K) → Verify (Tidy -AAAA). Output: 7 wordlist files + Malay documentation.
