# code-and-llm-collaboration

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/code-and-llm-collaboration) [![GitMCP](https://img.shields.io/endpoint?url=https://gitmcp.io/badge/shimo4228/code-and-llm-collaboration)](https://gitmcp.io/shimo4228/code-and-llm-collaboration) [![View Code Wiki](https://assets.codewiki.google/readme-badge/static.svg)](https://codewiki.google/github.com/shimo4228/code-and-llm-collaboration)

An [Agent Skill](https://agentskills.io/specification) that catalogs **four load-bearing patterns for layering deterministic code and LLM calls in one pipeline**. The interesting design question is not "which one should I use?" but "in what order, and with what contract between the layers?" — code owns determinism, auditability, and control flow; the LLM owns meaning; neither owns the other's half.

## Install

### Claude Code

```bash
# Copy into your global skills directory
cp -r skills/code-and-llm-collaboration ~/.claude/skills/code-and-llm-collaboration
```

### SkillsMP

```bash
/skills add shimo4228/code-and-llm-collaboration
```

## The Four Patterns

| # | Pattern | Shape | Characteristic failure when skipped |
|---|---------|-------|-------------------------------------|
| 1 | LLM → Code guard | LLM proposes; code validates schema + content before anything persists | Hallucinations silently corrupt the store |
| 2 | Code filter → LLM | Code narrows input structurally; LLM does semantic work on the focused slice | 100× token cost, diluted attention |
| 3 | LLM judge + Code enforce | LLM judges; code (plus a human gate for high stakes) executes | Prompt injection gains write access to policy |
| 4 | Code orchestrator + LLM worker | Code owns the loop, termination, retries; LLM does one focused task per step | The LLM "decides when it's done" — it never is |

Real pipelines stack all four: the orchestrator wraps everything, each step filters then calls the LLM, every output passes a guard, and any durable-state mutation goes through judge + enforce. The skill includes minimal code sketches, failure-mode lists, and a 7-question diagnostic checklist.

## When It Triggers

- Designing a pipeline that mixes semantic work (extraction, classification, distillation) with structural work (validation, filtering, dispatch)
- Adding an approval gate, a self-updating config, or a multi-step agentic workflow
- Reviewing a pipeline where "who owns termination?" has no clear answer

## Curation model

This repository is **manually curated**, not script-synced: it is the generalized publication of patterns that run operationally (with project-specific examples) inside the [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) project. Improvements flow here editorially, the same way the paired ADRs were extracted.

## Related skill

[when-code-when-llm](https://github.com/shimo4228/when-code-when-llm) applies the same structural-vs-semantic axis one level down: the per-task "which tool for this one property" decision. This skill is the per-pipeline composition catalog.

## About this skill

This skill is a design-pattern skill from the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) research line — a Zenodo-citable six-phase bidirectional growth loop ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)) for sustaining intent alignment between an AI agent and its operator over time. It is the "how" counterpart to [AKC ADR-0008](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md); Pattern 4 underlies [ADR-0004](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0004-two-stage-distill-pipeline.md) and Pattern 3 underlies [ADR-0005](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0005-human-approval-gate.md). AKC is one of three research lines by [@shimo4228](https://github.com/shimo4228), alongside [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)) — autonomous agents grounded in four contemplative axioms — and [Agent Attribution Practice (AAP)](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)) — harness-neutral ADRs on accountability distribution.

## License

MIT
