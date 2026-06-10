# The AI Agent Engineering and Development Framework

## Beyond Prompt Engineering: The 10 Pillars of Production Agent Engineering

**A practitioner framework for designing, building, and operating reliable AI agents beyond demos**

**Author:** Vijay Prakash Tiwari

**Version:** 1.2 (publication-ready)

**Date:** June 2026

---

## Abstract

Large language models (LLMs) are increasingly deployed as agents: systems that plan, call tools, retrieve knowledge, remember prior interactions, and act across multi-step workflows [2][14][15]. Yet much industry discussion still centres on the model and the prompt. This paper argues that prompt quality is a necessary but insufficient condition for reliable agents, and that production agents are better understood as a systems-engineering problem. Drawing on published research in agent reasoning [1][2][3][4], tool use [5], retrieval and context management [6][16], memory architectures [7][8], model routing [17][18], agent evaluation [9][10][11][12], and industry guidance from Anthropic [19][20], OpenAI [21], and Google [22], this paper synthesizes ten coordinated engineering disciplines — Prompt, Goal, Trajectory, Context, Memory, Tool, Intelligence, Execution, Governance, and Evaluation Engineering — into a single practical framework. Three worked case studies (telecom incident root-cause analysis, customer billing disputes, and automated software maintenance) illustrate how the pillars map onto real systems and how the absence of any single pillar produces characteristic failure modes. The paper also maps the pillars onto a six-phase development lifecycle — specify, design, build, evaluate, deploy, operate — and proposes a dual-evaluation principle: production agents must be graded on both end outcomes and trajectory quality [29][30].

**Positioning statement.** This paper does not claim that any individual discipline — prompting, retrieval, memory, tool use, routing, governance, or evaluation — is newly invented here. Each has an established research and practice literature, cited throughout. The contribution is the synthesis: a coordinated, teachable, production-oriented framework, in the spirit of practitioner frameworks such as the Twelve-Factor App methodology and its agent-era successors [24].

---

## 1. Introduction: Why Prompt Engineering Is Not Enough

The first wave of LLM adoption asked a narrow question: *how do we phrase better instructions?* Techniques such as chain-of-thought prompting demonstrated that instruction design measurably changes model reasoning quality [1], and "prompt engineering" became a recognized skill.

The second wave recognized that even excellent instructions fail when the model lacks relevant information. Retrieval-augmented generation grounded model outputs in external knowledge [6], and a broader discipline of *context engineering* — the systematic design of the entire information payload a model sees at inference time — has since been formalized in both academic surveys [16] and industry practice [20][27].

The third wave is agentic. Modern systems plan multi-step tasks, invoke external tools [2][5], coordinate multiple models [13], and operate semi-autonomously inside business workflows [21][22]. Surveys of LLM-based agents document an explosion of architectures combining planning, memory, tool use, and reflection [14][15].

At this point the engineering question changes. It is no longer *"what should the prompt say?"* but:

> **How should an intelligent system be engineered so that it reliably achieves a goal in the real world?**

Industry guidance converges on the same conclusion. Anthropic advises teams to build agents from simple, composable patterns — prompt chaining, routing, parallelization, orchestrator–workers, evaluator–optimizer — and to add autonomy only when it demonstrably improves outcomes [19]. OpenAI's guide to building agents stresses model selection, well-defined tools, structured instructions, incremental orchestration, and guardrails with human-in-the-loop intervention [21]. Google's agents whitepaper describes a cognitive architecture in which a model, an orchestration layer, and tools must be designed together [22].

What is missing is a single integrative vocabulary that practitioners, architects, and managers can share. This paper proposes one.

### 1.1 The central claim

A production-grade agent is not:

```text
Prompt + LLM = Agent
```

A more realistic decomposition is:

```text
Clear Goal
+ Designed Trajectory
+ Right Context
+ Useful Memory
+ Proper Tools
+ Correct Intelligence
+ Reliable Execution
+ Governance
+ Evaluation
= Production-grade Agent
```

Each line corresponds to an engineering discipline with its own design questions, failure modes, and measurable outcomes. We call the combined practice **Agent Engineering** and organize it into ten pillars.

---

## 2. Foundational Distinction: Goal, Plan, and Trajectory

Before presenting the pillars, one distinction deserves emphasis because it underlies several of them.

**A goal is the desired end state.** "Identify the root cause of the outage with supporting evidence." The goal defines what success looks like, when to stop, and what trade-offs are acceptable.

**A plan is the intended path before execution.** "Collect logs, analyze metrics, check deployments, write the RCA." Planning in LLM agents has been studied extensively — from interleaved reasoning-and-acting in ReAct [2] to deliberate search over reasoning branches in Tree of Thoughts [3].

**A trajectory is what actually happens during execution** — the realized sequence of model calls, tool invocations, observations, retries, and corrections. The term is standard in the agent-evaluation literature: benchmarks such as AgentBench [9] and τ-bench [12] judge agents not only on final answers but on the interaction trajectory that produced them, and reflection methods such as Reflexion explicitly feed trajectory feedback back into the agent [4].

The distinction matters because two agents with the same goal and even the same plan can realize very different trajectories:

```text
Agent A: read one log → guess → answer
Agent B: collect logs → collect metrics → check deployments
         → form hypotheses → validate against evidence → answer
```

