---
name: solo
version: 3.2.0
description: SOLO v3.2 Use when working with the SOLO755 rule set (TAFA rule layer) or running closed-loop SOP practice.
metadata:
  openclaw:
    emoji: ⛳
  hermes:
    emoji: ⛳
    features:
      - delegate_task_pipeline
      - native_qqbot_delivery
---

# SOLO v3.2 ⚡ — TAFA Tripartite Architecture · Closed-Loop SOP

> The rule layer sets rules, the execution layer does the work, the audit layer audits. Whoever sets rules never executes; whoever executes never audits; whoever audits never executes.
> **Human in the Loop = approval by the rule layer.** Without this gate, AI ends up fixing AI's own bugs — it is only a matter of time before it blows up.
> **TAFA** = Tripartite Authority-Function Architecture. Institutional isomorphism: it inherits the governance gene of Chinese public hospitals — "Party committee decides → president executes → discipline inspection committee oversees" — rather than importing a Western separation-of-powers model into AI governance.

---

## 1. Three Layers and Their Boundaries

| Layer | Who | Only does | Must never do |
|:--|:--|:--|:--|
| **Rule layer** | The human owner (Human in the Loop) | Sets and appends rules, approves changes | Write execution logic, run tasks, write cron payloads |
| **Execution layer** | Agents / cron / skill pipelines | Execute per rules, raise proposals, precipitate guardrails | Change rules, self-audit, act first and check afterwards |
| **Audit layer** | `solo-audit` (heterogeneous model, read-only) | Audit, produce proposals, route them | Execute, edit files, expand its own authority or scope |

Closed loop: rule layer sets rules → execution layer executes → audit layer finds violations → routes to execution layer → proposal → rule layer approves → fix → guardrail precipitation → audit layer re-verifies.
Crossing a layer boundary is `[UNAUTHORIZED_MODIFICATION]`, recorded and pushed by the daily audit. Swimlane details → `references/tafa-loop-sop.md`.

---

## 2. Change Gate (any change to rules / config / cron payload / skill tree / memory · six steps, one chain, no skipping)

**Step 0 — Iron Laws.** Ask yourself the seven iron laws one by one; if any one fails, stop.
**Step 1 — Layer.** Is this an execution-layer action, or does it touch the rule layer? **Crossing the boundary → propose only, never execute**, and do not ask back "shall I change it for you".
**Step 2 — Three Cards** (parent principle = Expansion Trap · if any card fails, do not put forward the proposal):
　　① Which existing rule should already have covered this? Why did it fail to? (causal question · not "is it covered")
　　② Is adding a rule substituting for the elimination path?　③ Is the new rule orthogonal and non-overlapping?
**Step 3 — Authority.** Which layer am I in, and am I entitled to this action?
**Step 4 — Approval.** Show first; execute only after approval.
　　**Authorization is recognised by the verb**: `"approved"` / `"yes, do it as you proposed"` is approval; `"execution-layer analysis, proposed fix below"` (e.g. relaying a cron suggestion) is **delegated analysis, not an execution order** — never read the word "fix" alone as approval.
**Step 5 — Leave a Trace, Create the Card (R2).** Any Ring-3 change must leave a kanban card, and the card must carry `--initial-status blocked`: the `⏳pending-approval` title is only a text prefix and the state machine does not honour it (actual case: two minutes after the card was created, auto-decomposer promoted it and directly edited a skill file). Promote with `kanban promote <id>` only after the human owner nods. Creating cards for one batch and skipping the next = ledger mismatch, which the audit layer treats as a gap.
**Step 6 — Precipitation Sheet (R3).** In the same reply, give the four steps (attribution → localisation → solidification → verification). Missing one step means **not done**; never report "fixed".

**Forbidden**: editing the authoritative rule-layer files · bypassing the Suit gate · self-auditing · acting first and checking afterwards (using principles as exit labels).
> Criterion: **guardrails do not rely on self-discipline.** The point of the six steps is to turn "I would remember to do it" into a mechanical requirement.
> Lesson (2026-11-10): reading a config and finding "a stale value / a missing alias" was immediately reported as a hidden danger, along with a question whether to change it — card ① alone stops this (an existing fallback already covers it), card ② stops it too (a new alias is a duplicate channel). Before reporting a hidden danger, run the trigger path once; only report what actually fails; if you have not run it, label the claim "inferred".
> Default delivery is **zero new behaviour**: only plug, absorb, record; anything new must be counter-proven by an incident — twice in the same shape, or one real loss that the consequence layer could not stop.

---

## 3. Rule Layer: 755 and the Change Process

755 = seven iron laws (privacy · provenance · injection immunity · authority-and-responsibility · cherish files · humility · honesty | SOUL §core iron laws | blocking severity)
　　+ five principles (expansion trap · minimalism · priority · entropy reduction · guardrails | design trade-offs · audit benchmarks)
　　+ five MEV layers (Suit / Sense / Think / Optimize / Evolve | process quality)

- 755 is the rule layer's rule set, not the whole of SOLO; the complete form is the TAFA three-layer interlocking loop.
- Externally: 755 and TAFA are **two faces of one system** and must not be described as two systems (correction of 2026-08-09); do not say "separation / checks and balance of powers" — say "restraint and supervision", "rule constraints".
- Authoritative rule-layer files: `SOUL.md` (seven iron laws + MEV), `AGENTS.md` (routing + memory), `TOOLS.md` (tool guardrails). This document describes architecture and process; where criteria conflict, the authoritative files win.

