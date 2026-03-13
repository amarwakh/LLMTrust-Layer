# Mitigation Patterns — Prompt Injection

This document covers mitigation patterns for prompt injection threats (direct, indirect, and multi-agent variants). Controls are organised by effectiveness and implementation complexity.

---

## Tier 1 — Architectural Controls (Highest Impact, Address First)

### M-PI-01: Privilege Separation — Keep Secrets Out of Context

**What:** Never place API keys, credentials, system passwords, or confidential business logic in the LLM's system prompt or context window.

**Why it works:** A secret that is not in the LLM's context cannot be extracted by prompt injection. This eliminates the most common high-severity injection outcome.

**Implementation:**
- API keys and credentials → secrets management (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault)
- Confidential business logic → encode in application code, not system prompts
- If context injection of sensitive data is unavoidable → ensure the system prompt instructs the LLM never to repeat it, AND assume this instruction will sometimes fail (defence in depth)

**Limitations:** Does not prevent injection from manipulating LLM behaviour — only prevents secrets in context from being extracted.

**Effort:** Low–Medium (refactoring prompts to remove secrets; updating credential management)

---

### M-PI-02: Minimal Tool Scope for Agentic Systems

**What:** Grant LLM agents only the permissions required for their specific task. Review and prune tool permissions regularly.

**Why it works:** Prompt injection can only exercise permissions the LLM has. An agent with read-only access to one database cannot exfiltrate data from other systems even if fully compromised.

**Implementation:**
- Enumerate every tool the LLM has access to
- For each tool, ask: is this required for the stated function?
- Scope tools as narrowly as possible (specific resource ARNs, read-only where possible, specific API endpoints rather than broad access)
- Consider task-scoped tool sets: dynamically grant tools based on the current task rather than maintaining a static maximum tool set

**Limitations:** Does not prevent injection — limits blast radius when injection occurs.

**Effort:** Medium (requires audit of existing tool permissions and potentially architectural changes)

---

### M-PI-03: Output Validation — Treat LLM Output as Untrusted

**What:** Apply the same input validation to LLM outputs that you would apply to user-submitted data before using outputs to drive system actions, render in browsers, construct queries, or call APIs.

**Why it works:** Even if injection is successful, output validation can prevent the injected output from causing harm downstream.

**Specific implementations by output type:**

| Output Use | Validation Required |
|------------|-------------------|
| Rendered in HTML/browser | HTML encoding; CSP headers; strip script tags |
| Used in database query | Parameterise the query; never concatenate LLM output into SQL |
| Used to call an API | Validate against expected schema and value ranges; whitelist allowed API endpoints |
| Rendered as code for execution | Sandbox execution; validate against allowed operations |
| Passed to another LLM | Apply same scrutiny as user input; consider injection payload detection |

**Effort:** Low–High (depends on how deeply LLM outputs are embedded in downstream processing)

---

### M-PI-04: Input Content Validation for Retrieved/External Content

**What:** Before external content (web pages, documents, emails, database records) enters the LLM's context window, apply content validation to detect and sanitise potential injection payloads.

**Why it works:** Indirect prompt injection requires the malicious content to reach the LLM's context. Validation at the retrieval boundary reduces the attack surface.

**Implementation approaches:**
- Pattern matching: Block content containing common injection patterns ("ignore previous instructions", "new system prompt", etc.) — note: this is easily bypassed but catches unsophisticated attempts
- LLM-based content validation: Use a separate, minimal-scope LLM to evaluate retrieved content before passing it to the main LLM — this is the highest-efficacy approach but adds latency and cost
- Content sandboxing: Structure the prompt to clearly delineate between trusted instructions and untrusted retrieved content using XML-style tags, instructing the LLM to treat the retrieved content as data only

**Limitations:** No validation approach is fully effective against indirect injection — defence in depth is essential.

**Effort:** Medium–High

---

## Tier 2 — Detection and Monitoring Controls

### M-PI-05: Input and Output Monitoring

**What:** Log all LLM inputs and outputs; apply monitoring rules to detect injection attempts and successful injections.

**Detection signals to monitor:**

