# Examples

Agent-system examples demonstrating the Back to Basics principles. Each example shows a common overcomplicated approach and a simpler, testable alternative.

---

## 1. Multi-Agent Documentation Q&A

**User Request:** "Build an internal tool that answers questions about company documentation."

**❌ What Overbuilt Systems Do**

- Create planner, researcher, critic, and judge agents.
- Coordinate them through a state graph.
- Ask one model call to review another model call.
- Rely on prompts to enforce document permissions.

This design adds handoffs and latency before demonstrating that one agent cannot complete the task.

**✅ What Should Happen**

Start with one model, one prompt, and direct documentation tools:

```text
search_docs(query, user_id) -> retrieved documents
answer(question, retrieved_documents) -> answer with citations
```

Keep the boundaries deterministic:

- Filter document permissions during retrieval.
- Confirm that every citation refers to a retrieved document.
- Reject citations to stale or missing documents.
- Evaluate the system on real employee questions.

Add another agent only if the baseline reveals a failure that requires isolation, specialization, parallelism, or independent verification.

---

## 2. Store Everything in a Vector Database

**User Request:** "Make our coding agent remember every conversation, file, and decision."

**❌ What Overbuilt Systems Do**

- Embed every conversation and every file read.
- Retrieve the top 50 chunks on every turn.
- Treat more retrieved content as better memory.
- Serve embedded file snapshots after the source files have changed.

This increases prompt size, retrieval latency, and the risk of using stale information.

**✅ What Should Happen**

Use memory only when it changes a later decision:

- Keep a small set of durable decisions as explicit state.
- Let the agent search prior conversations when needed.
- Read current files directly instead of retrieving old snapshots.
- Retrieve a small number of relevant, reranked results.
- Expire information that no longer affects current work.

Compare three configurations on the same tasks:

```text
memory disabled
top-5 retrieval
top-50 retrieval
```

Measure accepted task completion, unnecessary actions, latency, and token cost. Keep the smallest configuration that performs best.

---

## 3. Add a Planner and a Critic

**User Request:** "Should I add a planning stage and a self-critique loop before execution?"

**❌ What Overbuilt Systems Do**

- Add planning because the task might become complex.
- Add a critic because the model might make mistakes.
- Let planner, worker, and critic calls repeat the same reasoning.
- Create an open-ended correction loop.

The proposed layers address hypothetical failures rather than observed ones.

**✅ What Should Happen**

Start with the direct baseline:

```text
clear task prompt
required context
direct tools
single tool-use loop
```

Collect real failures and classify their source:

- Reasoning failure → improve the model, reasoning budget, or task framing.
- Missing context → improve context selection.
- Tool failure → improve the tool interface.
- Invalid output → add a deterministic check.
- Irreversible action → add an approval or permission boundary.

Use tests, schemas, idempotency keys, and read-back checks before adding another model call. Bound any correction loop.

---

## 4. Refund Agent With Compliance Requirements

**User Request:** "Simplify a slow refund agent that has six agents and four approval gates."

**❌ What Overbuilt Systems Do**

- Model order lookup, policy retrieval, and payment execution as separate agents.
- Add several approval gates that reviewers routinely approve.
- Remove all controls when optimizing for speed.
- Use another model as the primary payment-safety check.

Both extremes are incorrect: unnecessary coordination adds latency, while removing required controls creates unacceptable risk.

**✅ What Should Happen**

Use one model with direct tools:

```text
get_order(order_id)
get_refund_policy(order_id)
issue_refund(order_id, amount, idempotency_key)
```

Enforce payment controls outside the model:

- Cumulative refunds cannot exceed the order total.
- The order must be in a refundable state and policy window.
- Currency and payment method must match.
- Credentials must enforce the refund limit.
- Every payment request must use an idempotency key.

Keep one human approval boundary where policy or segregation of duties requires it. Record each decision in an append-only audit log.

Replay real refund cases against the full and reduced designs. Measure correctness, latency, cost, approval overrides, and intervention rate.

---

## 5. Maximum Reasoning Effort Everywhere

**User Request:** "Use maximum reasoning on every request because it scores four points higher on our benchmark."

