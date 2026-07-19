---
name: discipline-inspection
version: 2.6.0
description: |
  Discipline Inspection v2.6.0 ⚔️ Methodology v2.3: Dual-Factor + 6 Modules + P1/P2 Framework + 10-Agent Full-Gate Pipeline + Resilience + LESSON(urgency routing) + Quality Dashboard + KG Activation + Agent Tuning + KG Writeback + Hermes-Native Self-Repair (Phase 8). Focused on party discipline inspection case analysis.
platforms:
  - openclaw
  - hermes
tools:
  - ripgrep
  - sessions_spawn
  - memory_search
  - tavily_search
  - skill_manage
metadata:
  openclaw:
    emoji: ⚔️
  hermes:
    emoji: ⚔️
    features:
      - instant_self_repair
      - delegate_task_parallel
---

# Discipline-Inspection ⚔️ v2.6.0

> **Discipline as the yardstick, vigilance as constant.** 10-Agent Full-Gate Pipeline + Schema Gates (100% coverage) + Pipeline Resilience + LESSON Collection + Quality Dashboard + Knowledge Graph Enrichment + Agent Tuning + Case Ruling Logic (L1/L2/L3) + **Hermes-Native Instant Self-Repair (Phase 8)**.
> 🔓 Open source under MIT License.

> 📎 Shared config: `skills/supervision-shared/shared-config.yaml` · Agent files: `agents/*.md` (10 files, full Gate coverage) · Case index: `references/case-index.json`

---

## 🛡️ No-Authority Boundary

This skill is a **refs-only / no-authority** capability package.

**Outputs:** violation_finding_ref · evidence_chain_ref · article_match_ref · responsibility_assessment_ref · sanction_recommendation_ref · mitigating_aggravation_ref · owner_gate_handoff_ref

**This skill NEVER produces:** Final sanction decisions · Accountability conclusions · Organizational action decisions · Final case characterization · Any output substituting for committee meetings or statutory procedures.

> Boundary corresponds to SOLO 655 Iron Rule ④ (minimal agency — every authorization is temporary, scoped, and revocable).

---

## ⛔ Entry Block (Cannot Skip · Cannot Degrade)

**Sole entry point:** Suit Phase 1 Confirmation → `sessions_spawn Agent 0 (scope)`.

The following are recorded as `[UNSOURCED-EXECUTION]`:
- Searching regulations / writing analysis / drafting reports in the main session
- Citing regulation article numbers from memory
- Skipping any Agent on grounds of "simple case"

**This clause is not degradable under any circumstances.**

---

## 📋 Pipeline

### Phase → Ref Mapping

| Phase | Agent | Output | Ref Family |
|:------|:------|:-------|:-----------|
| 0 | Scope | `agent0-scope.json` | `source_pack_ref` |
| 1a ∥ 1b | Search-rg ∥ Search-pkulaw | json | `article_match_ref` ∥ `version_verified_ref` |
| 1c | Merge | `agent1-merged.json` | `merged_search_ref` |
| 2 | Audit | `agent2-audit.json` | `evidence_chain_ref` |
| 3 | Analyze | `agent3-analyze.json` | violation + responsibility + sanction refs |
| 4 | Draft | `agent4-draft.md` | Synthesis of upstream refs |
| 5 | Review | `agent5-review_ledger.json` | Scoring matrix + fixes |
| 6 | Revise | `agent6-final.md` | Corrected final |
| 7 | Publish | `agent7-publish-report.json` | Lesson collection + Quality dashboard + IMA upload |
| 8 | Self-Repair | `_auto_fix_log.json` | Hermes-native instant patch via `skill_manage` |

### Pipeline Diagram

```
Agent 0 ─┬─→ Agent 1a (rg WIKI) ─┐
         └─→ Agent 1b (pkulaw)  ─┤ ∥ parallel
                                 ↓
                          Agent 1c (merge)
                                 ↓
                          Agent 2 (audit)
                                 ↓
                          Agent 3 (analyze)
                                 ↓
                          Agent 4 (draft)
                                 ↓
                          Agent 5 (review)
                                 ↓
                          Agent 6 (revise)
                                 ↓
                          Agent 7 (publish: LESSON + quality + IMA)
                                 ↓
                          Agent 8 (self-repair: skill_manage auto-patch) ← 🆕 v2.6.0 Hermes-native
```