Same goal. Different trajectory. Different reliability. Plans are intentions; trajectories are reality — and production engineering is largely the discipline of keeping reality close to intention, or recovering gracefully when it diverges.

### 2.1 Two trajectory layers in production systems

In production agent systems, the trajectory concept usefully splits into two layers that must be engineered together:

- **Cognitive trajectory** — the evolving reasoning–action–observation trace of the model itself: thoughts, tool calls, observations, corrections, and the final answer [2][4].
- **Workflow trajectory** — the concrete executed path through the orchestrated system: graph nodes visited, branches taken, retries, delegations, and checkpoints [23].

The distinction matters operationally. Cognitive-trajectory failures (hallucinated tool arguments, premature conclusions) are addressed with prompting, validation loops, and reflection [4]; workflow-trajectory failures (lost state, unbounded loops, unrecoverable errors) are addressed with orchestration, checkpointing, and stopping conditions [19][23][24]. Mature engineering disciplines outside software — robotics and autonomous-vehicle control — have long treated trajectory quality as a first-class, measurable engineering object; LLM-agent practice is only now catching up, which is precisely why it deserves a named pillar.

---

## 3. The Ten Pillars

```text
Agent Engineering
├── 1.  Prompt Engineering        How do we instruct the agent?
├── 2.  Goal Engineering          What should the agent achieve?
├── 3.  Trajectory Engineering    How should the agent move toward the goal?
├── 4.  Context Engineering       What should the agent see right now?
├── 5.  Memory Engineering        What should persist over time?
├── 6.  Tool Engineering          What actions can the agent perform?
├── 7.  Intelligence Engineering  Which model should do which work?
├── 8.  Execution Engineering     How does the agent run reliably?
├── 9.  Governance Engineering    How do we keep the agent safe and compliant?
└── 10. Evaluation Engineering    How do we know the agent is good?
```

Architecturally, the ten pillars describe a **deterministic envelope around a probabilistic core**. The model — the probabilistic core — supplies reasoning. Everything else is deterministic engineering wrapped around it: the goal contract, policy checks, the context builder, the trajectory controller, tool schemas, memory-write rules, the runtime state machine, and the evaluation loop. Production reliability comes from the envelope, not from hoping the core behaves. Each pillar below is therefore presented as a production *control surface*: a definition, the question it answers, design guidance, the metrics that make it observable, and the failure mode when it is absent.

### 3.1 Pillar 1 — Prompt Engineering

**Definition.** The design of instructions, role descriptions, constraints, examples, and output formats given to the model. Structured prompting techniques such as chain-of-thought demonstrably improve multi-step reasoning [1], and few-shot exemplars, explicit schemas, and behavioral constraints remain the cheapest reliability lever available [19][21].

**Example (incident-analysis agent):**

```text
You are an incident analysis assistant.
Use only the evidence provided in context.
Never assert a root cause without citing supporting evidence.
Return: structured RCA, confidence score, and evidence list.
```

**Key metrics.** First-pass schema-valid output rate; prompt-token overhead; evaluation pass rate; regression rate after prompt changes.

**Failure mode if absent.** The agent produces an unstructured essay, ignores format requirements, or asserts unsupported claims. OpenAI's guidance treats clear, structured instructions as one of the three foundations of agent design, alongside models and tools [21].

### 3.2 Pillar 2 — Goal Engineering

**Definition.** The conversion of a business intention into a **machine-actionable objective contract**: the desired end state, acceptance criteria, stopping conditions, refusal policies, escalation triggers, and decision rights. Anthropic notes that production agents commonly need explicit stopping conditions (such as maximum iterations) to retain control [19]; benchmarks like τ-bench show that agents frequently fail precisely on goal adherence — following domain rules consistently to task completion — rather than on raw capability [12].

**Example:**

```text
Goal: identify the most likely root cause of the outage.
Success: confidence ≥ 0.8, ≥ 3 pieces of corroborating evidence,
         no unsupported claims.
Stop:    when success criteria are met, or after 15 tool calls,
         or escalate to a human if confidence < 0.8.
```

**Key metrics.** Completion rate against acceptance criteria; false-completion rate; escalation precision; human-override rate.

**Failure mode if absent.** The agent loops — searching, finding more data, searching again — and never terminates, or terminates arbitrarily. A vague goal ("help the user") gives the system no basis for deciding that it is done.

### 3.3 Pillar 3 — Trajectory Engineering

**Definition.** The design, control, and optimization of the path the agent takes: task decomposition, tool sequencing, branching logic, retry and recovery paths, reflection loops, and validation steps.

This pillar draws directly on the agent-reasoning literature. ReAct interleaves reasoning traces with actions so each step is grounded in observations [2]. Tree of Thoughts structures exploration over alternative reasoning branches [3]. Reflexion adds verbal self-feedback over failed trajectories so subsequent attempts improve [4]. Anthropic's workflow patterns — chaining, routing, parallelization, orchestrator–workers, evaluator–optimizer — are precisely trajectory design patterns [19].

**Example (telecom RCA trajectory):**

```text
Alert intake → classify incident → look up service dependencies
→ retrieve logs → analyze metrics → correlate with deployments
→ generate hypotheses → validate against evidence
→ draft RCA → human approval if confidence is low
```

**Key metrics.** Steps-to-completion; replanning frequency; loop-abort rate; tool-success ratio; p95 task latency.

