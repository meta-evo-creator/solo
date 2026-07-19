---
name: "solo-audit"
description: "SOLO Audit v5.8 — Hermes-Native: delegate_task + QQ Bot + 进化指标量化 + 膨胀自动检测 + 特权环合规审计。⛔ Confidential."
platforms:
  - openclaw
  - hermes
tools:
  - read
  - exec
  - cron
  - memory_search
  - sessions_spawn
  - delegate_task

metadata:
  openclaw:
    emoji: 🔍
  hermes:
    emoji: 🔍
    pipeline: delegate_task
    delivery: native_qqbot
---

# SOLO 审计 Agent v5.8.0

> 协议规范。四智能体管线：收集者不验证，验证者不判决，分析者推提案。
> **v5.8 Hermes 655增强：** COLLECT新增进化指标+膨胀扫描 · ANALYZE新增特权环合规审计 · 655从设计原则升级为可量化活系统。
> v5.7 Hermes适配：`sessions_spawn` → `delegate_task` · `wecom_mcp` → QQ Bot原生交付。
> v5.6 新增：Agent 1 COLLECT深化DI质量仪表板消费。

---

## 0. 协议签名

```yaml
PROTOCOL: solo-audit v5.7.0
入口:    { trigger: 'cron' | 'direct', scope: AuditScope }
出口:    AuditResult | void(零违反) + 655Analysis | void(零违反)

┌──────────────────────────────────────────────────────────────┐
│ 平台适配层 (v5.7新增)                                        │
│                                                              │
│  OpenClaw:                    Hermes:                        │
│    sessions_spawn             delegate_task                  │
│    wecom_mcp 推送             QQ Bot 原生交付 (deliver:origin)│
│    session_status             无等效 (降级: 手动健康度估算)   │
│    toolsAllow                 无等效 (子代理全工具集)          │
└──────────────────────────────────────────────────────────────┘

外调度器 (主会话):
  模型: deepseek-v4-flash (Hermes) / deepseek/deepseek-v4-flash (OpenClaw)
  工具: [delegate_task, cron, read, terminal, file]
  职责: 确定scope → 顺序delegate 4个Agent → 聚合产出 → 交付
  交付协议 (v5.7 Hermes-native):
    1. 零违反 → 静默退出（cron deliver:origin 不做任何事）
    2. 有违反 → 最终回复即审计摘要（cron 自动推送到 QQ）
    3. P0 级提案 → 在最终回复中特别标注 ⚡ 引起注意
    ⚠️ 不再调用 wecom_mcp，不再使用 msg/doc/todo 品类

Agent 1 (COLLECT — 能层):
  模型: deepseek-v4-flash (Hermes) / deepseek/deepseek-v4-flash (OpenClaw)
  角色: 证据收集者，禁止判断
  工具: [read, terminal, memory_search, write]
  输出: RawEvidence → 直接写 memory/audit/evidence/ (不含violations/proposals)
  v5.2 新增:
    - 读取上次archive检查历史提案闭环状态
    - 收集每个cron最近5次运行的模型+error+duration
  v5.6 新增 — DI质量仪表板消费:
    - 读取 memory/inspection-drafts/_pipeline_quality_log.json（若存在）
    - 提取最近3次DI管线运行的 agent5_score + agent2_audit 状态
    - 计算趋势：当前分 vs 前3次平均分 → 偏离>15分 → 记录为 evidence 项
    - 若文件不存在 → 记录 `[DI_QUALITY_UNAVAILABLE]`，不视为violation
    - 输出到 evidence 文件时标注 `source: DI质量仪表板`
  v5.8 新增 — 655进化指标消费:
    - 读取 memory/.mev/evolution-metrics.json
    - 提取 closed_rate（闭环率）、auto_patches_applied（自动修复数）、regressions（回归数）
    - 闭环率 < 50% → 记录为 evidence 项 `[EVOLVE_STAGNATION]`
    - 回归数 > 0 → 记录为 evidence 项 `[REGRESSION_DETECTED]`
    - 文件不存在 → 标注 `[EVOLVE_METRICS_UNAVAILABLE]`
  v5.8 新增 — 膨胀自动检测消费:
    - 运行 `python scripts/expansion-scan.py`
    - 读取 memory/.mev/expansion-scan.json
    - overall_status=🔴 → 记录为 evidence 项 `[EXPANSION_CRITICAL]`
    - overall_status=🟡 → 记录为 observation
    - 逐条warnings纳入evidence

Agent 2 (VERIFY — 审层):
  模型: deepseek-v4-pro (强制，异构验证)
  角色: 交叉验证者，禁止修改文件
  工具: [read] (无terminal/write)
  ⛔ 硬约束 (v5.8新增): 若尝试写文件或执行命令 → 任务立即标记为 FAIL。Gate验证时检查上游文件修改时间戳未被变更。
  输出: Violations[] + Proposals[] + chain_of_custody

Agent 3 (JUDGE — 审层):
  模型: deepseek-v4-flash (Hermes) / deepseek/deepseek-v4-flash (OpenClaw)
  角色: 判决归档者，不能推翻VERIFY结论
  工具: [write] (无read/terminal)
  输出: archive.json + audit-summary.md（供外调度器使用）
  v5.7 变更: 不再直接推送。JUDGE写本地文件→外调度器聚合最终回复→cron自动交付
  输出文件:
    1. archive/audit-{date}.json（完整归档）
    2. delivery/audit-{date}-summary.md（摘要，外调度器将其作为最终回复）

Agent 4 (ANALYZE — 能层·v5.4新增):
  模型: deepseek-v4-flash (Hermes) / deepseek/deepseek-v4-flash (OpenClaw)
  角色: 655分析者，基于JUDGE产出做SOLO 655对标分析并生成高阶提案
  工具: [read, write] (只读archive，写analysis报告)
  输出: 655Analysis → 写 memory/audit/analysis/YYYY-MM-DD-655.json
```