**Each Agent runs in isolated session (context:isolated, lightContext:true). Main session only handles spawn + gate + file existence verification + Agent 7 IMA upload call + Agent 8 auto-patch.**

### Guardrail Routing

| Mode | Trigger | Pipeline | Agents |
|:-----|:--------|:---------|:------:|
| **full** | Case characterization, sanction recommendation | 0→(1a∥1b)→1c→2→3→4→5→6→7 | 9+1 |
| **interview** | Interview outline | 0→(1a∥1b)→1c→2→3+4→7 | 6+1 |
| **quick** | Regulatory consultation, article lookup | 0→(1a∥1b)→1c→2→7 | 5+1 |

**Routing decision:** After Agent 0 completes, select mode based on `task_type`.

---

## 🔒 Protocol

### Agent Gate Protocol (v2.4 — Schema Validation)

After each Agent completes, execute gate in order:

**Gate A — File Existence (v2.3):** `read <output> → exists + size > 0`. File not found → mark `failed` → write `pipeline_failure_log.json` → do NOT proceed.

**Gate B — Schema Validation (v2.4 NEW):** Parse JSON output, validate against Agent's `## Output Schema` section in its agent file.
- Required fields present → PASS → proceed to next Agent
- Required fields missing → mark Agent `SCHEMA_FAIL` → write `pipeline_failure_log.json` → **return to that Agent for re-run (max 1 retry)**
- Markdown output (Agent 4/6): `rg` search for required heading markers

**Gate C — Summary Record (v2.4 NEW):** Gate passed → record `<output_path> + <agent_name> + <status> + <key_metrics>` to quality context for downstream agents.

```
Gate flow: A(exists?) → B(schema valid?) → C(record summary) → next Agent
                        ↓ FAIL
                   pipeline_failure_log.json → re-run Agent (1 retry) → still FAIL → halt
```

**⛔ Gates A+B only verify structure — do NOT read full content into main session context.**

### Solo Status Protocol
Before/after each Agent spawn, update `./solo/pipeline-status.json` with pipeline_id + phase status (structure defined in `skills/solo/SKILL.md`).

### Output Path Protocol
```
{pipeline_output_dir}/
├── agent0-scope.json          ← Phase 0: scoping + regulation_list
├── agent1a-search-rg.json     ← Phase 1a: rg WIKI (includes source_line)
├── agent1b-search-pkulaw.json ← Phase 1b: pkulaw version verify ∥
├── agent1-merged.json         ← Phase 1c: merge (match + discrepancy tags)
├── agent2-audit.json          ← Phase 2: citation audit
├── agent3-analyze.json        ← Phase 3: deep analysis
├── agent4-draft.md            ← Phase 4: report/outline
├── agent5-review_ledger.json  ← Phase 5: quality audit
├── agent6-final.md              ← Phase 6: final version
├── revision_log.json            ← Phase 6: revision log
├── agent7-publish-report.json   ← Phase 7: lesson scan + quality dashboard report
├── pipeline_failure_log.json    ← v2.4: failure tracking + resume checkpoint
├── _quality.json                ← v2.4: pipeline quality metrics aggregation
└── _lessons.json                ← v2.4: structured lesson collection
```
**task_id format:** `DI-YYYYMMDD-seq`

### Pipeline Resilience (v2.4 NEW)

**Failure Log:** Any Agent gate failure → write `pipeline_failure_log.json`:
```json
{
  "pipeline_id": "DI-YYYYMMDD-seq",
  "failed_agent": "agentN",
  "failure_type": "FILE_MISSING | SCHEMA_FAIL | GATE_FAIL",
  "failure_detail": "...",
  "completed_outputs": ["agent0-scope.json", "agent1a-search-rg.json", ...],
  "retry_count": 0,
  "timestamp": "ISO-8601"
}
```

**Resume Protocol:** Before spawning any Agent, check if its output file already exists in `pipeline_output_dir/`:
- Output exists + not marked FAILED in `pipeline_failure_log.json` → **skip** (already completed)
- Output exists + marked FAILED → **re-run** (retry)
- Output missing → **spawn** (normal execution)

**Max retries:** 1 per Agent. After 2nd FAIL → pipeline halts, report to domain owner with `pipeline_failure_log.json`.

**Cross-session resume:** Pipeline can be resumed from a different session if `pipeline_failure_log.json` + completed outputs exist. Main session reads failure log → spawns next pending Agent.

---

## Agent Specifications

