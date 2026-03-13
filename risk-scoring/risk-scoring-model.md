# LSEF Risk Scoring Model

**Purpose:** Produce a quantified, comparable risk score for each identified LLM security threat — enabling prioritised remediation rather than treating all risks as equally urgent.

**Design principle:** This model is deliberately simple. A scoring system that takes 2 days to apply provides less value than one that takes 20 minutes and gets used consistently.

---

## Scoring Dimensions

Each identified threat is scored across four dimensions. Each dimension is scored 1–4.

### Dimension 1 — Likelihood (L)

How likely is this threat to be exploited in your specific deployment context?

| Score | Level | Criteria |
|-------|-------|----------|
| 1 | Low | Requires advanced attacker with insider knowledge; no public exploit; limited access |
| 2 | Medium-Low | Requires skilled attacker; technique is known but not widely used; some access controls |
| 3 | Medium-High | Technique is publicly documented; accessible to motivated non-expert; limited controls |
| 4 | High | Trivially exploitable; public tools exist; no access controls; actively exploited in wild |

**Likelihood modifiers — increase by 1 if:**
- The LLM endpoint is publicly accessible without authentication
- The threat technique has been widely covered in security research in the last 6 months
- Your system type (agentic, RAG, multi-tenant) is the primary target for this technique

**Likelihood modifiers — decrease by 1 if:**
- Strong authentication and authorisation controls are in place
- The system is internal-only with limited user population
- Existing controls already substantially mitigate the attack path

---

### Dimension 2 — Impact — Confidentiality (IC)

What is the worst-case impact on data confidentiality if this threat is realised?

| Score | Level | Criteria |
|-------|-------|----------|
| 1 | Minimal | Only public or non-sensitive data exposed |
| 2 | Limited | Internal but non-regulated data exposed; limited blast radius |
| 3 | Significant | PII, credentials, or confidential business data exposed; moderate blast radius |
| 4 | Critical | Regulated data (HIPAA, PCI, GDPR), credentials with broad access, or IP of strategic value exposed |

---

### Dimension 3 — Impact — Integrity / Action (IA)

What is the worst-case impact on system integrity or from unintended actions?

| Score | Level | Criteria |
|-------|-------|----------|
| 1 | Minimal | LLM output quality degraded; no real-world actions affected |
| 2 | Limited | Incorrect information delivered; minor unintended actions easily reversible |
| 3 | Significant | Significant data modified or deleted; communications sent; service disrupted |
| 4 | Critical | Large-scale data destruction; external communications sent at scale; downstream system compromise |

---

### Dimension 4 — Exploitability (E)

How easy is it for an attacker to construct and execute an exploit?

| Score | Level | Criteria |
|-------|-------|----------|
| 1 | Difficult | Requires deep technical knowledge; no public tooling; significant effort |
| 2 | Moderate | Requires some technical skill; limited public documentation |
| 3 | Easy | Well-documented technique; tools or examples available publicly |
| 4 | Trivial | Public exploit code or step-by-step guides; no special skills required |

---

## Score Calculation

```
Raw Score = L × ((IC + IA) / 2) × E
```

| Raw Score Range | Risk Rating | Recommended Action |
|----------------|-------------|-------------------|
| 1.0 – 4.0 | 🟢 Low | Document and monitor; address in normal roadmap |
| 4.1 – 8.0 | 🟡 Medium | Address within 90 days; document interim controls |
| 8.1 – 12.0 | 🟠 High | Address before deployment or within 30 days post-deployment |
| 12.1 – 16.0 | 🔴 Critical | Block deployment or remediate within 14 days; escalate to security leadership |

---

## Scoring Template

Use this table to score each identified threat:

| Threat | Likelihood (L) | Impact-C (IC) | Impact-I (IA) | Exploitability (E) | Raw Score | Rating | Owner | Due Date |
|--------|---------------|---------------|---------------|-------------------|-----------|--------|-------|----------|
| Direct Prompt Injection | | | | | | | | |
| Indirect Prompt Injection (if RAG) | | | | | | | | |
| Context Window Extraction | | | | | | | | |
| Training Data Poisoning (if applicable) | | | | | | | | |
| Model Weight Exfiltration (if applicable) | | | | | | | | |
| Inference Attacks | | | | | | | | |
| Supply Chain Compromise | | | | | | | | |
| Sensitive Data Exfiltration | | | | | | | | |
| Excessive Agency (if agentic) | | | | | | | | |
| SSRF via Tool Use (if agentic) | | | | | | | | |
| Multi-Agent Trust Exploitation (if multi-agent) | | | | | | | | |

---

## Example Scoring — Customer-Facing RAG Chatbot

This example illustrates how to apply the model for a typical internal knowledge assistant deployed to external customers.

**System:** Customer-facing product support chatbot with RAG over public documentation and an authenticated user knowledge base. No tool use. Customers are authenticated.

| Threat | L | IC | IA | E | Score | Rating |
|--------|---|----|----|---|-------|--------|
| Direct prompt injection | 3 | 2 | 2 | 3 | 9.0 | 🟠 High |
| Indirect prompt injection via retrieved docs | 2 | 3 | 2 | 2 | 5.0 | 🟡 Medium |
| Context window extraction (system prompt) | 3 | 2 | 1 | 3 | 6.75 | 🟡 Medium |
| RAG over-retrieval (cross-tenant data access) | 2 | 4 | 1 | 2 | 5.0 | 🟡 Medium |
| Training data poisoning | 1 | 2 | 2 | 1 | 2.0 | 🟢 Low |
| Supply chain (model provider compromise) | 1 | 3 | 3 | 1 | 3.0 | 🟢 Low |
| Inference attacks | 2 | 2 | 1 | 2 | 3.0 | 🟢 Low |
| Jailbreak / safety bypass | 3 | 1 | 2 | 3 | 6.75 | 🟡 Medium |

**Result:** Direct prompt injection is the highest priority risk for this system. The recommended pre-deployment action is: implement output content validation, add injection monitoring, complete a red team exercise focused specifically on injection before launch.

---

## Calibration Notes

**This model will under-score some risks in specific contexts.** Common calibration adjustments:

- **Regulated data environments (HIPAA, PCI-DSS, FedRAMP):** Increase IC by 1 across all threats involving data exposure — regulatory penalties amplify actual impact
- **Agentic systems with broad tool access:** Increase IA by 1 across all injection-related threats — the blast radius of a successful injection is larger
- **High-profile / consumer-facing deployments:** Consider adding a reputational impact dimension — a successful jailbreak that goes viral has impact beyond the direct security harm
- **Internal-only systems with small user populations:** Decrease L by 1 across most threats — reduced access means reduced exploitation probability

The model is a tool for prioritisation, not a substitute for security judgment. Use it to structure the conversation, not to end it.
