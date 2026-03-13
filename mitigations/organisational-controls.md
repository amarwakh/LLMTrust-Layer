# Organisational Controls for LLM Security

Security frameworks and technical controls fail without the organisational structures that embed them into how teams actually work. This document covers the organisational and process controls that determine whether LLM security practices are sustained rather than applied once and forgotten.

---

## OC-01: LLM Security Review Gate

**What:** A mandatory security review process for LLM systems before production deployment — and for significant changes to existing systems.

**Minimum viable process:**

1. **Intake:** AI/ML team submits a system description and answers a standard intake questionnaire (system type, data in scope, tool access, user population)
2. **Triage:** Security team uses intake information to determine review depth required — lightweight checklist vs. full threat model
3. **Review:** Security engineer works through the appropriate checklist with the AI/ML team; risks documented and owners assigned
4. **Disposition:** Approve / Conditional approve / Hold decision made with clear criteria
5. **Tracking:** Outstanding items tracked to resolution; not closed until remediated or formally accepted

**What makes this work:**
- The process must be fast enough that teams don't route around it. A review that takes 4 weeks will be avoided. Target: 2–5 days for standard reviews.
- Security engineers who conduct reviews must be knowledgeable about LLM security specifically — not just general AppSec
- The checklist must be updated as the threat landscape evolves

**What kills this process:**
- Treating every LLM feature as requiring a 2-week deep review (teams learn to not surface LLM features to security)
- Approving without resolution criteria (outstanding items that never close)
- Security team becoming a bottleneck rather than an enabler

---

## OC-02: LLM Security Champion Program

**What:** Embed security champions in AI/ML teams who are trained specifically in LLM security — enabling security review and guidance to happen earlier in the development process, before formal security review gates.

**Why this matters for LLM security specifically:** LLM security issues are often architectural — they are much cheaper to fix during design than after a system is built. A security champion embedded in the team can flag "this RAG system needs access-controlled retrieval" at the design stage, not after deployment.

**Champion training minimum:**
- OWASP LLM Top 10 familiarity
- Prompt injection: what it is, how to test for it, how to recognise injection-vulnerable designs
- Data classification: understanding what data is sensitive and how it affects LLM architecture decisions
- Secure system prompt design: what belongs in a system prompt and what does not

**Effort to establish:** Medium. Champion program setup requires training investment; ongoing requires 2–4 hours/month per champion.

---

## OC-03: LLM Security Incident Classification

**What:** Define what constitutes an LLM security incident and establish a response process before incidents occur — not after.

**LLM-specific incident categories:**

| Category | Examples | Severity Guidance |
|----------|----------|-------------------|
| Successful prompt injection | System prompt extracted; safety filter bypassed; injected instructions executed | High–Critical depending on data exposed or actions taken |
| Sensitive data exfiltration via LLM | PII, credentials, or confidential data appearing in LLM outputs | High–Critical |
| Jailbreak at scale | Coordinated attempt to bypass content safety across many users | Medium–High |
| RAG data boundary violation | User retrieves documents outside their authorised scope | High |
| Agentic system unintended action | LLM agent takes action outside its intended scope | High–Critical |
| Model availability attack | Prompt-based DoS or cost attack via token exhaustion | Medium |

**Response process minimum:**
- Who is notified for each severity level
- How to rapidly disable or rate-limit the LLM system if needed
- How to preserve evidence (logs, inputs, outputs) for post-incident analysis
- What constitutes mandatory disclosure (to users, regulators, or publicly)

---

## OC-04: Red Team Cadence

**What:** Schedule regular offensive security exercises specifically targeting LLM systems — not just at launch but on an ongoing basis.

**Recommended cadence:**
- **Pre-deployment:** Full red team exercise before any new LLM system goes to production
- **Quarterly:** Targeted red team of highest-risk LLM systems; focus on new techniques from the past quarter
- **Post-model-update:** Abbreviated re-test when the base model version changes (model updates can change injection susceptibility)
- **Post-incident (industry):** When a significant LLM security incident occurs publicly, test your own systems for the same technique

**What the LLM red team should cover beyond traditional AppSec:**
- Prompt injection (all taxonomy variants — direct, indirect, multi-turn, multi-agent)
- Jailbreak and safety filter bypass testing
- Data extraction and context window probing
- Agentic system: tool chain abuse, privilege escalation via creative tool use, SSRF
- Social engineering the LLM: can it be manipulated to provide incorrect security guidance to users?

---

## OC-05: LLM Security Metrics

**What:** Define and track metrics that indicate whether your LLM security posture is improving or degrading.

**Recommended metrics:**

*Coverage metrics:*
- % of LLM systems in production that have completed pre-deployment security review
- % of LLM systems with active monitoring configured
- % of LLM systems with injection test suite in CI/CD

*Incident metrics:*
- Number of LLM security incidents per quarter (by category)
- Mean time to detect LLM security incidents
- Mean time to contain LLM security incidents

*Program health metrics:*
- Outstanding security review findings by age and severity
- Red team exercise completion rate vs. schedule
- Security champion training completion rate

**Reporting cadence:** Monthly to security leadership; quarterly to executive team for high-risk LLM deployments.

---

## OC-06: Staying Current — LLM Threat Intelligence

**What:** A defined process for the security team to stay current on emerging LLM attack techniques and update the security program accordingly.

**The LLM threat landscape moves fast.** New injection techniques, new jailbreak methods, and new attack categories emerge continuously. A framework written today will have gaps within 6 months if not actively maintained.

**Minimum viable threat intelligence process:**
- Designate one security team member as the LLM security SME — responsible for tracking emerging research
- Monthly review of: OWASP LLM project updates, MITRE ATLAS additions, notable LLM security incidents and research publications
- Quarterly update of this framework and associated checklists based on new threat intelligence
- When a significant new technique emerges: assess whether existing deployed systems are vulnerable; update test suites; communicate to AI/ML teams

**Key sources to monitor:**
- OWASP LLM Top 10 project (owasp.org/www-project-top-10-for-large-language-model-applications)
- MITRE ATLAS (atlas.mitre.org)
- Arxiv cs.CR (security research including LLM security papers)
- AI safety and security communities (Anthropic, DeepMind, OpenAI safety publications)
- Security conference proceedings (DEF CON AI Village, NeurIPS security workshops)