### Agent 0: Scope (Issue Scoping)
**Input:** User-provided case facts
**Output:** `agent0-scope.json` — case_summary · legal_framework · evidence_assessment · task_type · downstream_handoff · regulation_list (parallel pipeline key field)
**Agent file:** `agents/scope.md` (includes Step 0b identity verification + identity→regulation mapping)

### Agent 1: Search (Parallel Architecture)
**Architecture:** Agent 0 `regulation_list` → (1a ∥ 1b) simultaneously
- **1a (search-rg):** Full-text rg search of regulation library → `agent1a-search-rg.json`. See `agents/search-rg.md`.
- **1b (search-pkulaw):** pkulaw version verification → `agent1b-search-pkulaw.json`. See `agents/search-pkulaw.md`.

### Agent 1c: Merge (Three-Way Regulation Merge)
**Input:** agent0-scope.json + agent1a + agent1b
**Output:** `agent1-merged.json` — three-way match per regulation (rg hit + pkulaw status) with discrepancy markers (UNVERIFIED / search_miss). Five-status classification per regulation: matched / text_only / version_only / missing.
**Agent file:** `agents/merge.md` (v2.4-P3: formerly inline, now with dedicated agent file + Output Schema + Gate B coverage)

### Agent 2: Audit (Regulation Citation Audit)
**Input:** agent0-scope.json + agent1-merged.json
**Output:** `agent2-audit.json` — article number verification · [UNCERTAIN] block check · Subject-Behavior-Result three-element verification · version consistency · conclusion: PASS / PASS_WITH_WARNINGS / FAIL
**Agent file:** `agents/audit.md`

### Agent 3: Analyze (Deep Analysis · v2.3)
**Input:** agent0-scope.json + agent1-merged.json + agent2-audit.json
**Output:** `agent3-analyze.json` — P1 conceptual framework → violation+responsibility two-factor analysis (11-step workflow with embedded M2-M9 modules) → dual-round adversarial debate → case matching → P2 procedural guidance
**Agent file:** `agents/analyze.md` (includes full P1+P2 framework, 11-step workflow, YAML output template, medical case specialty)

### Agent 4: Draft (Report Writing)
**Input:** agent0-scope.json + agent3-analyze.json
**Output:** `agent4-draft.md` — seven-chapter report with interview outline guardrails (criminal transition warning + regulation cross-reference + signature subject three-layer distinction)
**Agent file:** `agents/draft.md`

### Agent 5: Review (Quality Audit)
**Input:** agent4-draft.md + agent1a + agent1b
**Output:** `agent5-review_ledger.json` — Twenty-Four-Character Policy 6-dimension scoring matrix (25/20/20/15/10/10 weights). ≥80 PASS · 60-79 REVISE · <60 REJECT
**Agent file:** `agents/review.md`

### Agent 6: Revise (Fix)
**Input:** agent4-draft.md + agent5-review_ledger.json
**Output:** `agent6-final.md` + `revision_log.json`
**Agent file:** `agents/revise.md`

### Agent 7: Publish (v2.6.0 — IMA Upload + LESSON Collection + Quality Dashboard + Self-Repair Handoff)

**Agent file:** `agents/publish.md` (includes Output Schema for Gate B validation)

**Step 7a — IMA Upload:** Called from main session: `node skills/solo-file-transfer/scripts/ima_upload.cjs <agent6-final.md> <KB_ID>`

**Step 7b — LESSON Collection (v2.4 → v2.5.1):** After upload, scan all agent output files for `[LESSON]` markers:
- `rg "\[LESSON\]" memory/inspection-drafts/{task_id}/` across all json/md files
- Collect structured lessons → append to `memory/inspection-drafts/_lessons.json`:
```json
{
  "pipeline_id": "DI-YYYYMMDD-seq",
  "date": "ISO-8601",
  "lessons": [
    {
      "source_agent": "agent3",
      "category": "methodology | regulation | case | procedure",
      "urgency": "P0 | P1 | P2",
      "lesson": "...",
      "action": "UPDATE_AGENT | ADD_CASE_TAG | UPDATE_REGULATION | NOTE_ONLY"
    }
  ]
}
```
**Urgency routing (v2.5.1 NEW):**
- `urgency: P0` (critical methodology bug / systematic error pattern) → **IMMEDIATE notification** to domain owner. Compose: `"⚡ DI URGENT LESSON | {task_id} | {source_agent} | {lesson_summary_first_80_chars}"` → send via messaging tool. Duration from finding to notification: <1 minute (vs 30-day monthly cron cycle).
- `urgency: P1` (important pattern / recurring gap) → flagged for next regulation-manager monthly cron review.
- `urgency: P2` (minor improvement) → recorded only, no active push.
- Default (no urgency field): treated as P1.