**Failure mode if absent.** The agent uses tools in arbitrary order, skips validation, jumps to conclusions, or cannot recover when a tool fails. The goal was right; the journey was wrong.

### 3.4 Pillar 4 — Context Engineering

**Definition.** Deciding what information the agent sees at each step: retrieval, selection, compression, isolation, and context-window management. Mei et al. formalize context engineering as the systematic optimization of the model's full inference-time information payload, spanning retrieval, processing, and management of context [16]. Industry guidance emphasizes treating context as a finite resource to be curated, not a bucket to be filled [20][27]. Retrieval-augmented generation remains the canonical mechanism for grounding agents in external knowledge [6].

**Example.** For an incident at 02:00: the current alert, the affected service topology, the last two hours of logs, the last 24 hours of metrics, recent deployments, and known incident patterns — and nothing else.

**Key metrics.** Retrieval hit rate; evidence-citation rate; stale-context rate; tokens per successful run; hallucination rate.

**Failure mode if absent.** Two symmetrical failures occur. *Starvation:* the agent never sees that a release shipped ten minutes before the outage, and misses the cause entirely. *Overload:* irrelevant context dilutes attention and degrades reasoning — a phenomenon the context-engineering literature treats as a first-class design constraint [16][20]. A strong model with poor context is a confident guesser.

### 3.5 Pillar 5 — Memory Engineering

**Definition.** What the agent remembers beyond the current context window, and how that memory is stored, retrieved, summarized, updated, and expired. MemGPT introduced hierarchical memory management — paging information between a bounded context window and external storage, in analogy to operating-system virtual memory [7]. Park et al.'s generative agents demonstrated that long-horizon believable behavior requires recording, synthesizing, and retrieving experience streams [8].

A useful working split:

```text
Context = what the agent sees right now
Memory  = what the agent can recall later

Short-term : current session state
Long-term  : facts that survive across sessions
Episodic   : past incidents, conversations, actions
Semantic   : distilled patterns, preferences, domain knowledge
```

**Key metrics.** Memory hit rate; stale-memory rate; cross-session completion uplift; memory-poisoning incidence [32].

**Failure mode if absent.** Every task starts from zero. The same outage pattern recurs and the agent repeats last month's investigation, at full cost, with no benefit from prior experience. The converse risk also exists: ungoverned memory *writes* enlarge the attack surface — memory poisoning is now a named threat category in agentic-security guidance [32].

### 3.6 Pillar 6 — Tool Engineering

**Definition.** What external capabilities the agent can invoke and how those capabilities are specified. Toolformer showed that models can learn when and how to call external APIs [5]; Google's whitepaper organizes agent tooling into extensions, functions, and data stores that bridge models to external systems [22]; Anthropic emphasizes the agent–computer interface — tool naming, documentation, and ergonomics — as a primary determinant of agent reliability [19]; OpenAI recommends well-defined, narrowly scoped tools as a design foundation [21].

Practical tool-design rules recur across sources: one job per tool; typed input and output schemas; separate read tools from write tools so confirmation flows stay tractable; least-privilege scopes per tool; idempotent side effects [19][21]. The Model Context Protocol (MCP) standardizes this layer across vendors as an open interface for exposing tools and data sources to agents [31].

**Example (RCA agent toolset):** log search (OpenSearch), metrics (Prometheus), cluster state (Kubernetes API), release history (deployment system), ticketing, and a known-errors knowledge base.

**Key metrics.** Tool-selection accuracy; schema-failure rate; API success rate; side-effect incident rate.

**Failure mode if absent.** The agent suspects a pod restart, has no cluster access, cannot verify, and produces a hedge instead of an answer. Reasoning without tools cannot become action.

### 3.7 Pillar 7 — Intelligence Engineering

**Definition.** Selecting, routing, and combining the right model for each sub-task: small models for classification and routing, mid-sized models for summarization and extraction, frontier reasoning models for causal analysis and planning, specialist models for code or vision, and independent verifier models for review.

This is an active research area. RouteLLM trains routers on human-preference data to choose between strong and weak models per query, cutting cost by more than half without measurable quality loss [17]. FrugalGPT demonstrates cascades that invoke progressively stronger models only when cheaper ones are insufficient [18]. Multi-agent frameworks such as AutoGen make heterogeneous model composition a first-class pattern [13]. OpenAI's practical guidance is the same: not every task needs the smartest model; route by task difficulty and iterate [21]. The most useful planning metric here is **cost-of-pass** — the expected cost of obtaining a *correct* result — rather than raw capability: a 2025 study of agent-framework complexity found that a deliberately right-sized framework retained 96.7% of a leading open-source agent's GAIA performance while improving cost-of-pass by 28.4%, and that added architectural complexity frequently yields diminishing returns [35].

**Example:**

```text
Intent classification  → small model
Routing / delegation   → small model
Planning and analysis  → frontier reasoning model
Tool execution         → deterministic code and APIs
Verification           → independent evaluator model
Response formatting    → small model
```

**Key metrics.** Cost-of-pass [35]; success per dollar; route accuracy; escalation-to-stronger-model rate; p95 latency per route.

**Failure modes if absent.** *Overpowered:* every step uses the most expensive model — high cost, high latency, poor scalability. *Underpowered:* a cheap model performs deep causal reasoning and misses the chain entirely. The principle: agent design is not choosing one model; it is orchestrating the right intelligence for the right task.

### 3.8 Pillar 8 — Execution Engineering

