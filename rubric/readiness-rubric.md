# Production Readiness Rubric

Score your agent **0–5 on each of the ten pillars**. Total out of 50.

| Score | Meaning |
|-------|---------|
| 0 | Not considered |
| 1 | Handled informally |
| 2 | Partially designed |
| 3 | Designed and documented |
| 4 | Tested and monitored |
| 5 | Production-grade with feedback loops |

## Scorecard

| # | Pillar | Score (0–5) | Evidence / notes |
|---|--------|:-----------:|------------------|
| 1 | Prompt | | |
| 2 | Goal | | |
| 3 | Trajectory | | |
| 4 | Context | | |
| 5 | Memory | | |
| 6 | Tools | | |
| 7 | Intelligence | | |
| 8 | Execution | | |
| 9 | Governance | | |
| 10 | Evaluation | | |
| | **Total** | **/50** | |

## Interpretation

- **Below 30/50** → prototype. Impressive demos do not change this.
- **Any of Execution, Governance, or Evaluation at 0–1** → not production-ready, regardless of total. These three are where *production trust* is won or lost.
- **40+/50 with all three of Execution/Governance/Evaluation at ≥3** → genuinely production-grade; focus on continuous improvement.

## What "good" looks like per pillar

| Pillar | A 4–5 looks like |
|--------|------------------|
| Prompt | Versioned prompts, output schemas, regression tests on held-out cases |
| Goal | Explicit success criteria, stopping conditions, escalation triggers in code |
| Trajectory | Designed workflow with failure branches, retries, and validation loops |
| Context | Curated retrieval with provenance; measured hit rate; context budgets |
| Memory | Scoped stores, TTL/forgetting, governed writes, measured cross-session uplift |
| Tools | Typed schemas, read/write separation, least-privilege scopes, call logging |
| Intelligence | Task-to-model routing measured by cost-of-pass, with independent verification |
| Execution | Durable checkpoints, idempotent side effects, retries, timeouts, cost caps |
| Governance | Approval gates, audit logs, data boundaries, threat-model-based controls |
| Evaluation | Golden dataset, repeated-trial benchmarks with variance, online monitoring |

See the [white paper](../whitepaper.md) for the reasoning and citations behind each pillar.