Lessons with `action: UPDATE_AGENT` → flagged for next regulation-manager monthly cron review (P1/P2) or immediate push (P0).

**Step 7c — Quality Dashboard (v2.4 NEW):** Aggregate pipeline quality metrics → write `memory/inspection-drafts/_pipeline_quality_log.json`:
```json
{
  "pipeline_id": "DI-YYYYMMDD-seq",
  "timestamp": "ISO-8601",
  "quality": {
    "agent2_audit": "PASS | PASS_WITH_WARNINGS | FAIL",
    "agent5_score": 0-100,
    "agent5_dimensions": {
      "accurate_characterization": 0-100,
      "clear_facts": 0-100,
      "conclusive_evidence": 0-100,
      "appropriate_disposition": 0-100,
      "complete_procedures": 0-100,
      "procedural_compliance": 0-100
    },
    "upstream_feedback": [{"agent": "agentN", "issue": "...", "severity": "critical|high|medium|low"}],
    "lessons_generated": 0,
    "retries": 0
  }
}
```
**Quality trend check:** After append, compare current score with last 3 runs' average → deviation >15 points → flag for review.

⛔ **Report purity:** IMA upload = pure analysis content only. No pipeline IDs / Agent identifiers / audit metadata. Metadata → `memory/inspection-drafts/<case>/`.

---

## Phase 8: Self-Repair (v2.6.0 · Hermes-Native) 🆕

> **The pipeline that fixes itself.** Phase 8 runs in the main session after Agent 7 completes — it is NOT a spawned agent. It uses Hermes's `skill_manage` tool to instantly patch agent files when P0 lessons are collected.

### Self-Repair Protocol

```
Agent 7 completes → main session reads agent7-publish-report.json
                         ↓
              For each lesson in lessons[]:
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   urgency=P0       urgency=P1       urgency=P2
   action=UPDATE    action=UPDATE    any action
        ↓                ↓                ↓
   confidence=HIGH  confidence=any    → NOTE_ONLY
        ↓                ↓           (record only)
   skill_manage(        ↓
   action='patch',  write to
   old_string,      _pending_fixes.json
   new_string)      → flagged for
        ↓           solo-audit review
   write success
   to _auto_fix_log.json
```

### Decision Matrix

| Urgency | Action | Confidence | Behavior |
|:--------|:-------|:-----------|:---------|
| P0 | UPDATE_AGENT | HIGH | **Auto-patch** via `skill_manage(action='patch')` — no human review |
| P0 | UPDATE_AGENT | MEDIUM | Push notification to domain owner for one-click approval |
| P0 | UPDATE_AGENT | LOW | Push notification for full review |
| P0 | UPDATE_REGULATION | any | Write to `_pending_fixes.json` for regulation-manager cron |
| P0 | UPDATE_KG | any | Write to `_pending_fixes.json` for monthly cron |
| P0 | ADD_CASE_TAG | any | Auto-apply to case-index.json (confined to JSON append) |
| P1 | UPDATE_AGENT | any | Write to `_pending_fixes.json` → flagged for next solo-audit |
| P2 | any | any | Record only in `_lessons.json` |

### Auto-Fix Log (`_auto_fix_log.json`)

Every auto-patch writes an audit trail:
```json
{
  "pipeline_id": "DI-YYYYMMDD-seq",
  "timestamp": "ISO-8601",
  "auto_fixes": [
    {
      "lesson_index": 0,
      "source_agent": "agent3",
      "target_file": "agents/analyze.md",
      "target_section": "## Step 5 — Responsibility Assessment",
      "patch_applied": true,
      "old_text_preview": "first 80 chars...",
      "new_text_preview": "first 80 chars..."
    }
  ],
  "pending_fixes": [],
  "notifications_sent": 0
}
```

### P0 Notification Format (MEDIUM/LOW confidence)