---

## 1. 数据类型定义

```typescript
// ---- 原有类型不变（AuditScope, RawEvidence, Violation, Proposal）----

// v5.4 新增: 655分析结果
type Analysis655 = {
  source: 'agent4-analyze';
  date: string;
  audit_id: string;

  // 6铁律对标: 每条violation映射到铁律
  iron_laws: Array<{
    law: string;               // 隐私/溯源/免疫/权责/惜文件/谦逊
    violations: string[];       // 映射到这条铁律的violation IDs
    trend: '恶化' | '持平' | '改善'; // 相对于上次审计的趋势
    assessment: string;         // 1-2句话评估
  }>;

  // 5原则对标: violations是否属于膨胀陷阱等
  principles: Array<{
    principle: string;          // 膨胀陷阱(基石)/纯净/压缩/优先/降熵
    violations: string[];
    pattern_detected: boolean;  // 是否命中该原则的模式
    assessment: string;
  }>;

  // 5MEV过程评估: 审计循环的健康度
  mev_five: Array<{
    layer: string;              // Suit/Sense/Think/Optimize/Evolve
    status: '✅' | '⏸️' | '❌';
    assessment: string;
  }>;

  // 综合提案（高阶归并，基于V+Proposals提炼）
  strategic_proposals: Array<{
    id: string;                  // SP-001, SP-002...
    priority: 'P0' | 'P1' | 'P2';
    based_on: string[];          // 基于哪些 V-xxx / P-xxx
    recommendation: string;      // 一句建议
    action: string;              // 操作
    expected_impact: string;     // 预期效果
  }>;

  // 总体评级
  overall_health: '🟢健康' | '🟡需关注' | '🔴需紧急处理';
  one_line_summary: string;
}

type AuditResult = {
  audit_id: string;
  date: string;
  scope: AuditScope;
  model: string;
  evidence_sources: string[];
  delivery_failed?: boolean;
  alignment: {
    iron_laws: Array<{ law: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    mev_five: Array<{ layer: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    principles: Array<{ principle: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    governance: Array<{ dimension: string, status: '✅' | '⏸️' | '❌', note?: string }>;
  };
  governance_records: Array<{ operation: string, ring: number, ring_name: string, result: string, reason: string }>;
  violations: Violation[];
  positive_findings: string[];
  benchmark_sync: { loaded: boolean, matched: string[], missed: string[] };
  proposals: Proposal[];
  // v5.4: 挂载655分析引用
  analysis_655_path?: string;
}
```

---

## 2. 管线架构

```
                                  权层规则
                            (SOUL.md · solo · policy.json)
                                  │
                                  ↓
 外调度器 ─→ DELEGATE ─→ DELEGATE ─→ DELEGATE ─→ (有违反?) ─→ DELEGATE
     │           │           │           │           │            │
     ↓           ↓           ↓           ↓           ↓            ↓
  确定scope  Agent 1     Agent 2     Agent 3    静默退出     Agent 4
             COLLECT     VERIFY      JUDGE                  ANALYZE
             (能层)      (审层Pro)    (审层)                  (能层·v5.4)

  COLLECT出:     VERIFY出:      JUDGE出:        ANALYZE出:
  RawEvidence    Violations[]   archive.json     655Analysis
                 Proposals[]    +summary.md
```

