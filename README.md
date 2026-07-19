# SOLO ⚡ — TAFA Triple-Power Separation Architecture

> **One person, an entire agent army.**
> 权层定规则，能层干实事，审层管审计。制定规则的人不执行，执行规则的人不监督，监督执行的人不参与执行。

## Why TAFA?

The agent ecosystem is converging on single-layer swarms — OpenAI Swarm, Microsoft Agent Framework, Agency Swarm. All follow the same pattern: agents orchestrate agents, with quality left to external eval frameworks.

**TAFA is structurally different:**

| | Market (Swarm-style) | TAFA |
|:--|:---------------------|:-----|
| **Architecture** | Single-layer agent mesh | **Three-layer interlocked闭环** |
| **Quality** | External eval (RagaAI, MLflow) | **Built-in Schema Gates at every handoff** |
| **Feedback** | Human review → manual fix | **Phase 8 Self-Repair (<1min auto-patch)** |
| **Permission** | API-key level | **4-Ring Privilege Model (structurally impossible to violate)** |
| **Domain** | General-purpose | **Discipline inspection · Hospital audit · Compliance** |

> *"Who audits the auditor?" — the question every agent framework avoids. TAFA answers it: Rule Layer sets rules, Execution Layer follows them, Audit Layer verifies both. Three layers, zero overlap.*

## Architecture

```
┌──────────────────────────────────────────────────┐
│              ① Rule Layer — 655 System            │
│  6 Iron Laws · 5 Principles · 5 MEV Layers        │
│  Sets rules, never executes.                      │
│              │                         ▲          │
│     rules constrain                 audit feeds   │
│              ▼                         │          │
├──────────────────────────────────────────────────┤
│         ② Execution Layer — Skill Pipelines       │
│  discipline-inspection ⚔️ · MSF ⚒️                 │
│  supervision-inspection 🔍 · deep-research 🔍      │
│  hospital-inspection 🏥 · compliance-analysis 🏛️   │
│  Executes rules, never audits itself.             │
│              │                                    │
│     outputs for audit                             │
│              ▼                                    │
├──────────────────────────────────────────────────┤
│          ③ Audit Layer — solo-audit 🔍            │
│  Pro-only · Read-only · Proposal-only             │
│  Audits execution, never participates.            │
└──────────────────────────────────────────────────┘
```

## Contents

| Directory | Version | Description |
|:----------|:-------:|:------------|
| `SKILL.md` (root) | v2.2 | SOLO TAFA meta-architecture |
| `skills/discipline-inspection/` | v2.6.0 | 10-Agent DI pipeline with self-repair |
| `skills/solo-audit/` | v5.8 | 4-Agent audit pipeline with 655 analysis |

## Install

```bash
git clone https://github.com/meta-evo-creator/solo.git
```

## License

MIT License — see individual skill files for details.
