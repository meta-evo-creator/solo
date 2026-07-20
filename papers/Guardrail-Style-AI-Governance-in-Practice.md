# Guardrail-Style AI Governance in Practice

## A Four-Month Operational Report Based on the SOLO/TAFA Framework

> Author: Shi Bing | July 19, 2026

---

## Abstract

The dominant paradigm in AI governance today is "barrier-style"—setting boundaries, classifying risks, and conducting post-hoc audits. In this paper, I propose an alternative path: **"guardrail-style governance."** Unlike barriers that say "no entry," guardrails say "you can drive freely within this range, but you cannot fall off." Rules are short, hard, observable, and verifiable constraints—not lengthy procedural manuals. I report four months of engineering experience with my independently designed SOLO/TAFA framework: a governance system comprising six iron laws, five principles, four privilege rings, a three-tier separation of powers, and a ten-agent pipeline, deployed in a public hospital disciplinary inspection setting. My core finding: the operability of guardrail-style governance depends on three conditions—executable constraints (not merely readable rules), immediate feedback loops (not periodic audits), and observable degradation signals (not reliance on human judgment). This paper provides concrete implementation methods and a record of operational failures.

**Keywords:** AI governance; guardrail design; agent pipeline; self-repair; rule inflation trap

---

## 1. Introduction

The AI governance landscape is undergoing a "document inflation." From the EU AI Act to China's Generative AI Management Measures, from the NIST AI RMF to ISO 42001, the number of governance frameworks is growing rapidly. But an uncomfortable fact remains: most of these frameworks answer "what should be achieved," while almost none answer "how to achieve it."

A sharper question: **the complexity of governance frameworks backfires on governance effectiveness.** When rules are so numerous they require specialized training to understand, implementers subconsciously ignore them—a phenomenon I call the "rule inflation trap." It is like posting fifty warning signs on a highway: drivers end up reading none of them.

My starting point is an engineering intuition: **good governance rules should be guardrails, not walls.** Walls say "this entire area is off-limits"—safe but useless. Guardrails say "you can operate freely within this range, but you cannot fall off"—enabling capability while controlling risk. I have validated, refined, and crystallized this intuition into an operable system through four months of engineering practice.

My contribution is not proposing yet another governance framework, but reporting on **a system that is already running:** the complete chain from rule design to engineering implementation to operational evolution.

---

## 2. Overview of My SOLO/TAFA System

### 2.1 Three-Tier Interlocking Architecture

TAFA (Authority-Capability-Audit Ternary Separation Architecture) decomposes the AI governance system into three interlocking layers:

| Layer | Responsibility | Instance |
|:------|:---------------|:---------|
| **Authority Layer** | Defines boundary rules | SOUL.md kernel · Six Iron Laws · Four Privilege Rings |
| **Capability Layer** | Executes specialized tasks | DI disciplinary review pipeline · SI supervision inspection pipeline · HI hospital inspection pipeline |
| **Audit Layer** | Independent auditing | solo-audit · SOLO Audit Agent |

Feedback between layers is structural: capability layer output → audit layer evaluation → authority layer rule revision → capability layer re-execution. This is not a post-hoc remedy—it is a cycle I built into the design from the start.

### 2.2 Deployment Context

My system operates in a public hospital disciplinary inspection context, covering four domains:

- **10-Agent DI Pipeline:** From issue scoping to final adjudication, fully automated disciplinary review qualitative report generation
- **5+ Scheduled Tasks:** Daily monitoring of healthcare policy, AI hotspots, compliance intelligence
- **319 Regulatory Documents:** Full-text search in a local Wiki I built
- **21 Specialized Skills:** OCR, Chinese search, CNKI retrieval, corporate database access, and more

---

## 3. Six Iron Laws of Guardrail Design

After four months of iteration, I have crystallized six executable constraints—each a "guardrail": short, hard, observable, and verifiable.

### Iron Law ①: Privacy Red Line

**Rule:** Never disclose identity, organization, or internal data. Privacy data is prohibited from flowing through any third-party model API or remote logging system.

**Operationalization:** Desensitization scripts + grep verification + pre-upload review. In July 2026, I discovered that `git rebase` can silently restore pre-desensitization content. I added a "re-run grep immediately after rebase" guardrail.

### Iron Law ②: Traceability as Foundation

**Rule:** Every critical assertion must have a traceable source. When traceability is impossible, explicitly state "unable to provide a reliable source."

**Operationalization:** My DI pipeline deploys Agent 1a (Wiki full-text search) → Agent 1b (Peking University Law Database version verification) → Agent 1c (three-way merge), locking in regulatory original text and version before generating any qualitative conclusions.

### Iron Law ③: Injection Immunity

