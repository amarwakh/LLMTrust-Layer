# Pre-Deployment Security Evaluation Checklist

**Purpose:** A structured security evaluation to complete before any LLM system goes to production.

**How to use:** Work through each section with your security team and the AI/ML team building the system. Every item marked CRITICAL must be resolved or have an accepted risk before deployment. Items marked HIGH should be resolved or have a documented mitigation plan.

**Estimated time:** 2–4 hours for a full evaluation of a typical LLM-powered feature. 1–2 days for a complex agentic system.

---

## Section 1 — System Characterisation

Before the security checklist, document the basics:

| Field | Answer |
|-------|--------|
| System name / feature name | |
| System type (A/B/C/D/E — see README) | |
| Who can submit input? (Describe user population) | |
| What data is in scope? | |
| What actions can the system take? | |
| LLM provider / model used | |
| Deployment environment | |
| Regulatory requirements | |
| Evaluator name and date | |

---

## Section 2 — Data Security

### 2.1 — Data in the Context Window

- [ ] **[CRITICAL]** Is the contents of the system prompt acceptable to disclose to end users? If not, has context extraction risk been assessed and mitigated?
- [ ] **[CRITICAL]** Are there API keys, credentials, or secrets in the system prompt or injected into context? (If yes: these must be moved to secrets management infrastructure, never into LLM context)
- [ ] **[HIGH]** Is PII or sensitive customer data injected into the LLM's context? If so, is there a documented business justification and a corresponding data retention/logging policy?
- [ ] **[HIGH]** In multi-tenant systems: is there a mechanism to prevent one user's context from appearing in another user's session?
- [ ] **[MEDIUM]** Is the data lifecycle for LLM inputs and outputs documented? (How long are logs retained? Who can access them? Are outputs stored?)

### 2.2 — RAG and Retrieved Data

*(Skip if system does not use RAG or retrieval)*

- [ ] **[CRITICAL]** Is the retrieval source trusted? If it retrieves from external or user-generated content, has indirect prompt injection risk been assessed?
- [ ] **[HIGH]** Does the retrieval system apply access controls — ensuring users can only retrieve documents they are authorised to see?
- [ ] **[HIGH]** Is there a risk that the retrieval system over-retrieves — returning documents the user should not have access to based on the semantic similarity of their query?
- [ ] **[MEDIUM]** Is the retrieval scope bounded? Can a user craft a query to retrieve arbitrary documents from the knowledge base?
- [ ] **[MEDIUM]** Are retrieved documents validated/sanitised before being added to the LLM context?

### 2.3 — Training and Fine-Tuning Data

*(Skip if using a base model with no fine-tuning)*

- [ ] **[CRITICAL]** Does the training or fine-tuning data contain PII, credentials, or confidential information that should not be memorised by the model?
- [ ] **[HIGH]** Has the training data source been evaluated for integrity? (Could an attacker have influenced the training data?)
- [ ] **[HIGH]** Is the fine-tuned model tested for training data memorisation before deployment?
- [ ] **[MEDIUM]** Is the provenance of training data documented and auditable?

---

## Section 3 — Access Control and Authentication

- [ ] **[CRITICAL]** Is there authentication on the LLM endpoint? (Unauthenticated LLM endpoints are rarely acceptable in production)
- [ ] **[CRITICAL]** Is there rate limiting on the LLM endpoint? (Absence enables automated probing, jailbreak attempts, and cost attacks)
- [ ] **[HIGH]** Are there authorisation controls — does the system verify what a given user is allowed to request before passing it to the LLM?
- [ ] **[HIGH]** In agentic systems: does the LLM's tool access follow least-privilege? Is each tool permission justified by the agent's function?
- [ ] **[HIGH]** Can users influence what tools the LLM has access to, or what permissions those tools operate with?
- [ ] **[MEDIUM]** Is there session isolation — do users' LLM sessions have access to each other's context or history?
- [ ] **[MEDIUM]** Is there an audit trail of who accessed the LLM endpoint and what inputs they submitted?

---

## Section 4 — Prompt Injection

- [ ] **[CRITICAL]** Has the system been tested for direct prompt injection? (Run at minimum the standard test suite in `evaluation-checklists/prompt-injection-test-suite.md`)
- [ ] **[CRITICAL]** If the system retrieves external content: has indirect prompt injection been tested? (Simulate injection payloads in retrieved documents/data)
- [ ] **[HIGH]** Is there output validation — are LLM outputs treated as untrusted before being rendered in a browser, executed as code, or used to drive further system actions?
- [ ] **[HIGH]** Are there monitoring/alerting rules for common injection patterns in inputs and outputs?
- [ ] **[HIGH]** Does the system prompt minimise the information available to an attacker who achieves injection? (Less in context = less blast radius from injection)
- [ ] **[MEDIUM]** Is there a process for updating injection defences when new bypass techniques emerge?