**Definition.** Making the agent run reliably in production: orchestration, state management, retries, timeouts, checkpoints, streaming, parallelization, cost controls, and durable workflows. Orchestration frameworks such as LangGraph model agents as stateful graphs with persistence and checkpointing [23]; the 12-Factor Agents principles argue for owning control flow, treating agents as ordinary software with explicit state, and pausing/resuming via standard engineering mechanisms [24].

**Example:** an RCA workflow as a graph — classify → retrieve logs → retrieve metrics → analyze → validate → generate RCA — with a conditional edge to human approval when confidence is low and a checkpoint after every major step.

**Key metrics.** Run completion rate; retry recovery rate; p95 latency; timeout rate; duplicate-side-effect incidents.

**Failure mode if absent.** A tool call fails mid-investigation; with no retry and no checkpoint, the entire investigation is lost. The agent that worked in the demo dies on the first transient network error in production.

### 3.9 Pillar 9 — Governance Engineering

**Definition.** The control layer: permissions, human approvals, audit logs, security boundaries, data-access controls, policy enforcement, escalation rules, and risk thresholds. OpenAI's guide devotes an entire section to layered guardrails and planning for human intervention on high-stakes actions [21]. Regulatory and standards frameworks make this non-optional for many deployments: the NIST AI Risk Management Framework defines govern/map/measure/manage functions for trustworthy AI [25], its Generative AI Profile maps GAI-specific risks to over two hundred suggested actions [33], and the EU AI Act imposes binding obligations — risk management, logging, human oversight — on high-risk AI systems [26].

Governance also has an adversarial dimension that traditional application security does not cover. The OWASP Agentic Security Initiative catalogues threat categories specific to agents — prompt injection through retrieved content, memory poisoning, tool misuse, and unauthorized side effects among them — and recommends defense in depth: task-scoped tool permissions, sandboxed execution, governed memory writes, and trace-linked approvals [32]. Two design implications follow. First, every retrieved document and tool result is untrusted input. Second, confidence-based escalation to a human reviewer is a governance control, not a UX nicety.

**Example policy set:**

```text
confidence < 0.8            → human approval required
action touches production   → human approval required
data classified sensitive   → redact before model call
refund > ₹5,000             → manager approval
destructive operation       → blocked by default
```

**Key metrics.** Policy-violation rate; unsafe-action rate; approval burden; audit-log completeness.

**Failure mode if absent.** The agent misdiagnoses a database issue and autonomously restarts the production database. The outage worsens. The agent is no longer solving the incident; it *is* the incident. The risk is not hypothetical: when web agents are scored against explicit organizational policies, policy-compliant completion falls to under two-thirds of nominal completion — meaning a substantial fraction of "successful" agent runs violate at least one policy on the way to the goal [29].

### 3.10 Pillar 10 — Evaluation Engineering

**Definition.** Measuring whether the agent is actually good: task success rate, accuracy, cost, latency, hallucination rate, tool-use correctness, trajectory quality, human-override rate, regression testing, and online monitoring. The benchmark literature is rich: AgentBench evaluates LLMs-as-agents across eight interactive environments [9]; GAIA tests grounded multi-step assistance [10]; SWE-bench measures whether systems resolve real GitHub issues [11]; τ-bench evaluates tool-using agents in realistic user–agent dialogues with domain rules, and finds even strong agents succeed on fewer than half of airline-domain tasks and degrade sharply on consistency across retries [12]; WebArena demonstrates the capability gap starkly — humans complete 78% of realistic web tasks where the best contemporary agent baseline managed 14% [28]. Production teams additionally need *online* evaluation: golden datasets, regression suites, and human-feedback loops [19][21].

**The dual-evaluation principle.** Grade every agent on *both* end outcomes and trajectory quality. Outcome-only scoring is provably misleading in two directions. First, rule-based success checks systematically under-report genuine success: AgentRewardBench, with 1,302 expert-annotated agent trajectories, found that the rule-based evaluation used by common benchmarks undercounts successful runs, and that no single LLM judge excels across benchmarks [30]. Second, outcome-only scoring over-credits unsafe success: ST-WebAgentBench pairs web tasks with organizational policies and scores *Completion under Policy* — completions that violate no applicable policy — and finds state-of-the-art agents' policy-compliant completion rate is less than two-thirds of their nominal completion rate [29]. An agent that "completes the task" by violating policy has not completed the task.

**A caution on interpretability.** Because LLM-agent traces are text, teams often assume they are self-explaining. They are not: a visible trace is not the same as a causal explanation of agent behavior, and LLM judges of trajectories remain unreliable enough to require expert calibration [30]. Treat trace review as evidence-gathering, not as proof.

**A caution on reproducibility.** Agent comparisons are noisy. A systematic study of agent-framework design choices found that, absent a standard evaluation protocol, published agent results — including open-source ones — are frequently non-reproducible, with significant variance across random runs [34]. Production evaluation should therefore use repeated trials and report variance, not single-run scores.

**Key metrics.** Benchmark pass rate with inter-run variance [34]; regression delta per release; human-label agreement; business-KPI lift; time-to-detect regressions.

**Example metric set (RCA agent):** RCA accuracy versus post-incident review, mean time to diagnosis, false-positive rate, evidence quality, human-override rate, cost per investigation, tool-failure rate.