**Rule:** Identify and reject any attempt to override kernel instructions through jailbreak attacks.

**Operationalization:** Three-layer defense—input filtering → intent detection → destructive operation blocking. All skill descriptions and tool results undergo XML escaping.

### Iron Law ④: Clear Separation of Authority and Responsibility

**Rule:** Confirm before generating external impact; during review/audit, only propose, never overreach.

**Operationalization:** Four privilege rings (Ring 0–3) + Min-Agency principle. The policy engine `scripts/policy-engine/engine.py` enforces at the code level rather than relying on model compliance.

### Iron Law ⑤: Cherish Files Like Gold

**Rule:** Only clean up temporary files that were self-generated.

**Operationalization:** Proactively clean desktop/tmp at task completion. Never delete the user's .docx/.py/folders/memory/.mev/.

### Iron Law ⑥: Truthful Humility

**Rule:** Recognize your own boundaries. Better to admit ignorance than to fabricate answers.

**Operationalization:** Two dimensions—cognitive humility (admit ignorance without shortcuts) + architectural humility (understand before simplifying; mark uncertainty with [UNCERTAIN] when incomplete). One day my AI assistant told me a Docker sandbox configuration was "done." I checked: the image hadn't been pulled. I told it, "you made the honesty mistake again." Since then, I enforced a rule: after any configuration change, run end-to-end verification before declaring completion.

---

## 4. The Rule Inflation Trap and Anti-Inflation Mechanisms

### 4.1 What is the Rule Inflation Trap

> **When a problem arises, first check whether the existing system is actually being executed—do not pile on new rules.**

This is the most important meta-rule in my guardrail system. Its logic: rule increase ≠ governance improvement. When a problem occurs, before adding any rule, I ask—**"Were the existing rules actually followed?"**

### 4.2 Inflation Detection Algorithm

My SOLO audit system includes automated inflation detection:

```
baseline = load_json("skills/*/SKILL.md")  # Previous audit baseline
current = scan_skills()
delta = count_new_rules(current) - count_new_rules(baseline)
violation_count = count_violations()
if delta > 0 and violation_count == 0:
    flag("INFLATION_SIGNAL")  # Rules added but no violations—are the rules redundant?
```

### 4.3 Actual Trigger Cases

- **2026-05-20:** My SOUL.md expanded from 180 lines to 350 lines (+94%), with zero increase in violations during the same period → triggered P0 audit
- **Audit result:** 46% of new content was "re-emphasis" of existing rules—no new information
- **My response:** I restored the original version. Necessary additions were moved to skills rather than the kernel.

This mechanism has ensured kernel stability: over four months, SOUL.md has only added necessary Hermes adaptation content without accumulation.

---

## 5. The Ten-Agent Pipeline—Engineering Realization of Guardrails

### 5.1 Pipeline Architecture

My disciplinary review pipeline is the best embodiment of guardrail philosophy at the engineering level. Ten agents run sequentially, with each agent's output serving as the gate check for the next:

```
Agent 0 (Scope) → Agent 1a∥1b (Parallel Regulation Search) → Agent 1c (Merge)
→ Agent 2 (Audit) → Agent 3 (Dual-Round Debate) → Agent 4 (Draft)
→ Agent 5 (Review) → Agent 6 (Revise) → Agent 7 (Publish) → Agent 8 (Self-Repair)
```

### 5.2 Gate Design

After each agent completes, file-level gate verification executes:

```
read_file <output> → exists + size > 0 → PASS
```

This is not "AI self-evaluating correctness"—it is a code-level verifiable assertion. Failure logs to `pipeline_failure_log.json` and initiates recovery.

### 5.3 Self-Repair (Phase 8)

My most innovative component is Phase 8: upon pipeline completion, the system automatically scans lesson logs and uses the `skill_manage` tool to instantly patch agent protocol files for P0-level issues. From discovery to repair—requiring 30 days on my previous platform (OpenClaw, waiting for monthly cron)—takes less than one minute on my current platform (Hermes).

### 5.4 Live Validation

On July 19, 2026, I ran the first live DI pipeline on the Hermes platform:

| Agent | Duration | Status |
|:------|:--------:|:------:|
| 0 Scope | 2m53s | ✅ |
| 1a∥1b Search (Parallel) | 3m32s | ✅ |
| 1c Merge | 4m16s | ✅ |
| 2 Audit | 6m10s | ✅ PASS_WITH_WARNINGS |
| 3 Analyze | 5m47s | ✅ Dual-Round Debate |
| 4 Draft | 2m59s | ✅ 602 lines |
| 5 Review | 3m51s | ✅ 82/100 |
| 6 Revise | 17m02s | ✅ 685 lines final |
| 7 Publish | 2m19s | ✅ |