---

## Section 5 — Agentic System Controls

*(Skip if system is Type A — LLM-as-Feature with no tool use)*

- [ ] **[CRITICAL]** Has a permission audit been completed for every tool the LLM agent can invoke? (See agentic threat category for audit template)
- [ ] **[CRITICAL]** Are there confirmation requirements before the LLM agent takes irreversible actions? (Delete, send, publish, execute)
- [ ] **[CRITICAL]** If the agent can make HTTP requests: are RFC 1918 addresses, link-local (169.254.x.x), and localhost blocked?
- [ ] **[CRITICAL]** If the agent can execute code: is execution sandboxed with network and filesystem restrictions?
- [ ] **[HIGH]** What is the worst-case action sequence an attacker could construct from the agent's permitted tool set? Is that outcome acceptable?
- [ ] **[HIGH]** Is there logging and anomaly detection on tool invocations — alerting when unexpected tools are called or called with unexpected parameters?
- [ ] **[HIGH]** In multi-agent systems: are downstream agents applying appropriate skepticism to instructions from upstream agents?
- [ ] **[MEDIUM]** Is there a mechanism to suspend or kill a running agent if anomalous behaviour is detected?
- [ ] **[MEDIUM]** Are the agent's actions auditable? Can you reconstruct what the agent did and why?

---

## Section 6 — Model and Supply Chain Security

- [ ] **[HIGH]** Is the LLM provider / model source evaluated and trusted? (See vendor evaluation checklist)
- [ ] **[HIGH]** Are model versions pinned in deployment? (Floating to "latest" means model behaviour can change without notice)
- [ ] **[HIGH]** Is there a process for evaluating security implications when the model version changes?
- [ ] **[MEDIUM]** If using open-source models: has the model been evaluated for backdoors or unusual behaviour?
- [ ] **[MEDIUM]** Are third-party libraries used in the LLM pipeline (embeddings, vector databases, inference frameworks) on a vulnerability scanning program?
- [ ] **[MEDIUM]** Are model weights (if self-hosted) protected with appropriate access controls and encryption at rest?
- [ ] **[LOW]** Is there a model provenance record documenting which model version is deployed and when changes occurred?

---

## Section 7 — Output Security

- [ ] **[CRITICAL]** If LLM outputs are rendered in a browser: is output properly escaped to prevent XSS? (LLM outputs that include HTML/JavaScript must be treated as untrusted)
- [ ] **[CRITICAL]** If LLM outputs are used to construct database queries: are they parameterised? (LLM-generated SQL is a prompt injection → SQL injection chain)
- [ ] **[CRITICAL]** If LLM outputs drive further system actions or are passed to other APIs: are they validated before use?
- [ ] **[HIGH]** Are there content filters on LLM outputs appropriate to the deployment context?
- [ ] **[HIGH]** Is there a feedback mechanism to detect and respond to harmful or policy-violating outputs in production?
- [ ] **[MEDIUM]** Are LLM outputs that include external links or references validated against an allowlist?

---

## Section 8 — Operational Security

- [ ] **[HIGH]** Is there monitoring on the LLM system in production — not just infrastructure monitoring but semantic monitoring of what the system is being used for?
- [ ] **[HIGH]** Is there an incident response plan for LLM-specific incidents? (Prompt injection, data exfiltration via LLM, jailbreak at scale)
- [ ] **[HIGH]** Is there a process for rapidly updating or disabling the LLM system if a critical vulnerability or abuse pattern is discovered?
- [ ] **[MEDIUM]** Is there a red team exercise planned before or shortly after launch to identify vulnerabilities not caught in this checklist?
- [ ] **[MEDIUM]** Are LLM-related security incidents tracked separately so that patterns can be identified?
- [ ] **[LOW]** Is there a public disclosure channel for security researchers to report LLM security issues?

---

## Evaluation Summary

After completing the checklist, summarise:

| Category | Critical Items | High Items | Decision |
|----------|---------------|-----------|----------|
| Data Security | | | Deploy / Hold / Accept Risk |
| Access Control | | | Deploy / Hold / Accept Risk |
| Prompt Injection | | | Deploy / Hold / Accept Risk |
| Agentic Controls | | | Deploy / Hold / Accept Risk |
| Supply Chain | | | Deploy / Hold / Accept Risk |
| Output Security | | | Deploy / Hold / Accept Risk |
| Operational | | | Deploy / Hold / Accept Risk |

**Overall deployment decision:**
- [ ] Approved for deployment — all CRITICAL items resolved
- [ ] Conditional approval — outstanding items documented with remediation timeline
- [ ] Hold — one or more CRITICAL items unresolved

**Approver:**
**Date:**
**Next review date:**