**Failure mode if absent.** Someone "improves" a prompt; the output sounds better; accuracy silently drops; nobody notices for a quarter. The most dangerous agent is not the one that fails loudly — it is the one that sounds correct while drifting away from correctness.

---

## 4. A Teaching Mnemonic

The pillar initials — P G T C M T I E G E — yield a memorable sentence:

> **Pirates Grab Treasure, Carry Maps, Tools, Inspect Every Golden Egg**

```text
Pirates → Prompt      Carry → Context      Inspect → Intelligence
Grab    → Goal        Maps  → Memory       Every   → Execution
Treasure→ Trajectory  Tools → Tools        Golden  → Governance
                                           Egg     → Evaluation
```

An agent is like a pirate crew: it receives an order (prompt), chases treasure (goal), follows a journey (trajectory), reads island clues (context), keeps a map (memory), carries shovels (tools), assigns the right crew member to each job (intelligence), sails and digs (execution), follows the pirate code (governance), and bites the coin to check it is real (evaluation).

---

## 5. Case Study 1: Telecom Incident RCA Agent

### 5.1 Scenario

```text
Service:  Payment Gateway
Symptom:  API error rate > 20%
Time:     02:00
Impact:   customers cannot complete payments
```

The agent must investigate and produce a root-cause analysis with evidence.

### 5.2 Pillar mapping

| Pillar | Implementation |
|---|---|
| Prompt | Evidence-only instructions; structured RCA output schema |
| Goal | Root cause with ≥3 pieces of evidence and confidence ≥ 0.8 |
| Trajectory | Logs → metrics → deployments → hypotheses → validation → RCA [2][4] |
| Context | Alert, topology, 2h logs, 24h metrics, recent deploys, known patterns [16] |
| Memory | Past incidents for this service; recurring patterns; known fixes [7] |
| Tools | OpenSearch, Prometheus, Kubernetes API, deploy history, ticketing [22] |
| Intelligence | Small model classifies; frontier model reasons; verifier reviews [17] |
| Execution | Graph workflow with retries, timeouts, checkpoints [23] |
| Governance | Human approval below confidence threshold or for production actions [21][25] |
| Evaluation | RCA accuracy, time-to-diagnosis, override rate, cost per investigation [9] |

### 5.3 A realistic trajectory (plan vs reality)

Plan: collect logs → analyze metrics → check deployments → write RCA.

Reality:

```text
Collect logs → logs incomplete → retry against secondary index
→ fetch metrics → DB connection-pool spike found
→ check deployments → none in window
→ check DB configuration → connection limit lowered by config change
→ validate: config change timestamp matches error onset
→ RCA: confidence 0.86 → auto-file ticket → notify on-call
```

The plan survived two contacts with reality only because retry paths and branching were designed in advance (Pillar 3) and state survived the retries (Pillar 8).

### 5.4 Ablation: remove one pillar

| Missing pillar | Observed behavior |
|---|---|
| Prompt | Unstructured essay; unsupported claims |
| Goal | Investigation never terminates |
| Trajectory | Reads one log, guesses "database issue," wrong RCA |
| Context | Misses the config change entirely |
| Memory | Re-investigates a known recurring pattern from scratch |
| Tools | Cannot verify cluster state; produces hedged non-answer |
| Intelligence | Frontier model burns budget on log formatting, or a small model misses the causal chain |
| Execution | Transient tool failure destroys 20 minutes of investigation |
| Governance | Agent restarts the production DB at 02:00 unsupervised |
| Evaluation | RCA quality drifts for months without detection |

---

## 6. Case Study 2: Customer Billing-Dispute Agent

### 6.1 Scenario

A subscriber disputes a ₹1,499 charge. The agent must resolve the dispute within policy, with evidence, and without unauthorized refunds. This is exactly the class of rule-governed, tool-using dialogue task that τ-bench shows remains hard for current agents — particularly *consistent* rule-following across many interactions [12].

### 6.2 Design

- **Goal:** resolve the dispute with a policy-compliant outcome; success = customer informed, action taken or escalated, full audit trail; stop when resolution is recorded or escalation triggered.
- **Trajectory:** authenticate → retrieve billing history → classify dispute type → check policy eligibility → compute remedy → (auto-apply if within limits | escalate if not) → confirm with customer → log.
- **Context:** this customer's invoices and plan, the dispute policy version in force on the charge date, prior disputes by this customer.
- **Memory:** episodic record of past disputes and outcomes; semantic pattern "double-charge spikes after plan migrations."
- **Tools:** CRM, billing API (read), refund API (write, capped), policy knowledge base, ticketing.
- **Intelligence:** small model for intent and dispute-type classification; mid model for explanation drafting; verifier model checks the proposed remedy against policy before any write [17][19].
- **Execution:** idempotent refund calls with dedupe keys; checkpoint before any write; timeout-and-resume on billing-API latency.
- **Governance:** refunds above ₹5,000 require manager approval; PII redacted before model calls; every action logged [21][26].
- **Evaluation:** resolution rate, policy-violation rate (target: zero), customer-satisfaction delta, escalation precision, cost per resolution.

### 6.3 Characteristic failure without the framework

A prompt-only bot reads the complaint, apologizes fluently, and promises a refund it has no authority to issue — or issues one twice because the refund call timed out and was retried without idempotency. The first is a governance failure; the second is an execution failure. Neither is a prompt failure, and no prompt fix would have prevented them.