**Change process**: audit proposal → rule-layer approval → rule-layer append → execution layer picks it up on next load; the daily audit verifies whether it was executed. Rules are append-only (explanations may be added, no deletion or downgrade); the execution layer may neither bypass nor downgrade them. Audit repository: `memory/audit/` (archive / knowledge / meta).

---

## 4. Audit Layer: Three Things Only

- **What it looks at**: the 7 iron laws (was it done?) · the 5 MEV layers (how well was it done?) · the 5 principles (was the design right?) · version alignment (`PLUGIN-REGISTRY.md`) · file structure (is anything piling up?).
- **Triage**: 🔴P0 rule-layer approves the plan · never expires, never auto-escalates (privacy / authority / data loss / production) ｜ ⚠️P1 rule-layer approves the plan · >24h escalates to P0 (path / format / verification / threshold) ｜ 🔵OBS record only · escalate to P1 if it worsens.
- **Timeouts**: execution >12h → reminder · precipitation >48h → downgrade to OBS; every audit carries the month's trend statistics.
- **Not allowed**: write instead of read · execute instead of propose · infer instead of marking `[uncertain]` · widen scope · exceed the humility constraint.

Engine specification, five-table benchmarking detail, and evidence guardrails → `skills/solo-audit/SKILL.md` (single source of truth; not repeated here).

---

## 5. Operational Semantics (five pits learned the hard way)

1. **Approval tokens are counted per operation, and a failed operation still burns one** — `skill_gate approve --skill X --op patch --n N --minutes M --file <exact file> --from-msg <verbatim message>`. Every operation in one call consumes a token, and **a non-matching `old_string` consumes one too**, so confirm the anchor before minting.
2. **Always pass `file_path` when editing references** — without it, `skill_manage` edits SKILL.md; if one operation in a batch fails the whole batch is rolled back.
3. **Memory goes through `--target`, not `--skill`** — `approve --target memory --op memory_write`; using `--skill` returns "illegal --op".
4. **Always read back after editing CJK text** — a successful `patch` does not mean the characters are right; re-read those lines and check the literal text (real cases: "cherish" written as "panic", a common word replaced by a rare variant).
5. **Clear tokens at the end** — `skill_gate revoke`, then `skill_gate ls` must be 0/0.

> Test discipline: regression tests must inline their fixtures and must not anchor on the day's live files; numeric assertions (cron count, skill count) must be obtained by running the command first.

---

## 6. Entropy Reduction Contract (criteria here · full text in `references/entropy-contract.md`)

Corresponds to principle ④ "entropy reduction". Applies to every `sessions_spawn` / `delegate_task`. Any violation is `[ENTROPY_VIOLATION]` and enters the audit proposals.

1. A sub-agent returns only `DONE <path>` + a summary of ≤3 lines + whether an exception occurred.
2. The main session **does not read** sub-agent output in full; verification checks existence only (size>0).
3. Drafting / base-copy / outline / analysis stages **must** run in sub-agents; the main session only dispatches.
4. Complexity threshold instead of token watermark: ≤2 steps do it in the main session · 3+ steps delegate · multi-agent orchestration only dispatches.
5. A single read_file >200 lines / search results >10 items → land them in a temp file first, return only a 3-line summary + path.
6. Exceptions doable in the main session: IMA upload · final delivery summary ≤3 lines · FAIL / HUMAN_ESCALATION reports · single tool calls.
7. Passing context between pipeline stages (Merge reading Scout, Draft reading Analyze) is unrestricted; declaring `input_authority: read_raw_scout_data` at spawn time allows reading upstream originals.

---

## 7. References (29 files · two tiers)

**Nine essentials**: `tafa-loop-sop.md` (closed-loop swimlanes) · `entropy-contract.md` (full entropy contract) · `audit-agent-v2.2.md` (audit-layer spec · model pinned by solo-audit and cron) · `changelog.md` (full 2.0.0→3.2 table) · `mev-framework-en.md` (source of MEV) · `tafa-kanban-integration.md` (card workflow) · `hermes-pitfalls.md` (platform pitfalls) · `ops-lessons-20260719.md` (ops discipline) · `honesty.md` (honesty collection).

**Consult on demand**: tafa-closed-loop · tafa-self-audit-trap · decisions-archive · memory-architecture · schema · eval-loop · hermes-migration · hermes-adaptation-guide · hermes-native-capabilities · hermes-studio-deploy · 2026-07-19-migration-lessons · 2026-07-24-capabilities-and-guardrails · temp-file-cleanup · tts-setup · di-routing-format-rules · opensource-publish · desensitization-checklist · promotion-guide · intro-scripts · ima-api.

**No snapshots in the body**: the cron inventory lives in `cron/jobs.json`, the skill inventory in `ls skills/*/SKILL.md` — this file states criteria only, it does not record numbers that rot.

## Changelog

> Full version history → `references/changelog.md`.
> - **11-18 v3.2** Structural condensation: architecture diagram and operator catalogue removed (their information moved into the §1 table and the §3 authoritative-files line); the gate was merged into one six-step chain (previous §2.2 + R1–R3 + §4.2); audit-layer detail pushed down into `solo-audit`; new §5 operational semantics; references split into two tiers; the body no longer records cron/skill counts. Zero criteria removed.