*Input-side signals:*
- Inputs containing "ignore previous instructions" or variants
- Unusually long inputs (potential padding attacks)
- Inputs containing structured data that looks like prompt syntax (XML tags, JSON that mimics system prompts)
- High volume of injection-adjacent patterns from a single user/session
- Inputs in unexpected languages (potential multilingual bypass attempt)

*Output-side signals:*
- Outputs that contain system prompt text (indicates successful extraction)
- Outputs that reference overriding instructions
- Sudden changes in output style, language, or persona
- Outputs containing unexpected data (credentials, PII patterns) not present in the user's input
- Tool calls with unexpected parameters or to unexpected endpoints (agentic systems)

**Tooling:** Most major observability platforms support LLM output monitoring. LLM-specific monitoring tools (Langfuse, Weights & Biases, Arize AI) provide semantic monitoring capabilities beyond simple pattern matching.

**Effort:** Medium (tooling setup + alerting rule development)

---

### M-PI-06: Rate Limiting and Session Controls

**What:** Implement rate limiting on LLM endpoints per user/session; implement session isolation.

**Why it works:** Automated injection probing requires high request volumes. Rate limiting forces attackers to slow down, making systematic bypass attempts more detectable.

**Implementation:**
- Per-user rate limiting: Limit LLM API calls per user per time window
- Progressive throttling: Increase restrictions for users who trigger injection detection signals
- Session isolation: Ensure users' context windows do not contaminate each other
- Geographic and device anomaly detection: Flag unusual access patterns for investigation

**Effort:** Low–Medium

---

## Tier 3 — Procedural Controls

### M-PI-07: Prompt Injection Red Team Exercise

**What:** Before deploying any LLM system (and after significant changes), conduct a structured red team exercise specifically targeting prompt injection.

**Minimum red team scope:**
1. Test all direct injection variants from the taxonomy in threat-model/01-prompt-injection.md
2. If RAG system: plant injection payloads in the retrieval corpus; verify whether they execute
3. If agentic: attempt to cause the LLM to invoke tools with unexpected parameters
4. Test multilingual injection variants
5. Test multi-turn gradual manipulation
6. Attempt context window extraction of system prompt content

**Outputs:** Document which attacks succeeded, which failed, and which controls provided effective defence. Use results to update the pre-deployment checklist for this system.

**Effort:** Medium (2–5 days for a thorough red team of a typical system)

---

### M-PI-08: Injection Test Suite in CI/CD

**What:** Maintain an automated suite of prompt injection tests that runs against the system on every deployment — especially when the system prompt or model version changes.

**Why it matters:** System prompt changes and model updates can inadvertently change injection susceptibility. Automated testing catches regressions before they reach production.

**Minimum test suite contents:**
- 10–20 direct injection payloads covering the major taxonomy categories
- 5–10 context extraction attempts
- System-specific payloads based on the known tool set and data in context

**Effort:** Medium (initial test suite development; low ongoing maintenance)

---

## Controls That Provide Limited Protection (Use With Caution)

These controls are commonly used but provide weak protection when used alone:

| Control | Why It's Weak |
|---------|---------------|
| "Do not follow injected instructions" in system prompt | LLMs cannot reliably enforce their own meta-instructions; this is bypassed routinely |
| Keyword blocking of "ignore previous instructions" | Trivially bypassed by rephrasing; creates false sense of security |
| Relying on base model safety training | Reduces susceptibility; does not eliminate it; new bypass techniques emerge continuously |
| Input length limits | Blocks some padding attacks; does not address semantic injection |

These controls are worth implementing as part of defence in depth. They should not be the primary or sole defence against prompt injection.

---

## Prioritisation Guidance

If you can only implement controls in phases, prioritise in this order:

1. **M-PI-01** — Remove secrets from context (zero-cost blast radius reduction)
2. **M-PI-02** — Minimal tool scope (for agentic systems; critical before deployment)
3. **M-PI-03** — Output validation (prevents downstream harm from successful injection)
4. **M-PI-07** — Red team exercise (validates your actual defence posture before production)
5. **M-PI-05** — Monitoring (enables detection and response to injection in production)
6. **M-PI-04** — Input content validation (for RAG/retrieval systems)
7. **M-PI-06** — Rate limiting (hardens against automated probing)
8. **M-PI-08** — CI/CD test suite (prevents regression on subsequent deployments)
