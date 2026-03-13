# Case Study 01 — GenAI-Powered Security Escalation Triage

**System Type:** Type B (Internal LLM Platform) / Type A (LLM-as-Feature)
**Domain:** Security Operations
**Scale:** Large engineering organisation (200+ security engineers)
**Outcome:** ~60% reduction in investigation time per escalation

---

## Background

Large engineering organisations with significant application surface area generate high volumes of security escalations — potential vulnerabilities, exception requests, bar review submissions, and anomaly alerts. Security engineering teams are expensive and scarce. The ratio of escalations to available security engineers creates a triage bottleneck: high volumes of incoming signals, limited expert capacity to evaluate them, and pressure to make fast risk decisions without sacrificing quality.

Traditional triage relied on senior security engineers manually reading each escalation, applying contextual knowledge, and routing it to the appropriate queue with a priority rating. This process was slow, inconsistent (different engineers applied different risk thresholds), and did not scale as the application portfolio grew.

This case study documents the security evaluation conducted for a GenAI-powered triage system designed to address this bottleneck.

---

## System Architecture

```
Escalation Input
      │
      ▼
  LLM Triage Engine ──── Context Retrieval (RAG)
      │                  └── Historical escalations
      │                  └── Security policy documentation  
      │                  └── Threat intelligence feeds
      ▼
  Structured Output
  ├── Risk classification (Critical/High/Medium/Low)
  ├── Routing recommendation (team/queue)
  ├── Key risk factors (plain language summary)
  ├── Similar historical escalations (retrieved)
  └── Confidence score
      │
      ▼
  Human Security Engineer Review ──── Accept / Override / Escalate
      │
      ▼
  Resolution and Feedback Loop
```

**Key design decision:** The LLM generates a recommendation with supporting rationale. A human security engineer makes the final routing decision. The LLM does not close, deprioritise, or take action on escalations autonomously — it accelerates the human decision, it does not replace it.

---

## Security Threats Identified — Pre-Deployment Evaluation

### Threat 1 — Prompt Injection via Escalation Content

**Risk:** An attacker submitting a security escalation could embed prompt injection payloads in the escalation content (description, code snippets, URLs) designed to manipulate the LLM's risk classification — for example, causing the LLM to systematically misclassify a real vulnerability as low risk, or to always recommend routing to a low-capacity queue.

**Risk Score:** High (8.5)
- Likelihood: 3 (authenticated internal users, but insider threat is plausible; technique is well-documented)
- Impact-Confidentiality: 2 (escalation content is internal but not highly sensitive)
- Impact-Integrity: 4 (systematic misclassification of security escalations is a critical integrity failure)
- Exploitability: 3 (technique is publicly documented and accessible to a motivated insider)

**Mitigations applied:**
- Output validation: LLM risk classification outputs are validated against a structured schema — the LLM cannot output free-form routing instructions, only structured classifications within predefined categories
- Human review gate: All critical and high classifications undergo mandatory human review before routing — a single LLM misclassification cannot cause a high-severity escalation to be silently deprioritised
- Input/output logging: Full logging of escalation content and LLM output for retrospective analysis
- Anomaly monitoring: Statistical monitoring of classification distribution — a sudden spike in "low risk" classifications triggers a review

---

### Threat 2 — Sensitive Data Exposure via RAG Retrieval

**Risk:** The RAG system retrieves from historical escalations, which may contain sensitive details — vulnerability specifics, customer data referenced in bug reports, internal system details. Retrieval could surface data to users who should not have access to it based on semantic similarity of their query.

**Risk Score:** High (8.0)
- Likelihood: 2 (authenticated internal users only; requires deliberate probing)
- Impact-Confidentiality: 4 (historical escalations can contain PII, vulnerability details, and system architecture information)
- Impact-Integrity: 2 (data exposure rather than action impact)
- Exploitability: 2 (requires knowledge of retrieval system behaviour and deliberate crafting)

**Mitigations applied:**
- Access-controlled retrieval: Historical escalation retrieval is filtered by the submitting user's team membership — engineers can only retrieve escalations from their own organisation
- Sensitivity-based retrieval exclusions: Escalations marked as critical severity or containing specific sensitivity flags are excluded from the retrieval corpus
- Retrieval logging: All retrieval queries and returned documents are logged for audit

---

### Threat 3 — Model Drift and Inconsistent Risk Calibration

**Risk:** Over time, if the LLM's risk calibration drifts (due to model updates, prompt engineering changes, or distribution shift in escalation content), the triage system could systematically over- or under-classify risks without the change being detected.

**Risk Score:** Medium (6.0)
- Likelihood: 3 (model updates happen; distribution shift is common over time)
- Impact-Confidentiality: 1 (no direct data exposure)
- Impact-Integrity: 3 (systematic miscalibration of security risk ratings affects the entire security function)
- Exploitability: 2 (not directly exploitable by external actors; operational risk rather than security risk)

**Mitigations applied:**
- Classification distribution monitoring: Weekly automated report comparing current classification distribution against 90-day baseline — significant deviation triggers review
- Ground truth comparison: Sample of LLM classifications reviewed by senior security engineers monthly; agreement rate tracked as a calibration metric
- Model version pinning: LLM model version is pinned; updates require a calibration validation exercise before deployment

---

### Threat 4 — Over-Reliance and Skill Atrophy

**Risk:** Security engineers who use the LLM triage tool for an extended period may lose the skill and judgment to evaluate escalations independently — creating a single point of failure if the system is unavailable.

**Risk Score:** Medium (5.0)
- Likelihood: 3 (well-documented phenomenon in automation-assisted decision making)
- Impact-Confidentiality: 1 (no direct data exposure)
- Impact-Integrity: 3 (degradation of human security judgment is a meaningful long-term risk)
- Exploitability: 1 (not directly exploitable; systemic risk)

**Mitigations applied:**
- Regular unaided evaluation exercises: Quarterly exercises where engineers evaluate a set of escalations without LLM assistance; results compared against LLM baseline
- LLM rationale transparency: The system is designed to show the LLM's reasoning, not just its recommendation — engineers are trained to evaluate the reasoning, not just accept the output

---

## Lessons Applicable to Similar Systems

**1. The most important architectural decision is where the human stays in the loop.**
Designing the human review gate — what the LLM can do autonomously vs. what requires human approval — is the single most important security design decision for any security operations LLM system. For this system, the LLM recommends; humans decide. That boundary should be explicit, monitored, and revisited as trust in the system builds.

**2. Injection risk in security tooling is high-value for attackers.**
A prompt injection that affects a customer support chatbot causes reputational harm. A prompt injection that systematically miscategorises security escalations affects the security posture of the entire organisation. Apply proportionally higher scrutiny to injection risk in security-adjacent LLM systems.

**3. Measure what the system is supposed to improve.**
The system was designed to reduce investigation time and improve routing consistency. Both are measurable. Measure them continuously — not just at launch. If the metrics degrade, it is an early signal of model drift, data quality issues, or system abuse.

**4. Build the feedback loop from day one.**
Every override of the LLM's recommendation by a human engineer is a data point. Capturing and analysing overrides is the most valuable signal for improving calibration and detecting emerging abuse patterns.

---

## Outcome

Post-deployment measurement at 90 days:
- Mean investigation time per escalation: reduced by approximately 60% (from ~5 hours to ~2 hours)
- Routing accuracy (human agreement with LLM routing recommendation): 87%
- Critical escalation misclassification rate: 0% (all critical escalations correctly flagged)
- Engineer-reported confidence in LLM recommendations: 4.1/5.0

The system continues to operate with quarterly calibration reviews and ongoing classification distribution monitoring.