When a P0 lesson cannot be auto-applied (confidence != HIGH), the main session composes:
```
⚡ DI URGENT PATCH | DI-YYYYMMDD-seq | agentN | confidence: MEDIUM
Target: agents/analyze.md § Step 5
Lesson: {lesson_summary}
Old: {old_text preview}
New: {new_text preview}
→ Reply "apply" to auto-patch, "skip" to dismiss
```

### Interaction with Existing Cron

- The monthly regulation-manager cron still processes P1 lessons from `_pending_fixes.json`
- The solo-audit daily cron reads `_auto_fix_log.json` to verify patches were clean
- The P0 urgency routing **bypasses** the cron cycle entirely — patches land in seconds, not 30 days

### Safety Constraints

- **Only `UPDATE_AGENT` with `confidence=HIGH` is auto-applied.** All other combinations require notification.
- **Patch target is restricted to `agents/*.md` within this skill.** No external file patching.
- **Each auto-patch writes an audit log entry.** Rollback is possible via git history (this skill is under version control).
- **`skill_manage(action='patch')` uses fuzzy matching** — if `old_string` doesn't match, the patch fails cleanly without corrupting the file.

---

## Anonymization Protocol
Organization names use anonymized placeholders ("A tertiary hospital" / "A provincial hospital"). Sensitive data → reference Agent 0 scope file path, not written directly into prompts.

---

## Regulation Knowledge Base (Pluggable Provider Architecture)

Pipeline startup selects knowledge source by priority:
1. `WIKI_PATH` env → wiki-provider (full regulations)
2. `pkulaw-mcp` available → overlays pkulaw-provider (version verification)
3. Neither → default-provider (3 core regulation demo)

| Provider | Search | Version | Cases | Use Case |
|:---------|:------:|:-------:|:-----:|:---------|
| wiki-provider | 45+ full text | ❌ | 11 cases | Organizations with WIKI |
| pkulaw-provider | ✅ | ✅ | ❌ | pkulaw subscribers |
| default-provider | 3 core | ❌ | ❌ | Open-source users |

**Degradation:** No wiki → fallback default (`⚠️ Only 3 core regulations`). No pkulaw → Agent 1b outputs `VERSION_UNVERIFIED` (pipeline not blocked).

> 📚 Full regulation inventory maintained in wiki: `${WIKI_PATH}/discipline/regulations/` (45+ regulations), `${WIKI_PATH}/medical/` (8 healthcare standards), `${WIKI_PATH}/discipline/guiding-cases/` (11 CCDI guiding cases). Auto-updated monthly by regulation-manager cron.

---

## LEARNED PATTERNS

> Architectural methodology changes that shaped the current pipeline structure.
> Historical bug fixes (v1.0.x) are encoded in agent files — see `agents/*.md`.
> Execution-level patterns absorbed from practical cases → injected directly into agent files.

### v1.1.0 — Agent 1a/1b Split: pkulaw Structurally Unskippable
Splitting rg search + pkulaw verification into independent sub-sessions makes version verification structurally unskippable. **Principle:** High-risk substeps as independent sessions = impossible to bypass.

### v1.4.0 — Provider Architecture: Three-Layer Decoupling
Regulation data layer decoupled from pipeline via pluggable Provider interface (default/wiki/pkulaw). **Principle:** Pipeline methodology is the product, regulation data is the fuel — deliver separately, configure independently.

### v1.5.0 — Pipeline Parallelization: 0→(1a∥1b)→1c→2
Agent 0 feeds 1a and 1b simultaneously (no mutual dependency). 1b input changed from agent1a to agent0 — eliminates implicit coupling where "1a miss = 1b blind spot." **Principle:** Eliminating implicit coupling > increasing parallelism.

### v2.0.0 — Dual-Round Adversarial Debate + Structured Case Indexing
Agent 3 runs prosecution round + defense round (3 rebuttal points). Structured case indexing (11 guiding cases × 7 dimensional tags) for rule-based matching. **Principle:** Structured perspective-shifting combats cognitive blind spots; same agent runs two rounds — no new agent overhead.

### v2.3.0 — P1 Policy Framework + P2 Procedural Guidance
P1: 4 conceptual frameworks (Three Distinctions · Style-to-Corruption Gradient · Superficiality Identification · Seeing Through Appearance to Essence) as analysis backbone before violation determination. P2: 4 procedural rules (Sanction Matching · Asset Disposal · Accountability Pitfalls · Retirement≠Immunity) appended after conclusion. Agent 1c merge logic inlined (no standalone file). **Principle:** Higher-level conceptual framework reduces two-factor analysis blind spots.