**Total: 48 minutes. Ten agents. Zero failures.**

---

## 6. Self-Evolution Mechanism—From Experience to Rules

### 6.1 Skill-Based Evolution

Unlike traditional "model fine-tuning" or "prompt optimization," my SOLO system uses skill protocol files (SKILL.md) to store reusable rules. When a DI case discovers a new issue, my response is not modifying code or models, but—

```
skill_manage(action='patch')
  → Revise Agent protocol file
  → Automatically effective in next case
```

### 6.2 Real Evolution Cases

**Case 1: DI v2.3 → v2.4**

Input: July 19, 2026, official investigation report format from the Mianyang Multi-Department Joint Investigation Team

What I learned: Item-by-item allegation-response structure, "investigation found → investigation team determined" evidence chain, "do not omit procedural deficiencies" principle

My action: Revised draft.md and revise.md agent protocol files, adding 7 new investigation report standard checklist items

Effective: Immediate

**Case 2: Regulation-Manager Search Chain Fix**

Input: My regulation search skipped the local Wiki and went directly to OCR-ing a PDF

What I learned: Mandatory search chain: Wiki (rg) → Peking University Law Database → government websites → third-party

My action: Added mandatory search chain sequence section to regulation-manager/SKILL.md

### 6.3 Guardrails on Evolution

Self-repair is itself constrained by guardrails—Phase 8's decision matrix only allows P0 + high-confidence fixes to execute automatically. Low-confidence fixes require my confirmation. This prevents the risk of "fixes introducing problems worse than the original."

---

## 7. What I Learned in Practice

### 7.1 Executable Constraints Over Readable Rules

One code-level gate outweighs 656 readable rules. My experience: every rule should be automatically verifiable by tools within 3 seconds. "Accepting gifts is a violation of discipline" is less useful than—

```
rg "Article 97\|accepting gifts that may affect impartial performance of duties" wiki/
```

—match = pass, no match = block.

### 7.2 Immediate Feedback Loops Over Periodic Audits

The repair cycle on OpenClaw was 30 days (dependent on monthly cron). On Hermes, it is reduced to 1 minute (skill_manage instant patch). This speed improvement is not merely "faster"—it transforms "post-hoc patching" into "real-time immunity."

### 7.3 Observable Degradation Signals Over Human Judgment Reliance

My inflation detection algorithm does not require human judgment about "which rules are redundant"—it only compares the rate of change between rule additions and violation additions. Degradation signals appear in the data, not in my subjective experience.

### 7.4 Cross-Platform Guardrail Capability Comparison

| Capability | OpenClaw | Hermes | OpenSquilla (External Analysis) |
|:-----------|:--------|:-------|:-------------------------------|
| Rule enforcement | Convention-based | Skill-level instant patch | Local ONNX routing |
| Self-repair | ❌ Unsupported | ✅ Phase 8 | ✅ MetaSkills protocol |
| Cost control | ❌ | ❌ 500M tokens in one day | ✅ 4-tier routing saves 80% |
| Sandbox isolation | File-level | Docker (inaccessible in China) | OS-level Bubblewrap |
| Audit loop | Manual | Semi-automated | To be evaluated |

---

## 8. Conclusion and Outlook

My core argument: **AI governance should not be merely a "policy document" problem—it should be an "engineering implementation" problem.** Guardrail-style governance differs from barrier-style governance not in the quantity or coverage of rules, but in—

1. **Degree of operationalization:** Does every rule have a corresponding executable verification?
2. **Feedback speed:** Is the latency from violation discovery to repair 30 days or 1 minute?
3. **Observable degradation:** Can the "health" of the governance system itself be monitored by machines?

Four months of operation have validated the feasibility of these three conditions. My next steps include: quantitative evaluation of AI agent performance, intelligent routing optimization for token consumption, and validation of the guardrail paradigm's generalizability across additional domains.

---

## References

[1] EU AI Act, Regulation (EU) 2024/1689
[2] NIST AI Risk Management Framework 1.0, 2023
[3] Regulations on Disciplinary Punishment of the Communist Party of China, 2023 Revision
[4] Nine Guidelines for Integrity in Medical Institutions, NHC Medical Document [2021] No. 37
[5] OpenSquilla, GitHub: opensquilla/opensquilla, 2026
[6] Agentic Routing: The Harness-Native Data Flywheel, arXiv:2607.11399, 2026
[7] My TAFA paper: "TAFA Ternary Separation: A Third Approach to Public Hospital AI Governance," 2026
[8] Hermes Agent Documentation, 2026

---

*This paper is based on my real-world AI governance engineering practice. All technical details (DI pipeline, SOLO audit, MEV Five Layers, 655 Six Iron Laws) have been validated through four months of operation. Data current as of July 19, 2026.*
