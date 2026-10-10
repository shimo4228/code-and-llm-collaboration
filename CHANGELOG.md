# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Fixed

- Diagnostic checklist: the closing rule judged every question by "no" or "unclear", but questions 1–3 are not yes/no questions and question 4 was asked so that "yes" was the bad answer. A seam is now a question with no answer, an unclear answer, or a wrong one, with the two typical wrong answers named. Question 4 reads "Can the LLM write durable state without a code-enforce step?", so "yes" is plainly the wrong answer.

## [1.0.0] — 2026-06-12

Initial release as a standalone skill repository.

### Added

- `skills/code-and-llm-collaboration/SKILL.md` — four load-bearing patterns for layering deterministic code and LLM calls in one pipeline (LLM→Code guard, Code filter→LLM, LLM judge + Code enforce, Code orchestrator + LLM worker), with when-to-use, failure modes, minimal sketches, a composition guide, and a diagnostic checklist.

### Notes

- Previously published inside the [Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle) repository as a design-pattern skill under `docs/skills/`. Extracted here so the skill is installable on its own; the paired decision record [AKC ADR-0008](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md) stays in the research repository.
- This repository is manually curated: it generalizes an operational variant that runs inside the [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) project. There is intentionally no automated sync script.