**核心约束：**

| Agent | 层 | 模型 | 可写文件 | 可推送 | 可 delegate | 可 terminal |
|:------|:--:|:----:|:--------:|:------:|:--------:|:-------:|
| 1 COLLECT | 能层 | Flash | ✅ (仅 evidence/) | ❌ | ❌ | ✅ (只读命令) |
| 2 VERIFY | 审层 | **Pro** | ❌ | ❌ | ❌ | ❌ |
| 3 JUDGE | 审层 | Flash | ✅ (仅`memory/audit/`) | ❌ | ❌ | ❌ |
| **4 ANALYZE** | **能层** | **Flash** | ✅ **(仅`memory/audit/analysis/`)** | ❌ | ❌ | ❌ |

> **v5.7:** `delegate_task` 替代 `sessions_spawn`。子代理由外调度器顺序 delegate，每个完成后读回产出再启动下一个（因为存在数据依赖）。VERIFY 依赖 COLLECT 产出，JUDGE 依赖 VERIFY 产出，不能并行。

---

## 3. 各阶段详细定义

### 3.0 外调度器 (主会话) — v5.5 更新

```
INPUT:  AuditScope
ACTION:
  1. 确定scope：读取solo-audit SKILL.md + 当天日期
  2. 写入 scope.json 到 memory/audit/scope/YYYY-MM-DD.json
  3. SPAWN Agent 1 (COLLECT)
  4. WAIT → 读回 RawEvidence (只验证存在性，不读全文)
  5. 关键路径独立验证（v5.3）
  5a. 【v5.5新增】意图路径验证门禁（防false positive）:
      对Agent 1返回的RawEvidence中每条潜在violation路径，执行意图归属验证：
      - 读该路径文件的创建时间 → 是否属于当前执行周期内？
      - 读该路径内容的前3行 → 是否与当前审计scope定义的任务相关？
      - 若路径是历史残留（非当前执行周期产出，且与本次scope无关）→ 标记为 `[HISTORICAL_RESIDUE]`，不触发violation
      - 若路径影响不明 → 标记 `[UNCERTAIN_CONTEXT]` 降级为observation而非violation
     验证结果写入 `memory/audit/evidence/intent-verification.md`
  6. SPAWN Agent 2 (VERIFY)
  7. WAIT → 读回 violations + proposals
  8. 零违反 → 静默退出
     有违反 → SPAWN Agent 3 (JUDGE)
  9. WAIT → 读回 JUDGE 回传
 10. 判断 JUDGE 回传:
     if delivery_failed === true → 外调度器手动推送
     if type === 'approval_pending' → 记录等待审批
     if type === 'archive_done' → 确认归档
 11. 【v5.4新增】如果存在违反 → SPAWN Agent 4 (ANALYZE):
     sessions_spawn {
       prompt: "按§3.4执行，传入archive_path + findings_path",
       toolsAllow: [read, write]
     }
     ▼
 12. WAIT → 读回 655Analysis
 13. 推送最终分析摘要到用户：
     "📊 655分析: 🟢/🟡/🔴 | 6铁律 X过Y失 | 5原则 Z项异常 | SP-N项提案"
     每项strategic_proposal一行输出
     ▼
OUTPUT: AuditResult + 655Analysis
```

### 3.1 COLLECT — 证据收集 (Agent 1)

v5.8新增收集项：进化指标（evolution-metrics.json）+ 膨胀扫描（expansion-scan.py → expansion-scan.json）。

v5.6新增收集项：DI质量仪表板。读取 `memory/inspection-drafts/_pipeline_quality_log.json`，提取最近3次运行评分，计算趋势偏离度。偏离>15分作为潜在evidence记录。文件不存在时标注 `[DI_QUALITY_UNAVAILABLE]`。

### 3.2 VERIFY — 交叉验证 (Agent 2)

在形成violation前，必须对每项潜在违反执行意图归属判断：
- 读证据文件创建时间 → 是否在当前执行周期内
- 读文件内容前3行 → 是否与当前审计scope定义的任务路径相关
- 若为历史残留 → 标记 `[HISTORICAL_RESIDUE]`，不输出violation
- 若影响不明 → 降级为observation输出，不产violation

