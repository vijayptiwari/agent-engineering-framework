# Pre-Deployment Design Checklist

Walk this before shipping or significantly changing a production agent. Organized by the [six-phase lifecycle](../whitepaper.md#8a-from-framework-to-development-lifecycle): **specify before you prompt; evaluate before you deploy, and keep evaluating after.**

## Phase 1 — Specify (Goal, Governance)

- [ ] Exact success criteria are written down (not "help the user")
- [ ] Stopping conditions defined (success met, max steps, or escalate)
- [ ] Trade-off priorities ranked (accuracy vs cost vs latency vs safety)
- [ ] Risk thresholds and approval gates identified for high-stakes actions
- [ ] Decision rights clear: what the agent may do autonomously vs with approval

## Phase 2 — Design (Trajectory, Context, Tools, Intelligence)

- [ ] Workflow/trajectory designed, including failure branches and retries
- [ ] A validation/verification step exists before final output
- [ ] Context strategy: what is retrieved, excluded, and compressed at each step
- [ ] Tools have typed schemas; read and write tools are separated
- [ ] Tool permissions follow least privilege; side effects are idempotent
- [ ] Task-to-model mapping defined; cost-of-pass considered, not just capability

## Phase 3 — Build (Prompt, Execution, Memory)

- [ ] Prompts versioned; output schema enforced; prohibited claims specified
- [ ] Orchestration has checkpoints after major steps
- [ ] Retries, timeouts, and cost caps configured
- [ ] State survives transient failures (resume, not restart)
- [ ] Memory: what persists, what expires, how writes are governed

## Phase 4 — Evaluate (gate before deploy)

- [ ] Golden dataset exists for the task
- [ ] Offline benchmark run with **repeated trials and variance reported**
- [ ] Dual evaluation: both end outcomes and trajectory quality scored
- [ ] Policy-compliance / side-effect scoring in place (not just task success)
- [ ] Regression suite passes versus the previous version

## Phase 5 — Deploy (Execution, Governance)

- [ ] Staged rollout (canary or shadow) before full traffic
- [ ] Approval gates active for risky/destructive/production-touching actions
- [ ] Audit logging captures every action and decision
- [ ] Sensitive data redacted before model calls; secrets never enter context
- [ ] Cost and latency budgets enforced with alerts

## Phase 6 — Operate & improve (Evaluation, Memory, Context)

- [ ] Online monitoring tracks success rate, cost, latency, override rate
- [ ] Human-feedback capture feeds back into the golden dataset
- [ ] Drift detection: scheduled re-benchmarking, not one-time validation
- [ ] Memory curated over time (stale entries expired, poisoning checked)

## Security cross-check (Governance, all phases)

- [ ] Every retrieved document and tool result treated as untrusted input
- [ ] Prompt-injection and memory-poisoning defenses in place
- [ ] Tool calls sandboxed where they touch real systems
- [ ] Approvals are trace-linked and reviewable

If you cannot tick the **Evaluate**, **Deploy**, and **Operate** boxes, you have a promising prototype — not a production system.