**❌ What Overbuilt Systems Do**

- Optimize only for the aggregate benchmark score.
- Ignore doubled latency and increased cost.
- Keep planner and critic stages that duplicate the model's added reasoning.
- Add a routing layer without testing whether the router improves the system.

**✅ What Should Happen**

Compare complete configurations, not isolated benchmark scores:

```text
standard reasoning
maximum reasoning
difficulty-based escalation
```

Measure accepted output, latency, cost, retries, abandonment, and human intervention.

If maximum reasoning helps, re-evaluate planning, reflection, and critic stages that may now be redundant. Keep an escalation router only if it outperforms both fixed configurations after accounting for its own errors and cost.

The objective is useful output per unit of time and cost.

---

## 6. Long-Horizon Migration Drift

**User Request:** "A database-migration agent drifts after many tool calls. Should we add a supervisor agent?"

**❌ What Overbuilt Systems Do**

- Add a supervisor that watches the same transcript as the worker.
- Let the model define and revise completion criteria.
- Keep one context alive for hundreds of tool calls.
- Ask another model to judge whether the migration is complete.

The supervisor has no independent evidence and cannot reliably detect errors shared with the worker.

**✅ What Should Happen**

Make migration state explicit:

```text
migration ledger generated from the database schema
one bounded stage per fresh context
idempotent and resumable migration scripts
deterministic verify_all command
```

Generate the ledger mechanically from the schema. Mark work complete only when deterministic checks pass:

- Source and destination row counts match.
- Required columns contain no unexpected null values.
- Foreign-key integrity holds.
- Checksums and aggregate values match.
- Sentinel queries pass for high-risk transformations.

Parallelize only independent tables with isolated state. Keep staging credentials, execution limits, logs, and human approval for production changes.

---

## 7. Legacy Harness on a New Model

**User Request:** "Our old text-based reasoning harness behaves erratically after a model upgrade."

**❌ What Overbuilt Systems Do**

- Keep a parsed text scratchpad alongside native tool calling.
- Require a visible `THINK:` block despite native reasoning support.
- Require several candidate approaches before every action.
- Tune prompt placement until one model snapshot behaves correctly.

This creates competing action channels and duplicates reasoning already performed by the model.

**✅ What Should Happen**

Test four configurations:

```text
legacy harness unchanged
native tools with legacy planning instructions
native tools without the THINK block
native tools without either cognitive layer
```

Use native tool calls as the only execution channel. Preserve reasoning blocks and tool identifiers exactly as required by the model API.

Reinvest in tool interfaces:

- Return focused errors with a recovery action.
- Return failing test names instead of complete logs.
- Make write operations idempotent.
- Detect duplicate execution during migration.

Keep the smallest configuration that maintains or improves accepted outcomes.

---

## Anti-Patterns Summary

| Principle | Anti-Pattern | Better Approach |
|---|---|---|
| **Model and Harness Are One System** | Judge the model or harness in isolation | Compare complete configurations on the same tasks |
| **Start From the Bare Baseline** | Add layers for hypothetical failures | Begin with one prompt, direct tools, and one loop |
| **Make Every Layer Pay Rent** | Keep a layer without a deciding metric | Name its benefit, cost, and retention test |
| **Compress, Don't Coordinate** | Split one decision across several agents | Use explicit state and direct tools |
| **The Model Owns How, the Harness Owns What** | Force planning, reflection, and debate stages | Let the model reason; keep task and safety boundaries |
| **Prove It With a Living Eval** | Optimize only for benchmark quality | Measure accepted output, latency, cost, and intervention |

## Key Insight

The overbuilt examples are not wrong because they use agents, retrieval, memory, planning, or guardrails. They are wrong because those layers were added before the system demonstrated a need for them.

A good agent system:

- Begins with the simplest viable baseline.
- Adds one mechanism for one observed failure.
- Keeps operational controls that address real risk or policy requirements.
- Replaces expensive judgment with deterministic checks when possible.
- Re-evaluates inherited layers after model or task changes.
- Measures both quality and operational cost.

**The goal is not the fewest components. The goal is the smallest system that reliably satisfies the task and its constraints.**
