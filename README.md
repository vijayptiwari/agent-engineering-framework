# The AI Agent Engineering & Development Framework

### Beyond Prompt Engineering: The 10 Pillars of Production Agent Engineering

> A practitioner framework for designing, building, and operating reliable AI agents — not prompt-only demos.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-blue.svg)](LICENSE)
![Status](https://img.shields.io/badge/status-v1.2%20living%20document-brightgreen)
![Contributions](https://img.shields.io/badge/contributions-welcome-orange)

---

## TL;DR

A production agent is not `Prompt + LLM`. It is a **deterministic envelope around a probabilistic core** — the model reasons; everything around it makes the system reliable:

**Goal + Trajectory + Context + Memory + Tools + Intelligence + Execution + Governance + Evaluation**

Prompt engineering is one pillar of ten. Reliability comes from the system around the model.

📄 **[Read the full white paper →](whitepaper.md)** &nbsp;·&nbsp; 🧾 **[Production Readiness Rubric →](rubric/readiness-rubric.md)** &nbsp;·&nbsp; ✅ **[Design Checklist →](checklists/production-agent-checklist.md)**

---

## The 10 Pillars

| # | Pillar | The question it answers |
|---|--------|--------------------------|
| 1 | **Prompt Engineering** | How do we instruct the agent? |
| 2 | **Goal Engineering** | What should the agent achieve, and when does it stop? |
| 3 | **Trajectory Engineering** | How should the agent move toward the goal? |
| 4 | **Context Engineering** | What should the agent see right now? |
| 5 | **Memory Engineering** | What should persist over time? |
| 6 | **Tool Engineering** | What actions can the agent perform? |
| 7 | **Intelligence Engineering** | Which model should do which work? |
| 8 | **Execution Engineering** | How does the agent run reliably in production? |
| 9 | **Governance Engineering** | How do we keep the agent safe, compliant, and auditable? |
| 10 | **Evaluation Engineering** | How do we know the agent is actually good? |

**Mnemonic:** *Pirates Grab Treasure, Carry Maps, Tools, Inspect Every Golden Egg.*

---

## Why this exists

Most agent discussion over-focuses on the model and the prompt. But agents fail in production for reasons no prompt can fix: vague goals, no validation loops, missing governance, unsupervised actions, silent quality drift. This framework gives engineers, architects, and product teams **one shared vocabulary** for reasoning about agents as production systems — backed by 35 cited sources across peer-reviewed research, industry guidance (Anthropic, OpenAI, Google), and standards (NIST, OWASP, EU AI Act).

It is a **synthesis**, not a claim of inventing each discipline. See the [Originality & Attribution Note](whitepaper.md#appendix-a-originality-and-attribution-note).

---

## What's inside

```
agent-engineering-framework/
├── README.md                              ← you are here
├── whitepaper.md                          ← the full cited white paper (v1.2)
├── LICENSE                                ← CC BY 4.0
├── CONTRIBUTING.md                        ← how to propose changes
├── CITATION.cff                           ← cite this work
├── rubric/
│   └── readiness-rubric.md                ← score any agent 0–5 per pillar
└── checklists/
    └── production-agent-checklist.md       ← pre-deployment design checklist
```

---

## Quick start: use it on your own agent

1. Read the [white paper](whitepaper.md) (≈20 min).
2. Score your agent with the [readiness rubric](rubric/readiness-rubric.md). Any pillar at 0–1 among **Execution, Governance, Evaluation** means you have a prototype, not a product.
3. Walk the [design checklist](checklists/production-agent-checklist.md) before your next deploy.

---

## Contributing

This is a **living, open framework** and contributions are welcome — counter-examples, case studies, additional citations, corrections, and translations especially. Please read [CONTRIBUTING.md](CONTRIBUTING.md) first. Disagreement is useful: if a pillar is wrong or missing, open an issue and make the case.

---

## Citation

If you reference this framework, please cite it (see [CITATION.cff](CITATION.cff)):

> Tiwari, V. P. (2026). *The AI Agent Engineering & Development Framework: The 10 Pillars of Production Agent Engineering* (v1.2).

---

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE). You may share and adapt it, including commercially, with attribution.
