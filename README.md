# Back to Basics: Agent System Design Guide

A single-file behavioral skill for designing, reviewing, and simplifying agent systems.

> **Agent system design is the art of reducing system entropy.**
> **Every layer must remove more uncertainty than it introduces.**

Derived from Shijie Wang's [Back to Basics: A Philosophy for Agent System Design](https://x.com/shijiew_/status/2082495518484107415).

English | [简体中文](./README.zh.md)

## The Problem

Agent engineering spent several years adding. More context. More tools. More skills. More loops. More agents. More orchestration. Each layer was added for a real reason — models were unreliable, context windows were small, tool use was fragile. Very little of it has ever been removed.

The result is that most agent systems cannot answer a short list of questions about themselves:

- What uncertainty does this component remove?
- Did it remove more than it introduced?
- Was it written for a model that still exists?
- What is the model deciding here, and what is the code guaranteeing?
- Is this something the system should be doing at all?

Meanwhile the failure is asymmetric. Under-scaffolding fails visibly, through unreliable behavior. Over-scaffolding looks like mature engineering while failing through latency, duplicated reasoning, brittle coordination, and architectures nobody can debug. Both sides are failure modes. Only one of them looks like one.

## The Skill

Six principles in one file:

| Principle | Corrects |
|---|---|
| **Model and Harness Are One System** | Evaluating a layer without the model it serves; crediting wins to the harness and losses to the model |
| **Start From the Bare Baseline** | Starting from a framework's maximum architecture instead of the intended model and operating constraints |
| **Make Every Layer Pay Rent** | Layers that sound sophisticated but never name what they remove |
| **Compress, Don't Coordinate** | Relocating ambiguity across more agents, roles, and retries instead of reducing it |
| **The Model Owns How, the Harness Owns What** | Confusing minimalism with deleting guardrails, and keeping "how to think" rules written for weaker models |
| **Prove It With a Living Eval** | Intuition instead of ablation; benchmark points instead of useful output per unit of time |

Plus an uncertainty map (model, context, state, execution, coordination, verification) that requires naming where a failure came from before proposing a mechanism for it.

Read it: [`skills/back-to-basics/SKILL.md`](skills/back-to-basics/SKILL.md).

## The Six Principles Explained

### 1. Model and Harness Are One System

**Never evaluate a layer apart from the model it serves.**

The system = model + prompt + context + tools + state + controls.

"The model cannot do X" and "our harness fixes X" are both unverified claims. Verify either by comparing the system with and without the layer. Do not attribute successes to the harness and failures to the model. When the model changes, re-establish the baseline; existing scaffolding may no longer help.

### 2. Start From the Bare Baseline

**Begin with the strongest model and the simplest loop.**

The baseline = one clear prompt, required context, direct tools, one loop.

Run it on real work and record where it fails. Add one layer per observed failure, then measure its effect. Never run a bare baseline against live or irreversible work; use replay or a sandbox.

Escalate only as far as the observed failure requires: no graph when a loop is sufficient; no loop when a single tool call is sufficient; no tool call when a single response is sufficient.

### 3. Make Every Layer Pay Rent

**State the uncertainty a layer removes and the cost it adds.**

Ask of each layer:

- Which observed failure does it address? A hypothetical failure is not sufficient.
- Which uncertainty does it remove?
- What does it cost in latency, tokens, hidden state, and maintenance?
- Could a better prompt, tool, or context achieve the same result?
- Which metric will decide whether it is retained?

A layer that cannot answer these questions is a candidate for removal. Audit combinations too: retrieval and planning are each defensible, and together they can produce a confident plan built on stale documents.

### 4. Compress, Don't Coordinate

**A good harness compresses complexity. A poor one relocates it.**

- Prefer one model with better context and tools over additional agents.
- Add an agent only for isolation, parallelism, specialization, or independent verification.
- Prefer explicit state over conversational handoffs.
- Prefer one outcome check over repeated self-critique; bound the correction loop.
- Memory must change a later decision. Select it, compress it, expire it.
- Allow the model to plan. Do not have planner, worker, and critic calls re-derive the same reasoning.

The test: has the decision become simpler, or has the same ambiguity been distributed across more components?

### 5. The Model Owns How, the Harness Owns What

**The model chooses how to reason. The harness defines the task, the limits, and the pass criteria.**

- Remove cognitive scaffolding as models improve: forced planning, reflection, critics, debate.
- Retain the boundaries: permissions, sandboxes, cost limits, approvals, and tests.
- A test converts an expensive judgment into an inexpensive fact.
- A stronger model raises the pass rate. It does not remove the need for the test.
- Match each guardrail to the severity of the outcome it prevents. Excessive gates stall legitimate work; insufficient gates permit irreversible errors.
- Never remove a control that policy or law requires.
- An audit log that no one reads remains a candidate for removal. Identify its consumer.

