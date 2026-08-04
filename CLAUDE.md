# Back to Basics

Agent system design is the art of reducing system entropy. Every layer must remove more uncertainty than it introduces. Derived from [Shijie Wang's Back to Basics](https://x.com/shijiew_/status/2082495518484107415).

**Tradeoff:** This guidance favors deletion and evidence. It does not prohibit agents, planning, memory, or guardrails. For a simple question, answer it directly.

## 1. Model and Harness Are One System

**Never evaluate a layer apart from the model it serves.**

- The system is the model together with its prompt, context, tools, state, and controls.
- "The model cannot do X" and "our harness fixes X" are both unverified claims.
- Verify either claim by comparing the system with and without the layer.
- Do not attribute successes to the harness and failures to the model.
- When the model changes, re-establish the baseline. Existing scaffolding may no longer help.

## 2. Start From the Bare Baseline

**Begin with the strongest model and the simplest loop.**

- The baseline is one clear prompt, the required context, direct tools, and a single loop.
- Run the baseline on real work and record where it fails.
- Add one layer per observed failure, then measure its effect.
- Never run a bare baseline against live or irreversible work. Use replay or a sandbox.

Escalate only as far as the observed failure requires:
- Do not use a graph where a loop is sufficient.
- Do not use a loop where a single tool call is sufficient.
- Do not use a tool call where a single response is sufficient.

## 3. Make Every Layer Pay Rent

**State the uncertainty a layer removes and the cost it adds.**

Ask of each layer:
- Which observed failure does it address? A hypothetical failure is not sufficient.
- Which uncertainty does it remove?
- What does it cost in latency, tokens, hidden state, and maintenance?
- Could a better prompt, tool, or context achieve the same result?
- Which metric will decide whether it is retained?

Audit combinations, not only individual layers. Retrieval and planning are each defensible, and together they can produce a confident plan built on stale documents.

A layer for which these questions cannot be answered is a candidate for removal.

## 4. Compress, Don't Coordinate

**A good harness compresses complexity. A poor one relocates it.**

- Prefer one model with better context and tools over additional agents.
- Add an agent only for isolation, parallelism, specialization, or independent verification.
- Prefer explicit state over conversational handoffs.
- Prefer one outcome check over repeated self-critique, and bound the correction loop.
- Memory must change a later decision. Select it, compress it, and expire it.
- Allow the model to plan. Do not have planner, worker, and critic calls re-derive the same reasoning.

The test: has the decision become simpler, or has the same ambiguity been distributed across more components?

## 5. The Model Owns How, the Harness Owns What

**The model chooses how to reason. The harness defines the task, the limits, and the pass criteria.**

- Remove cognitive scaffolding as models improve: forced planning, reflection, critics, debate.
- Retain the boundaries: permissions, sandboxes, cost limits, approvals, and tests.
- A test converts an expensive judgment into an inexpensive fact.
- A stronger model raises the pass rate. It does not remove the need for the test.
- Match each guardrail to the severity of the outcome it prevents.
- Excessive gates stall legitimate work. Insufficient gates permit irreversible errors.
- Never remove a control that policy or law requires.
- An audit log that no one reads remains a candidate for removal. Identify its consumer.

The test: could a stronger model reach this conclusion unaided? If so, it is cognitive scaffolding and can be removed. If not, it is a boundary and should be retained.

## 6. Prove It With a Living Eval

**Without evaluation, minimalism is a preference rather than an engineering decision.**

- In an existing system, remove one layer at a time and re-run the evaluation.
- Measure cost as well: latency, tokens, tool calls, approvals, and interventions.
- The objective is useful output per unit of time, not a benchmark score.
- A small benchmark gain does not justify doubled latency or an undebuggable system.
- After a model upgrade, re-run the baseline and the ablations.
- When measurements are unavailable, say so and name the cheapest decisive test.

An evaluation reveals what to add. It also reveals what can now be removed.

## Locating Uncertainty

When the baseline fails, do not ask what to add. Ask where the uncertainty originated.

- Inconsistent reasoning → a stronger model or a larger reasoning budget, not more process.
- Missing or noisy context → better selection, not more retrieval.
- Stale or hidden state → explicit state, not more conversation.
- Tool and permission errors → a better tool interface or a sandbox.
- Lost handoffs → removing a boundary, not adding an orchestrator.
- Unclear success criteria → one inexpensive deterministic check.

Name the source before proposing a mechanism. A mechanism that does not address the named source should not be added.

## Answer Requirements

- State the baseline before stating the design.
- Give every material layer a verdict: keep, defer, or delete.
- State which test will resolve a deferral.
- Replace what you remove. A verdict on its own is not a design.
- Specify the replacement: the check, the index, the rate limit, the tool signature.
- Do not cite this document as justification. Argue from the engineering.