### v2.6.0 — Hermes-Native Instant Self-Repair (2026-07-19) 🆕

**The feedback loop collapses from 30 days to <1 minute.**

In OpenClaw, the DI pipeline could only *collect* lessons — actual agent file patches required an external monthly cron cycle. In Hermes, the `skill_manage` tool is available to the main session, enabling:

- **Instant P0 patching:** `skill_manage(action='patch')` fires immediately after Agent 7, with `confidence=HIGH` gate for safety
- **Structured handoff:** Agent 7 now includes `target_file`, `old_text`, `new_text`, and `confidence` fields — making each lesson a machine-executable patch proposal
- **Audit trail:** Every auto-patch writes `_auto_fix_log.json` for review by solo-audit cron
- **Tiered routing:** P0/HIGH → instant, P0/MEDIUM → notification, P1 → pending queue, P2 → record only

**Platform dependency:** Phase 8 is **Hermes-native** — it requires `skill_manage` which is not available in OpenClaw's agent sessions. On OpenClaw, the pipeline gracefully degrades: Phase 8 is skipped, lessons go to `_lessons.json` as before. No pipeline change needed — the `platforms` field in frontmatter declares both.

**Principle:** Self-repair capability is the difference between a tool that reports its mistakes and a tool that fixes them. The DI pipeline is now the latter.

### v2.5.1 — Urgency Routing + KG Writeback + Ecosystem Tightening (2026-07-18)

**Feedback loop acceleration (SP-ECO-001):** LESSON collection upgraded with `urgency` field. P0 lessons trigger immediate notification (<1 minute vs 30-day monthly cron cycle). P1/P2 follow normal monthly review path.

**Bidirectional KG activation (SP-ECO-004):** Agent 3 `kg_enrichment` was read-only. Added `kg_writeback` field to propose new edges/nodes/updates discovered during analysis. Monthly cron Part D5 consumes `category:"kg"` lessons → high-confidence proposals auto-applied, medium/low flagged for review.

**Ecosystem coordination:** Monthly cron Part A now coordinates with weekly cron (new: `weekly regulation check` every Monday). solo-audit v5.6 deepened DI quality dashboard consumption (trend deviation detection).

**Principle:** An evolution ecosystem is not defined by its components — it is defined by the feedback loops between them. v2.5.1 shortens the DI→Cron→DI loop from 30 days to <1 minute for critical lessons, and makes the KG a living graph instead of a static snapshot.

### v2.5.0 — Full-Gate Pipeline (9/9) + Resilience + LESSON + Quality Dashboard + KG + Ruling Logic + Tuning (2026-07-18)

**The v2.5 milestone: 9-agent pipeline with 100% Schema Gate coverage.** Three-case CCDI live-fire validation confirmed all mechanisms.

**Structural hardening (P0):** Every agent file has `## Output Schema` → Gate B validates structure at every handoff. `merge.md` closes the last unprotected step (formerly inline, now a full agent). Schema rigidity fix: agent output templates aligned with Schema declarations, eliminating the most common failure mode (2 SCHEMA_FAILs in live test). Pipeline resilience: `pipeline_failure_log.json` + resume-from-checkpoint → 2 recoveries in live test without re-running completed agents.

**Knowledge refinement (P1):** Case-index.json upgraded to L1(tag)/L2(ruling_logic)/L3(difference) matching with `ruling_logic` (core_principle + distinction_criteria + applicable_when + not_applicable_when) for all 11 cases. Structured LESSON collection pipeline: `[LESSON]` → `_lessons.json` → monthly cron → Agent Tuning fields. Quality dashboard aggregates per-run metrics with trend detection.

**Ecosystem integration (P2):** Knowledge graph activation (55 nodes, 58 edges) → Agent 3 1-hop enrichment with `kg_enrichment` observability. Monthly cron Part C3: semi-automated case labeling + similarity matching + ruling_logic draft generation. All 9 agent files have `Execution Tuning` sections ready for lesson injection.

**Guard improvements (P3):** `scope.md` Step -1 WIKI_PATH check prevents silent degradation. Case ruling logic L2 matching enables logic-level (not just tag-level) case comparison.

**Principle:** A precision factory needs every handoff guarded, every signal collected, every lesson fed back. Quality is not a checkpoint — it is a continuous loop that starts at the first gate and never stops.

> Full changelog with file-level changes → `references/changelog.md`