The test: could a stronger model reach this conclusion unaided? If so, it is cognitive scaffolding and can be removed. If not, it is a boundary and should be retained.

### 6. Prove It With a Living Eval

**Without evaluation, minimalism is a preference rather than an engineering decision.**

- In an existing system, remove one layer at a time and re-run the evaluation.
- Measure cost as well: latency, tokens, tool calls, approvals, and interventions.
- The objective is useful output per unit of time, not a benchmark score.
- A small benchmark gain does not justify doubled latency or an undebuggable system.
- After a model upgrade, re-run the baseline and the ablations.
- When measurements are unavailable, say so and name the cheapest decisive test.

An evaluation reveals what to add. It also reveals what can now be removed.

## Install

**Option A: Claude Code plugin (recommended)**

From within Claude Code, first add the marketplace:
```text
/plugin marketplace add shijiew/back-to-basics-skills
```

Then install the plugin:
```text
/plugin install back-to-basics@back-to-basics-skills
```

This installs the guidance as a Claude Code plugin, making the skill available across all your projects.

**Option B: CLAUDE.md (per-project, Claude Code)**

New project:
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/CLAUDE.md
```

Existing project (append):
```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/CLAUDE.md >> CLAUDE.md
```

**Option C: AGENTS.md (per-project, Codex and other compatible tools)**

Tools that automatically load `AGENTS.md` will apply this guidance to every task. Use this option only when you want always-on guidance. For selective loading, prefer the plugin or Cursor rule.

New project:
```bash
curl -o AGENTS.md https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/AGENTS.md
```

Existing project (append):
```bash
echo "" >> AGENTS.md
curl https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/AGENTS.md >> AGENTS.md
```

**Option D: Cursor project rule**

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/back-to-basics.mdc \
  https://raw.githubusercontent.com/shijiew/back-to-basics-skills/main/.cursor/rules/back-to-basics.mdc
```

The Cursor rule is agent-requested rather than always-on. See [`CURSOR.md`](CURSOR.md) for details.

Choose one project-level mechanism. Installing equivalent guidance through `AGENTS.md`, `CLAUDE.md`, and a Cursor rule can load it more than once.

## When It Applies

Designing, reviewing, simplifying, or diagnosing an agent system or harness; deciding whether to add agent-level memory, planners, critics, orchestration, retrieval, tools, guardrails, or reasoning effort; comparing single- and multi-agent designs; or re-auditing architecture after a model upgrade.

It is intended for selective loading. Prompt length is not the criterion: short architecture questions may need it, while long implementation tasks may not. When no architecture judgment is needed, answer directly.

## What It Is Not

The skill is not anti-harness, anti-planning, anti-memory, or anti-multi-agent. It rejects components without a testable role while preserving non-negotiable risk and policy controls.

Externally imposed cognitive stages — mandatory planning phases, forced reflection, role-play critics — should be re-audited as models improve. Operational scaffolding — permissions, sandboxes, deterministic rules, audit, recovery, and cost limits — is justified by risk or evidence, and better reasoning does not make a permission check unnecessary.

## Examples

[`EXAMPLES.md`](EXAMPLES.md) contains seven use cases showing common agent-system anti-patterns and simpler, testable alternatives.

## Repository Layout

```text
skills/back-to-basics/SKILL.md   canonical skill
AGENTS.md                        drop-in copy for AGENTS.md-based tools
CLAUDE.md                        per-project Claude Code instructions
.cursor/rules/back-to-basics.mdc drop-in copy for Cursor
.claude-plugin/                  Claude Code plugin manifest
CURSOR.md                        Cursor installation and usage
EXAMPLES.md                      use cases and anti-patterns
README.zh.md                     Simplified Chinese README
```

`SKILL.md` is the source of truth. `AGENTS.md`, `CLAUDE.md`, and the `.mdc` rule contain the same guidance for different installation paths; update all four together.

## Customization

These guidelines are designed to merge with project-specific instructions. Add them to your existing project instruction file — `AGENTS.md` or `CLAUDE.md` — or create a new one.

For project-specific rules, add a section like:

```markdown
## Project-Specific Guidelines

- Use TypeScript strict mode
- All API endpoints must have tests
- Follow the error handling patterns in `src/utils/errors.ts`
```

## Attribution

Core ideas are derived from Shijie Wang's ["Back to Basics: A Philosophy for Agent System Design"](https://x.com/shijiew_/status/2082495518484107415).

The single-file, behavioral-guideline packaging pattern is inspired by [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills). Andrej Karpathy neither authored nor endorsed this skill, and no content is copied from that repository.

## Core Insight

> Back to basics does not mean building less. It means being able to say what every part of the system is there for.

The graph you never drew leaves no trace in production and none in the eval. Nothing reminds you of the complexity you did not build.

The goal is not the fewest components. It is the smallest system that reliably satisfies the task and its constraints.

## License

MIT