---

## 7. Case Study 3: Software-Maintenance Agent

### 7.1 Scenario

An agent receives a failing-test bug report in a large repository and must produce a candidate fix as a pull request. SWE-bench established this as a measurable task: resolving real GitHub issues against real codebases [11].

### 7.2 Design

- **Goal:** a minimal patch that makes the failing test pass without breaking the existing suite; stop after N candidate patches or escalate with a diagnosis.
- **Trajectory:** reproduce failure → localize fault (search, read, trace) → form hypothesis → draft patch → run tests → if red, reflect and revise [4] → if green, run full suite → open PR with explanation.
- **Context:** failing test output, the relevant source files (not the whole repository), recent commits touching those files, project conventions [16][20].
- **Memory:** past fixes in this codebase; module-specific gotchas; flaky-test list.
- **Tools:** code search, file read/write, test runner, git, CI status.
- **Intelligence:** code-specialist model for patch synthesis; cheaper model for search-result triage; independent model reviews the diff before PR creation [13][17].
- **Execution:** sandboxed runs; checkpoint per iteration; hard budget on test-runner minutes.
- **Governance:** the agent may open PRs but never merge; no pushes to protected branches; secrets never enter context.
- **Evaluation:** resolved-issue rate (SWE-bench-style), regression rate, review-acceptance rate, cost per resolved issue [11].

### 7.3 Lesson

The highest-leverage pillar here is Trajectory: reproduce-before-fix and test-before-PR loops dominate outcome quality. The second is Evaluation: without a regression suite as ground truth, "plausible-looking diff" substitutes for "correct fix" — the exact hallucination-shaped failure that evaluation engineering exists to catch.

---

## 8. Cross-Cutting Failure Modes

| # | Failure | Symptom | Primary pillar |
|---|---|---|---|
| 1 | Vague goal | Agent never stops, or stops arbitrarily | Goal |
| 2 | Search → answer | No validation; confident errors | Trajectory |
| 3 | Context overload | Reasoning degrades as context grows | Context [16][20] |
| 4 | Context starvation | The decisive fact never reaches the model | Context |
| 5 | Goldfish agent | Repeats past work every session | Memory [7] |
| 6 | Tool roulette | Random or malformed tool calls | Tools [19][22] |
| 7 | One-model-fits-all | Cost blowups or shallow reasoning | Intelligence [17][18] |
| 8 | Demo-only agent | Dies on first transient failure | Execution [23][24] |
| 9 | Unsupervised actuator | Takes destructive actions confidently | Governance [21][25][26] |
| 10 | Silent drift | Quality decays without detection | Evaluation [9][12] |

A practical observation: failures 8–10 are the ones that end agent programs inside organizations. Demos fail on 1–7; *production trust* fails on 8–10.

---

## 8A. From Framework to Development Lifecycle

The ten pillars are not only an architecture checklist; they map onto a development lifecycle. Industry guidance consistently recommends incremental construction — start with the simplest system that could work, evaluate, and add complexity only when it demonstrably improves outcomes [19][21]. The pillars sequence naturally across that lifecycle:

| Phase | Primary pillars | Key activities |
|---|---|---|
| 1. Specify | Goal, Governance | Define success criteria, stopping conditions, risk thresholds, approval gates — *before* writing a prompt |
| 2. Design | Trajectory, Context, Tools, Intelligence | Design the workflow, the context strategy, typed tool schemas, and the task-to-model mapping |
| 3. Build | Prompt, Execution, Memory | Implement instructions and output schemas; build the orchestration graph with checkpoints, retries, timeouts; wire memory stores |
| 4. Evaluate | Evaluation | Golden dataset, offline benchmark, dual outcome-and-trajectory scoring [29][30], regression suite |
| 5. Deploy | Execution, Governance | Staged rollout behind approval gates; audit logging; cost and latency budgets |
| 6. Operate & improve | Evaluation, Memory, Context | Online monitoring, human-feedback capture, memory curation, periodic re-benchmarking |

Two practices deserve emphasis. **Specify before you prompt:** teams that begin at phase 3 (the prompt) inherit vague goals and absent governance, the two failure modes that are hardest to retrofit. **Evaluate before you deploy, and keep evaluating after:** phase 4 is a gate, but phase 6 is a loop — quality drift is detected only by systems that keep measuring [9][12].

## 8B. Shipping Changes: CI/CD for Agents as Continuous Contract Validation

Agent delivery is not only code shipping. A production agent release is the coordinated release of **prompts, tool schemas, routing policies, memory-write rules, evaluation thresholds, and governance controls** — and a regression can enter through any of them. A changed tool description can silently alter tool selection; a new model route can shift output style past a guardrail; a relaxed memory rule can admit poisoned writes [32]. CI/CD for agents is therefore best understood as *continuous contract validation*: every release re-proves that the goal contract, tool contracts, and policy contracts still hold.

A practical pipeline:

```text
Change proposal (prompt, tool schema, model route, policy, memory rule, or code)
→ Static checks         schemas, types, prompt linting
→ Unit / simulation     tool mocks, routing rules, policy tests
→ Offline evaluation    golden datasets, benchmarks, adversarial tests
   ├─ fail → diagnose from traces and failure clusters → revise
→ Trace replay          re-run recorded production trajectories in a sandbox
→ Canary / shadow       small-percentage or mirrored traffic
   ├─ SLO, cost, or policy breach → roll back and diagnose
→ Progressive rollout   with continuous tracing and human-feedback capture
→ Dataset refresh       failures become new golden cases
```

