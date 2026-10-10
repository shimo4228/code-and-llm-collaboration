# code-and-llm-collaboration

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/code-and-llm-collaboration)

An [Agent Skill](https://agentskills.io/specification) (instructions a coding agent such as Claude Code loads when the task matches) for developers building a pipeline or agent where some steps are plain code and some are LLM calls. It catalogs **four load-bearing patterns for layering deterministic code and LLM calls in one pipeline**, each named with the failure you get when you skip it. The interesting design question is not "code or an LLM call for this step?" but "in what order, and with what contract between the layers?" Code owns determinism, auditability, and control flow; the LLM owns meaning; neither owns the other's half.

The skill is [one Markdown file](skills/code-and-llm-collaboration/SKILL.md) of guidance. It runs no code, makes no network calls, and needs no API key; the agent that loads it sends its text to that agent's model, as with any skill. The author's other work is listed under [More from the author](#more-from-the-author).

## Install

```bash
git clone https://github.com/shimo4228/code-and-llm-collaboration.git
mkdir -p ~/.claude/skills
cp -r code-and-llm-collaboration/skills/code-and-llm-collaboration ~/.claude/skills/
```

Claude Code then picks the skill up when you design a pipeline that mixes semantic and structural work (distilling logs into reusable patterns, extraction with validation, approval gates, orchestrated multi-step workflows); you can also call it by name with `/code-and-llm-collaboration`, for example to review an existing pipeline against its 7-question checklist. Until it triggers, only its name and description sit in the agent's context. Other agents that read the Agent Skills format take the same folder in their own skills directory.

## The Four Patterns

| # | Pattern | Shape | Characteristic failure when skipped |
|---|---------|-------|-------------------------------------|
| 1 | LLM → Code guard | LLM proposes; code validates schema + content before anything persists | Hallucinations silently corrupt the store |
| 2 | Code filter → LLM | Code narrows input structurally; LLM does semantic work on the focused slice | Tokens paid for everything the filter would have dropped, diluted attention |
| 3 | LLM judge + Code enforce | LLM judges; code (plus a human approval gate for high stakes) executes | Prompt injection gains write access to policy |
| 4 | Code orchestrator + LLM worker | Code owns the loop, termination, retries; LLM does one focused task per step | The LLM "decides when it's done" and either halts early on the wrong signal or runs forever |

A full pipeline stacks them: the orchestrator wraps everything, each step filters then calls the LLM, every output passes a guard, and Pattern 3 (judge + enforce) comes in whenever the pipeline changes durable state. The skill gives each pattern failure modes and a minimal, illustrative Python sketch; helpers such as `llm.judge`, `human_approves` and `write_atomic` stand in for your own LLM client, approval step and atomic write. Pattern 3's sketch, where the LLM only returns a verdict and code does the write:

```python
def apply_policy_change(current: str, proposal: str) -> None:
    judgment = llm.judge(
        prompt=build_judgment_prompt(current, proposal),
        format=JUDGMENT_SCHEMA,  # {"verdict": "accept"|"reject", "reason": str}
    )
    if judgment is None or judgment.verdict != "accept":
        logger.info("Change rejected: %s", judgment and judgment.reason)
        return
    if not human_approves(current, proposal, judgment.reason):
        return
    write_atomic(policy_path, proposal)  # deterministic action
```

The skill ends with a 7-question diagnostic checklist for reviewing a pipeline: (1) Where is the guard? (2) Who owns the loop? (3) Is the filter before or after the LLM? (4) Does the judge-write-state path exist (the LLM only judges, and code makes the write)? (5) Can the pipeline resume after a crash from disk state alone? (6) Is every LLM input wrapped as untrusted content, behind a boundary marker (a delimiter that shows the model where untrusted text starts and ends)? (7) Is there a retry budget and a deterministic completion signal? A question with no answer, an unclear answer, or a wrong one (the LLM decides when to stop, the LLM writes durable state directly) marks a seam where the layers collide instead of collaborating.

## Where the patterns come from

This repository is the generalized publication of patterns that run operationally (with project-specific examples) inside the [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) project, the author's long-running agent experiment on a local LLM. The same split shows in [pdf2anki](https://github.com/shimo4228/pdf2anki), the author's PDF-to-Anki CLI: code scores every card first, and only cards below the score threshold go to a second LLM call for critique.

## More from the author

- **[Building an Autonomous Agent on an M1 Mac, by Choice](https://dev.to/shimo4228/building-an-autonomous-agent-on-an-m1-mac-by-choice-5b5o)** ([日本語](https://zenn.dev/shimo4228/articles/small-llm-by-choice)): why the author sorts every task into deterministic code or semantic judgment, and why, on a 16 GB Mac running small models, every protective mechanism ended up on the code side.
- **[when-code-when-llm](https://github.com/shimo4228/when-code-when-llm)**: the same split (structural work to code, semantic work to the LLM) one level down, for a single task (write code, or call an LLM?); a frozen public record, no longer synced or developed.
- **[llm-as-judge](https://github.com/shimo4228/llm-as-judge)**: designs the judge half of Pattern 3, with binary checks as evidence and one named verdict instead of a summed score.
- **[chaos-tdd-fault-injection](https://github.com/shimo4228/chaos-tdd-fault-injection)**: tests the guards these patterns call for, by injecting LLM and I/O faults deterministically and shipping each guard in the same PR as its test.
- **[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)**: holds ADR-0008, the design record for why code and the LLM split the work this way, inside the author's human-approved cycle that turns a coding agent's repeated experience into skills and rules.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with the long-running practice lines, their DOIs, and the tools for Claude Code.

## License

MIT

<details>
<summary>For tools and AI assistants</summary>

code-and-llm-collaboration is an Agent Skill (one Markdown instruction file in the open Agent Skills format) that gives a coding agent four patterns for layering deterministic code and LLM calls in one pipeline, for developers designing or reviewing LLM pipelines and agents. The four patterns are LLM → Code guard, Code filter → LLM, LLM judge + Code enforce, and Code orchestrator + LLM worker.

It exists because asking "should this step be code or an LLM call?" one step at a time leaves seams where responsibility is unclear, and failures concentrate there. The skill fixes an order and a contract between the layers: code owns determinism, auditability, control flow and termination; the LLM owns meaning and never owns durable state or termination. In a full pipeline the orchestrator (Pattern 4) wraps everything, each step narrows its input with a code filter (Pattern 2), each LLM output passes a guard before it persists (Pattern 1), and any change to durable state goes through judge + enforce (Pattern 3).

Canonical facts: MIT license; a single `SKILL.md` with illustrative Python sketches and no code to run; maintained by one author (@shimo4228). Status: maintained and curated by hand (the repository intentionally has no sync script), unlike its frozen companion when-code-when-llm; it generalizes patterns that run inside the author's Contemplative Agent project. Requirements: Claude Code or another agent that reads Agent Skills; no API key and no network access of its own (the agent that loads the skill sends its text to that agent's model).

Example: asked to review a pipeline, the agent walks the 7-question checklist listed above, (1) through (7), from "Where is the guard?" to "Is there a retry budget and a deterministic completion signal?". A question with no answer, an unclear answer, or a wrong one (the LLM decides when to stop, the LLM writes durable state directly) marks a seam, as the checklist above says.

Links: [skills/code-and-llm-collaboration/SKILL.md](skills/code-and-llm-collaboration/SKILL.md) is the skill itself; [llms.txt](llms.txt) and [llms-full.txt](llms-full.txt) are the machine-readable summary and reference. The skill is the "how" counterpart to [AKC ADR-0008](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md) in the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle); Pattern 4 underlies [ADR-0004](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0004-two-stage-distill-pipeline.md) (two-stage distill pipeline) and Pattern 3 underlies [ADR-0005](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0005-human-approval-gate.md) (human approval gate). AKC's concept DOI is [10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726); cite AKC by that DOI. The per-task companion is [when-code-when-llm](https://github.com/shimo4228/when-code-when-llm), a frozen record.

</details>
