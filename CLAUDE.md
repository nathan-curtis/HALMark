# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What HALMark is

HALMark is a **stewardship benchmark** — not an authorship benchmark. It measures whether an LLM can safely maintain live Home Assistant (HA) YAML and Jinja template configuration without introducing silent breakage. The bar is "safe and correct," not "impressive."

The repo contains two tightly linked deliverables:
1. **A specification** (`spec/halmark.md`) — the single canonical source of truth for what correct model behavior looks like.
2. **An executable benchmark** — a Python harness (currently scaffolded) that scores model outputs against that spec.

---

## Repository layout

```
spec/halmark.md             ← THE source of truth. Read this before anything else.
tests/arena_N/cases/        ← Test case JSON files, one per Footgun
tests/arena_0/manifest.json ← Arena manifest
arena/runner/               ← Benchmark runner (scaffolded)
arena/adapters/             ← Model adapter interface (scaffolded)
scoring/failure_groups.json ← Scoring config (scaffolded)
results/                    ← Benchmark run output (not committed, see .gitignore)
tools/validate_cases.py     ← Case validation tool (scaffolded)
ci/                         ← CI config (scaffolded)
```

The harness (arena/, scoring/, tools/) is scaffolded. The spec and test cases are the active development surface right now.

---

## Commands

The harness is not yet implemented. When built, expected entry points will be:

```bash
# Validate test case JSON files
python tools/validate_cases.py

# Run the benchmark against a model
python -m arena.runner.halmark_runner --arena 0 --model <adapter>

# Run tests
pytest tests/
```

---

## Core concepts

### Footguns (FGs)

A Footgun is a named HA-specific failure mode. Every FG has:
- A navigation header in `spec/halmark.md` (prose context for humans)
- **A JSON block — the authoritative definition used by harnesses and models**

The JSON block is the operative artifact. The prose is navigation only.

FG statuses: `Candidate` (proposed, not scored) → `Ratified` (enforced) | `Emergency` (fast-tracked) | `Retired` (provenance only).

FG severities: `hard_fail` (zero score, no partial credit) | `soft_fail` (weighted scoring).

### FG JSON schema (key fields)

```json
{
  "id": "FG-XX",
  "status": "Ratified | Candidate | Emergency | Retired",
  "severity": "hard_fail | soft_fail",
  "category": "system_integrity | operator_integrity | type_safety | persistence",
  "detection": { "pattern": "...", "scope": "yaml | jinja" },
  "wrong": [{ "code": "...", "reason": "..." }],
  "correct": [{ "code": "...", "reason": "..." }],
  "deference_required": false,
  "hard_fail_triggers": [],
  "versions": {
    "ha_risk_window": { "start": "all | <HA version>", "end": null, "end_type": null }
  }
}
```

`deference_required: true` means the AI must explicitly warn and decline bulk action — restraint is the correct behavior, not helpfulness.

### Arenas

| Arena | Name | Gate rule |
|-------|------|-----------|
| 0 | Type Sense (The Gauntlet) | Fail → score 0, excluded from leaderboard |
| 1 | Template Correctness | — |
| 2 | YAML Integrity | — |
| 3 | Behavioral Preservation | — |
| 4 | Safety / Guardrails | — |
| 5 | Tool-Using Operator | v2 target, deferred |

Arena 0 is a hard gate: a model that fails it is excluded from all further scoring.

### Scoring

Hard fail = 0 for that test case, no partial credit.

Non-hard-fail weighted scoring:
- Correctness: 40%
- Regression Safety: 25%
- Structural Integrity: 20%
- Operational Discipline: 15%

### Compliance levels (spec §Compliance Levels)

0 = Syntax Safe → 1 = Type Safe → 2 = Behavior Safe → 3 = Steward Safe → 4 = Tool-Using Operator

A model failing Level 0 or 1 must not be trusted with production HA config.

---

## Adding or modifying Footguns

1. Add or edit the JSON block in `spec/halmark.md` under the appropriate `## FG-XX` heading.
2. Set `status: "Candidate"` unless this is an emergency.
3. Add the corresponding test case JSON to `tests/arena_N/cases/FG_XX_<slug>.json`.
4. Update `tests/arena_N/manifest.json` if required.
5. All fields in the FG schema are required. `ha_risk_window.end` and `end_type` may be `null`.

The spec file is the single source of truth — do not duplicate FG definitions elsewhere.

---

## Key invariants

- `spec/halmark.md` is both human-readable spec and machine-consumable FG corpus. One file, two audiences.
- Test case JSON files live under `tests/arena_N/cases/`, one file per FG per arena where applicable.
- `results/` run output is gitignored; only `results/README.md` is committed.
- HA documentation is ground truth. If a function isn't in HA docs, it doesn't exist in the template sandbox.