Two evidence-based rules govern the gates. First, because agent runs exhibit significant inter-run variance, offline gates must use **repeated trials with variance reporting** — a single green run proves little [34]. Second, gates should track **cost-of-pass alongside accuracy**, since a "better" release that doubles the cost of a correct result may be a net regression in production [35].

---

## 9. Production Readiness Rubric

Score each pillar 0–5:

| Score | Meaning |
|---|---|
| 0 | Not considered |
| 1 | Handled informally |
| 2 | Partially designed |
| 3 | Designed and documented |
| 4 | Tested and monitored |
| 5 | Production-grade with feedback loops |

Interpretation guide: a total below 30/50, or any single pillar at 0–1 among Execution, Governance, and Evaluation, indicates a prototype — regardless of how impressive the demo is.

### Design checklist (abbreviated)

- **Goal:** exact success criteria? stopping conditions? trade-off priorities?
- **Prompt:** output schema? prohibited claims? role boundaries?
- **Trajectory:** step design? failure branches? validation loop?
- **Context:** what is retrieved, excluded, compressed? [16]
- **Memory:** what persists, what expires, how is it retrieved? [7]
- **Tools:** typed schemas? permissions? call logging? [22]
- **Intelligence:** task-to-model mapping? independent verification? [17]
- **Execution:** checkpoints? retries? timeouts? cost caps? [23][24]
- **Governance:** approval gates? audit trail? data boundaries? [25][26]
- **Evaluation:** golden dataset? regression suite? online monitoring? [9][12]

---

## 10. Related Work and How This Framework Differs

**Composable agent patterns (Anthropic).** *Building Effective Agents* catalogues workflow patterns and counsels simplicity [19]. Those patterns live primarily inside Pillars 3 (Trajectory) and 7–8 (Intelligence, Execution); this framework adds the surrounding system concerns — goals, memory, governance, evaluation — as co-equal disciplines.

**Practical agent guidance (OpenAI).** The *Practical Guide to Building Agents* covers model selection, tools, instructions, orchestration, and guardrails [21]. It is organized as a build sequence; this paper reorganizes equivalent and additional concerns as ten independently assessable disciplines suitable for audits and readiness scoring.

**Cognitive architectures (Google).** The *Agents* whitepaper's model–orchestration–tools triad [22] corresponds to Pillars 7, 3/8, and 6 respectively; the present framework extends the triad with explicit goal, context, memory, governance, and evaluation layers.

**12-Factor Agents.** A principles list for production agent software [24], strongest on what this paper calls Execution Engineering. The ten-pillar framing is broader (governance, evaluation, intelligence routing) and is designed to be scoreable.

**Academic surveys.** Agent surveys [14][15] and the context-engineering survey [16] map the research landscape; this paper is deliberately a *practitioner* synthesis of that landscape rather than new taxonomy of the literature.

---

## 11. Limitations and Future Work