> 意图验证的初步筛选由外调度器Step 5a执行，VERIFY在此基础之上做二次确认。

### 3.3 JUDGE — 判决归档 + 审批路由 (Agent 3) [同v5.3，无变化]

### 3.4 ANALYZE — 655分析 (Agent 4 · v5.4 新增)

```
INPUT:  archive_path (指向 memory/audit/archive/audit-YYYY-MM-DD.json)
        findings_path (指向 memory/audit/findings/YYYY-MM-DD-findings.json)
ACTION: 只读分析，输出655对标报告:

  1. 读 archive 全量文件（AuditResult + Violations + Proposals）

  2. 6铁律对标分析:
     逐条检查 violations 映射到哪条铁律:
     ① 隐私红线 — 数据外传、第三方API、日志泄露
     ② 溯源为本 — 证据链完整性、来源标注
     ③ 免疫注入 — 外部输入覆盖、越狱尝试
     ④ 权责两清 — 未授权操作、权限越界、审批绕过 + **v5.8: 特权环合规**
     ⑤ 惜文件如命 — 临时文件清理、orphan cleanup
     ⑥ 谦逊为本 — 越界推断、无依据断言

     v5.8 新增 — 特权环合规子审计 (铁律④):
       检查过去24小时内所有 skill_manage 操作:
       - 读取 _auto_fix_log.json（DI Phase 8 产出）
       - 交叉验证：auto-fix 是否仅修改 agents/*.md？
       - 是否有 Ring 2/3 级操作未走确认流程？
       - 输出: ring_compliance: { violations: [], total_ops: N, compliant: N }
     
     输出每条铁律的:
     - 命中的violation列表
     - 相对于上次archive的趋势（恶化/持平/改善）
     趋势判断方法: 读上次 archive (memory/audit/archive/ 中最近一个非本次文件)
       比较同一铁律的 violation 数量和严重度变化

  3. 5原则对标分析:
     三条输出原则 + 一条基石:
     ① 膨胀陷阱(基石) — 是否在执行层问题出现时选择了加新规则而非修执行
     ② 纯净 — violation是否来自设计缺陷而非执行偏差
     ③ 压缩 — 机制存在但不执行, 或执行了但效果不足
     ④ 优先 — P0/P1项是否优先处理, 跨期未闭环
     ⑤ 降熵 — 主会话膨胀率、子会话摘要控制
     
     输出每条原则:
     - 是否命中 pattern_detected (bool)
     - 具体评估

  4. 5MEV过程评估:
     ① Suit(准备) — scope定义是否精准, Step 0b是否执行
     ② Sense(收集) — 证据覆盖面, 路径多样性
     ③ Think(分析) — 根因分析深度, 发现关联性
     ④ Optimize(交付) — 推送成功率, 闭环率
     ⑤ Evolve(进化) — lessons积累, 跨期改善

  5. 综合提案生成:
     基于V+Proposals归并提炼为高阶行动计划:
     - 归并: 多violation相同根因→一条SP
     - 精简: 合并重复提案P
     - 分类: 6铁律类 / 5原则类 / MEV过程类

     格式:
     ```
     SP-001 [P0] [6铁律] 基于V-F001+V-F002 — 恢复ENTROPY_VIOLATION标记活性
     → 操作: COLLECT协议增加rg检测 → 自动联动审计
     → 预期: 标记从装饰品变监控工具
     ```

  6. 总体评级:
     🔴 需紧急处理: 存在高危violation + 跨期恶化 + 闭环率<50%
     🟡 需关注: 存在中危violation + 持平趋势
     🟢 健康: 仅低危violation + 改善趋势 + 闭环率>80%

  7. 生成一句话总结:
     格式: "本次审计发现N项违反, M项跨期未闭环, 核心问题是XXX"

OUTPUT: 写入 memory/audit/analysis/YYYY-MM-DD-655.json
        + 通过推送最终分析摘要

RULES:
  - 【核心】禁止修改 archive/ 和 findings/ 的内容
  - 【核心】禁止推翻 JUDGE 的判决结论
  - 【核心】只能写 analysis/ 目录
  - 【核心】趋势比较必须基于上次真实archive数据，不能猜
```

---

## 4. 问句表（对照检查卡）— v5.4 新增 655分析问句

### 4.1 铁律问句表 [同v5.1]

### 4.2 MEV问句表 [同v5.1]

### 4.3 原则问句表 [同v5.1]

### 4.4 655分析问句（v5.4新增 · Agent 4专用）

