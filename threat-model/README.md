# Threat Modeling Methodology for LLM Systems

## Why Traditional Threat Modeling Falls Short for LLM Systems

STRIDE and PASTA work well for traditional software. They were designed for systems with deterministic behaviour — where the same input reliably produces the same output, where data flows are bounded, and where trust boundaries are architectural rather than semantic.

LLM systems break several of these assumptions:

- **Non-deterministic outputs** — the same input can produce different outputs, making input validation and output sanitisation fundamentally harder
- **Semantic trust boundaries** — an attacker can manipulate system behaviour through natural language rather than exploiting code vulnerabilities
- **Emergent capabilities** — LLMs can perform actions their designers did not anticipate or intend, expanding the threat surface beyond what was explicitly built
- **Indirect data flows** — in RAG systems, the LLM retrieves data at runtime from sources that may not have been part of the original threat model
- **Dual-use architecture** — the same capability that makes an LLM useful (following instructions) is the capability that attackers exploit (prompt injection)

This does not mean traditional threat modeling is irrelevant. It means it is necessary but not sufficient. LSEF's threat modeling approach layers LLM-specific threat categories on top of your existing threat modeling practice.

---

## The LSEF Threat Modeling Process

### Phase 1 — System Characterisation

Before identifying threats, fully characterise what you are evaluating:

**Data Inventory**
- What data does the LLM have access to — directly (in context) or indirectly (via tools, RAG, function calls)?
- What is the sensitivity classification of each data type?
- What data could be present in training data, fine-tuning data, or system prompts?

**Trust Boundary Mapping**
- Who can submit inputs to the LLM? (External users, internal users, other systems, other LLMs?)
- What can the LLM's outputs trigger? (UI rendering, API calls, database writes, code execution, external service calls?)
- Where does the system prompt come from and who can modify it?
- What tools or functions can the LLM invoke, and what are their permissions?

**Actor Identification**

| Actor | Access Level | Likely Motivation |
|-------|-------------|-------------------|
| External unauthenticated user | Input only | Data exfiltration, service disruption, jailbreak |
| Authenticated external user | Input + some context | Privilege escalation, accessing other users' data |
| Internal user (employee) | System prompt, tool config | IP theft, insider threat, accidental misconfiguration |
| Third-party model/data provider | Training data, model weights | Supply chain compromise |
| Other AI agents (multi-agent) | Peer-level LLM messages | Indirect prompt injection, trust exploitation |

**Deployment Context**
- Is this a customer-facing product, internal tool, or developer platform?
- What regulatory environment applies? (GDPR, HIPAA, SOC2, FedRAMP?)
- What is the blast radius of a worst-case compromise?

---

### Phase 2 — Threat Enumeration

Work through each of the seven LSEF threat categories in order of typical severity:

1. [Prompt Injection](01-prompt-injection.md) — Manipulation of LLM behaviour through crafted inputs
2. [Training Data Poisoning](02-training-data-poisoning.md) — Compromising model behaviour at training time
3. [Model Weight Exfiltration](03-model-weight-exfiltration.md) — Theft of proprietary model IP
4. [Inference Attacks](04-inference-attacks.md) — Extracting training data or model internals through queries
5. [Supply Chain Risks](05-supply-chain.md) — Compromised dependencies, models, or providers
6. [Sensitive Data Exfiltration](06-data-exfiltration.md) — LLM as a vehicle for leaking data
7. [Agentic Workflow Risks](07-agentic-workflow-risks.md) — Risks specific to autonomous LLM systems

For each threat category, assess:
- **Applicability** — Does this threat category apply to your system type?
- **Attack surface** — Where specifically could this manifest in your architecture?
- **Likelihood** — Given your actor set and deployment context, how likely is exploitation?
- **Impact** — What is the worst-case outcome if this threat is realised?

---

### Phase 3 — Risk Prioritisation

After enumeration, apply the [LSEF Risk Scoring Model](../risk-scoring/risk-scoring-model.md) to each identified threat to produce a prioritised remediation roadmap.

The scoring model accounts for:
- Likelihood (actor capability × access × motivation)
- Impact (confidentiality, integrity, availability, reputational)
- Exploitability (is there a known technique? Is it documented publicly?)
- Mitigability (can controls significantly reduce risk?)

---

### Phase 4 — Control Mapping

For each high and critical risk, identify applicable controls from the [mitigations library](../mitigations/). Document:
- Which controls you will implement
- Which risks you will accept (with business justification)
- Which risks require architectural changes before the system can be deployed

---

## Using the Threat Model Template

The [threat-model-template.md](threat-model-template.md) provides a blank structured template for documenting a threat model for a specific LLM system. It should be completed for each distinct LLM system or feature before production deployment.

Recommended cadence:
- **New LLM systems:** Complete full threat model before deployment
- **Significant feature changes:** Re-evaluate applicable threat categories
- **Quarterly:** Review threat model against new threat intelligence
- **Post-incident:** Update threat model with newly observed attack patterns

---

## A Note on Threat Modeling Agentic Systems

Agentic LLM systems — where the LLM can take autonomous actions, call external APIs, browse the web, write and execute code, or interact with other AI agents — require particular care. The threat surface expands dramatically because the LLM is no longer just a text generator but an actor in the system.

The key principle for agentic threat modeling: **treat the LLM's tool-use capability as you would treat a privileged service account.** Ask: if an attacker could control what this tool-using LLM does, what is the worst action it could take? Work backward from that worst case to identify where input validation, sandboxing, confirmation requirements, and scope restrictions are needed.

See [07-agentic-workflow-risks.md](07-agentic-workflow-risks.md) for the full agentic threat category.