This framework is a synthesis, not an empirical result. It has not yet been validated by controlled comparison (e.g., scoring teams' agents on the rubric and correlating with production incident rates) — that is the natural next study. Pillar boundaries are also pragmatic rather than ontological: context and memory blur at the retrieval layer, and trajectory and execution blur at the orchestrator. Finally, the framework targets single- and few-agent systems; large multi-agent economies [13][14] may require additional coordination pillars.

Three research directions follow naturally from the framework. First, **pillar ablations as experiments**: implement a reference agent and measure how removing context curation, memory, governance gates, or execution durability changes task success, cost, side effects, and override rates on public benchmarks [9][28] — turning the pillars from a taxonomy into a testable design language. Second, **trajectory-aware evaluation**: trajectories should become scorable and trainable objects rather than debugging artifacts, with calibrated judges for success, side effects, and policy compliance [29][30]. Third, **a common trajectory schema**: a shared specification for goals, observations, tool actions, state updates, safety constraints, and evaluation hooks, so that traces are portable across orchestrators and comparable across teams.

Planned v1.3 additions: a LangGraph reference implementation, two further case studies (insurance claims; sales operations), and a comparison matrix against emerging agent-governance standards [25][26].

---

## 12. Conclusion

The future of AI agents will not be determined only by better models, but by better engineering around goals, trajectories, context, memory, tools, intelligence, execution, governance, and evaluation. Prompt engineering remains necessary. It was never sufficient. The teams that internalize this shift will build agents that are not merely impressive in demos but reliable in production — and reliability, not eloquence, is what earns an agent a permanent job.

---

## References

[1] J. Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models," *NeurIPS 2022*. arXiv:2201.11903. https://arxiv.org/abs/2201.11903

[2] S. Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models," *ICLR 2023*. arXiv:2210.03629. https://arxiv.org/abs/2210.03629

[3] S. Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models," *NeurIPS 2023*. arXiv:2305.10601. https://arxiv.org/abs/2305.10601

[4] N. Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning," *NeurIPS 2023*. arXiv:2303.11366. https://arxiv.org/abs/2303.11366

[5] T. Schick et al., "Toolformer: Language Models Can Teach Themselves to Use Tools," *NeurIPS 2023*. arXiv:2302.04761. https://arxiv.org/abs/2302.04761

[6] P. Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks," *NeurIPS 2020*. arXiv:2005.11401. https://arxiv.org/abs/2005.11401

[7] C. Packer et al., "MemGPT: Towards LLMs as Operating Systems," 2023. arXiv:2310.08560. https://arxiv.org/abs/2310.08560

[8] J. S. Park et al., "Generative Agents: Interactive Simulacra of Human Behavior," *UIST 2023*. arXiv:2304.03442. https://arxiv.org/abs/2304.03442

[9] X. Liu et al., "AgentBench: Evaluating LLMs as Agents," *ICLR 2024*. arXiv:2308.03688. https://arxiv.org/abs/2308.03688

[10] G. Mialon et al., "GAIA: A Benchmark for General AI Assistants," 2023. arXiv:2311.12983. https://arxiv.org/abs/2311.12983

[11] C. Jimenez et al., "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?" *ICLR 2024*. arXiv:2310.06770. https://arxiv.org/abs/2310.06770

[12] S. Yao et al., "τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains," 2024. arXiv:2406.12045. https://arxiv.org/abs/2406.12045

[13] Q. Wu et al., "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation," 2023. arXiv:2308.08155. https://arxiv.org/abs/2308.08155

[14] Z. Xi et al., "The Rise and Potential of Large Language Model Based Agents: A Survey," 2023. arXiv:2309.07864. https://arxiv.org/abs/2309.07864

[15] L. Wang et al., "A Survey on Large Language Model Based Autonomous Agents," *Front. Comput. Sci.*, 2024. arXiv:2308.11432. https://arxiv.org/abs/2308.11432

[16] L. Mei et al., "A Survey of Context Engineering for Large Language Models," 2025. arXiv:2507.13334. https://arxiv.org/abs/2507.13334

[17] I. Ong et al., "RouteLLM: Learning to Route LLMs with Preference Data," 2024. arXiv:2406.18665. https://arxiv.org/abs/2406.18665

[18] L. Chen, M. Zaharia, J. Zou, "FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance," 2023. arXiv:2305.05176. https://arxiv.org/abs/2305.05176

[19] Anthropic, "Building Effective Agents," December 2024. https://www.anthropic.com/engineering/building-effective-agents

[20] Anthropic, "Effective Context Engineering for AI Agents," 2025. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

[21] OpenAI, "A Practical Guide to Building Agents," April 2025. https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf

[22] J. Wiesinger, P. Marlow, V. Vuskovic, "Agents," Google whitepaper, September 2024.

[23] LangChain, "LangGraph Documentation." https://docs.langchain.com/oss/python/langgraph/overview

[24] D. Horthy / HumanLayer, "12-Factor Agents: Principles for Building Reliable LLM Applications." https://github.com/humanlayer/12-factor-agents

[25] NIST, "Artificial Intelligence Risk Management Framework (AI RMF 1.0)," NIST AI 100-1, January 2023. https://doi.org/10.6028/NIST.AI.100-1

[26] European Union, "Regulation (EU) 2024/1689 (Artificial Intelligence Act)," *Official Journal of the EU*, 2024. https://eur-lex.europa.eu/eli/reg/2024/1689/oj

[27] LangChain, "Context Engineering for Agents," 2025. https://blog.langchain.com/context-engineering-for-agents/

[28] S. Zhou et al., "WebArena: A Realistic Web Environment for Building Autonomous Agents," *ICLR 2024*. arXiv:2307.13854. https://arxiv.org/abs/2307.13854

[29] I. Levy et al., "ST-WebAgentBench: A Benchmark for Evaluating Safety and Trustworthiness in Web Agents," 2024. arXiv:2410.06703. https://arxiv.org/abs/2410.06703

[30] X. H. Lù et al., "AgentRewardBench: Evaluating Automatic Evaluations of Web Agent Trajectories," 2025. arXiv:2504.08942. https://arxiv.org/abs/2504.08942

[31] Anthropic, "Introducing the Model Context Protocol," November 2024. https://www.anthropic.com/news/model-context-protocol

[32] OWASP GenAI Security Project / Agentic Security Initiative, "Agentic AI – Threats and Mitigations," February 2025. https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/

[33] NIST, "Artificial Intelligence Risk Management Framework: Generative Artificial Intelligence Profile," NIST AI 600-1, July 2024. https://doi.org/10.6028/NIST.AI.600-1

[34] H. Zhu et al., "OAgents: An Empirical Study of Building Effective Agents," *Findings of EMNLP 2025*. arXiv:2506.15741. https://arxiv.org/abs/2506.15741

[35] N. Wang et al., "Efficient Agents: Building Effective Agents While Reducing Cost," 2025. arXiv:2508.02694. https://arxiv.org/abs/2508.02694

---

## Appendix A: Originality and Attribution Note

The individual disciplines synthesized here — prompting [1], agent reasoning and trajectories [2][3][4], tool use [5][22], retrieval and context [6][16][20], memory [7][8], model routing [17][18], multi-agent orchestration [13], execution patterns [19][23][24], governance [21][25][26], and evaluation [9][10][11][12] — each have established literatures, cited above. All prose in this paper is the author's own; all framework structure, the goal/plan/trajectory exposition, the pirate mnemonic, the case studies, and the readiness rubric are original contributions of this synthesis. Readers are encouraged to challenge, extend, and improve the framework.

*Feedback welcome. This is v1.2 of a living document.*