| # | 维度 | 问句 |
|:-:|:-----|:-----|
| 1 | 铁律 | 这些violation主要违反了哪几条铁律？趋势是恶化还是改善？ |
| 2 | 原则 | 是否检测到膨胀陷阱模式？压缩/降熵原则是否有新违反？ |
| 3 | MEV | 审计产出的质量在五层中哪一层最弱？Evolve层是否有闭环？ |
| 4 | 综合 | 能否将N条violations归并为M=⌈N/2⌉条高阶建议？有哪些是跨期重复的？ |

---

## 5. 异常处理 — v5.4 更新

| 异常 | 处理 | 输出 | 责任Agent |
|:-----|:-----|:-----|:---------:|
| `openclaw doctor` fail | 终止整个管线 | `[框架异常]` | 外调度器 |
| COLLECT目标文件不存在 | 标记`null`+继续 | RawEvidence中标注 | Agent 1 |
| 失败模式清单.json缺失 | 跳过基准比对+继续 | `[基准集缺失]` | Agent 2 |
| archive.json 写入失败 | 降级到本地保存 | `[归档降级: 手动迁移]` | Agent 3 |
| AUDIT_LEARNINGS.md 提案写入冲突 | 追加新段落 | 无 | Agent 3 |
| 推送失败 | 标记delivery_failed → 外调度器兜底 | `delivery_failed: true` | Agent 3 + 外调度器 |
| **analysis目录不存在** | **自动创建 `memory/audit/analysis/`** | **无** | **Agent 4** |
| **上次archive不存在（首次审计）** | **趋势标记为'首次审计·无基线'** | **trend: 'baseline'** | **Agent 4** |
| Agent spawn 超时(>120s) | 重试一次，失败后跳过 | `[AGENT_FAILED]` | 外调度器 |

---

## 6. 谦抑约束（各Agent工具隔离）

| Agent | 层 | 可读范围 | 可写范围 | 可推送 |
|:------|:--:|:---------|:---------|:-------|
| 1 COLLECT | 能层 | 全workspace（只读命令） | memory/audit/evidence/ | ❌ |
| 2 VERIFY | 审层 | 全workspace（只read） | ❌ | ❌ |
| 3 JUDGE | 审层 | ❌ (无read/exec) | memory/audit/archive/ | ✅ QQ Bot |
| **4 ANALYZE** | **能层** | **memory/audit/archive/ + memory/audit/findings/** | **memory/audit/analysis/** | ✅ **QQ Bot** |

---

## 7. 版本记录

| 版本 | 日期 | 变更 |
|:-----|:-----|:------|
| **v5.8** | **2026-07-19** | **655体系三大增强: Agent 1 COLLECT新增进化指标+膨胀扫描消费 · Agent 4 ANALYZE新增特权环合规子审计(铁律④) · 655从设计原则升级为可量化活系统。** |
| **v5.6** | **2026-07-18** | **Agent 1 COLLECT新增DI质量仪表板消费——读取_pipeline_quality_log.json提取最近3次运行趋势偏离度，纳入evidence收集。** |
| **v5.5** | **2026-07-09** | **§3.0 Step 5a新增意图路径验证门禁 + §3.2 VERIFY二次确认，降低审计false positive率。** |
| **v5.4** | **2026-07-04** | **新增Agent 4 ANALYZE — 审计结果按SOLO 655（6铁律+5原则+5MEV）对标分析，归并violations + proposals为高阶战略提案。管线从3 Agent扩展为4 Agent。** |

## ⛔ 禁令清单
以下事项明确禁止作为proposal提出，违背视为`[BLOCKED_PROPOSAL]`：
- Memory索引重建（SP-001） — 不使用OpenAI key，不需要memory_search功能
| **v5.3** | **2026-06-28** | **COLLECT/VERIFY/调度器: WIKI规范路径来自`openclaw wiki status`而非硬编码。消除[COLLECT_PATH_ERROR]—Agent猜路径导致的false positive。不枚举备选路径（无3个备选，框架只定义一个规范位置）。** |
| **v5.2** | **2026-06-27** | **JUDGE自检段+推送失败降级兜底; Agent 1新增历史提案闭环检查+模型稳定性数据收集; 审批项输出精简为一行操作+目标** |
| v5.1 | 2026-06-23 | 审批路由+scope_ring+三段式推送 |
| v5.0 | 2026-06-22 | 管线化重构：COLLECT→VERIFY→JUDGE |
| v4.7 | 2026-06-22 | 修复§3.2a重复存储诊断错误 |
