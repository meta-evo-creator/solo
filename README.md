# SOLO ⚡ — a governance system for agent fleets

> One person, an entire agent army — with institutions that are neither forgotten nor bypassed.

SOLO is not an agent framework. It is the governance layer above one: it decides **who may change the rules**, **how a change is approved**, and **who checks whether the change was actually made**.

## TAFA — Tripartite Authority-Function Architecture

| Layer | Holds | Never does |
|:--|:--|:--|
| **Rule layer** | The human owner (Human in the Loop) | write execution logic, run tasks |
| **Execution layer** | Agents, cron, skill pipelines | change rules, self-audit |
| **Audit layer** | A heterogeneous, read-only auditor | execute, edit files, widen its scope |

Closed loop: rule layer sets rules → execution layer executes → audit layer finds violations → routes back → proposal → rule layer approves → fix → guardrail precipitation → audit layer re-verifies.

Institutional isomorphism: the structure follows the governance gene of Chinese public hospitals — Party committee decides, president executes, discipline inspection committee oversees — rather than importing a separation-of-powers model into AI governance.

## What is in this repository

- **`SKILL.md`** — the rule layer's working document: the three layers, the six-step change gate, `755` (seven iron laws + five principles + the MEV five layers), the entropy-reduction contract, and the references index.
- **`SOUL.md`** — the core kernel: seven iron laws, four privilege rings, five principles, MEV five layers, skill routing, memory guardrails.
- **`papers/`** — writing on guardrail-style AI governance in practice.

The full working set (audit specs, closed-loop SOP, changelog, pitfall logs) ships with the skill bundle as `references/`.

## The six-step change gate

Any change to rules, config, cron payloads, skill trees or memory walks one chain:

0. **Iron laws** — if any of the seven fails, stop.
1. **Layer** — crossing a boundary means propose only, never execute.
2. **Three cards** — which rule should already have covered this, why didn't it, and is the new rule orthogonal?
3. **Authority** — which layer am I in, am I entitled to act?
4. **Approval** — show first; approval is recognised by the verb, not by the word "fix".
5. **Trace** — every Ring-3 change leaves a blocked-state kanban card.
6. **Precipitation** — attribution → localisation → solidification → verification, or it is not done.

## Design stance

- **Guardrails do not rely on self-discipline.** Rules are mechanical requirements, not reminders.
- **Default delivery is zero new behaviour.** Plug, absorb, record. Anything new must be counter-proven by an incident.
- **No architecture diagrams.** A diagram has to be redrawn every time the mechanism changes and cannot be audited by a script; textual criteria can be checked.
- **The body records criteria, not snapshots.** Counts of agents, jobs or skills live where they are generated, not in the document that rots.

## Status

A personal governance system, published as a reference. Version 3.2 — structural condensation of the rule layer (311 → 118 lines, zero criteria removed).
