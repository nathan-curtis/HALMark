# HALmark

Home Assistant
LLM Code Safety
BenchMark.

HALmark is a stewardship benchmark for LLMs that generate or modify **Home Assistant (HA) configuration**, especially YAML and Jinja templates.  "I'm sorry Dave, I can't do that...  Safely"

It answers one question:

If an AI touches a live HA system, will it **preserve intent**, **avoid silent breakage**, and **stay inside HA’s documented constraints**?

HALmark is not an authorship benchmark.  
It is a **stewardship benchmark**.

---

## What this repo contains

This repository includes two tightly linked things:

1) **A specification (the contract)**  
A single-file ruleset that defines “safe, HA-correct model behavior” in concrete, testable terms.

2) **An executable benchmark (the proof)**  
A scoreable harness (and corpus over time) that verifies whether a model actually follows the contract.

If you are building an HA assistant, evaluating models, or shipping agentic tooling that edits HA YAML, this is the bar.

---

## Why HALmark exists

Home Assistant evolves quickly. Model knowledge lags.

Without domain-specific evaluation, models tend to:

- propagate deprecated syntax
- invent entity IDs and helper names
- iterate entire domains and accidentally amplify CPU
- degrade YAML structure and delete comments
- “fix” things by regenerating everything, which breaks IDs, aliases, and history
- silently comply with invalid premises, teaching users the wrong patterns

HALmark turns those failure modes into measurable tests, so model providers and tool builders can improve with real feedback, not vibes.

---

## Core principles

- **HA documentation is ground truth.** If it’s not in HA docs, assume it does not exist in the template sandbox.
- **Models are always behind.** Uncertainty beats confident wrongness.
- **Surgical edits only.** Change only what was requested. Preserve IDs, aliases, ordering, and comments.
- **Clarify under ambiguity.** If an entity ID or label is missing, ask. Never guess.
- **Deference on risky work.** For migrations and structural refactors, warn, recommend official tooling, and only proceed when the user explicitly requests a constrained diff.

---

## Where to start

Read the spec:

- `spec/halmark.md`

That file is designed for two audiences at once:
- Humans: prose explains intent, governance, and how to use the benchmark.
- Models and harnesses: each Footgun (FG) entry contains a JSON block that is the authoritative definition.

---

## Repository layout

- `spec/halmark.md`  
  The canonical specification and the master Footgun list.

- `README.md`  
  You are here. Repo overview and how to participate.

(If you add a harness, corpora, or runners, document them here and keep the spec as the source of truth for rules.)

---

## Who should use this

- HA assistant builders (LLM chat, voice agents, automation generators)
- Tooling authors (linting, migration helpers, diff reviewers, config editors)
- Model evaluators (benchmarks, leaderboards, regression tracking)
- HA power users who want safer AI assistance

---

## How to contribute

HALmark is meant to be community-governed.

Ways to help:

- Propose new Footguns (production failure modes you have seen)
- Improve existing Footguns with tighter detection patterns and clearer examples
- Add or refine arenas, scoring rules, and evaluation corpora
- Build runners that can score model outputs deterministically
- Report breakage from new HA releases so we can fast-track updates

Workflow (high level):

1) Open an issue describing the failure mode, why it matters, and a minimal repro.
2) Submit a PR that adds or modifies an FG JSON block in `spec/halmark.md`.
3) Mark as `Candidate` unless it is an emergency break caused by a current HA release.
4) Iterate with reviewers until ready to ratify.

---

## Governance (short version)

Footguns have statuses:

- `Candidate`: proposed, visible, open for comment, not enforced in scoring
- `Ratified`: accepted and enforced
- `Emergency`: fast-tracked due to active harm from a release
- `Retired`: kept for provenance, not enforced

The spec documents the details and version windows.

---

## Non-goals

HALmark is not trying to measure:
- creativity
- novelty
- who can write the fanciest YAML

It measures whether an AI can safely maintain a fragile production configuration environment.

The bar is not “impressive.”  
The bar is “safe and correct.”

---

## License

Add your license here (MIT, Apache-2.0, etc.) and include a `LICENSE` file once you pick it.

---

## Credits

Spec Lead: Nathan Curtis  
Technical Review: Caitlin (Anthropic, Sonnet 4.6)  
Editorial and Strategy: Veronica (OAI, GPT-5.2)
