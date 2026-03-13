# Case Study 02 — Security Evaluation of GovCloud Region Review Automation

**System Type:** Type A (LLM-as-Feature) / Type B (Internal Security Platform)
**Domain:** Regulated Cloud Security — FedRAMP, GovCloud, Sovereign Deployments
**Scale:** 77+ region reviews processed; ~64 engineering hours/month saved
**Outcome:** Automated security review pipeline for GovCloud, Opt-In, and China region deployments with sustained security bar compliance

---

## Background

AWS services that want to expand into regulated deployment environments — GovCloud (US-East and US-West), Opt-In regions, and China (Beijing and Ningxia operated by NWCD) — must pass a region review process before they can operate in those environments. These reviews exist because regulated deployments carry additional compliance obligations: FedRAMP High/Moderate authorisation for GovCloud, China-specific data residency and cybersecurity law requirements, and Opt-In region-specific data sovereignty controls.

Each region review requires the service team to demonstrate that their architecture, data handling, access controls, encryption posture, and operational procedures meet the bar for the target environment. Reviews are conducted by security engineers who evaluate the service's security design against environment-specific control requirements.

The bottleneck: as AWS's service portfolio grew, region review volume exceeded the capacity of security engineers to process them manually at the required quality level and speed. Reviews were queuing. Service teams were waiting weeks for security clearance to expand into regulated environments. The security team was spending significant engineering hours on reviews that followed highly repeatable evaluation patterns.

**The problem was not judgment capacity — it was throughput on repeatable evaluation work.**

---

## System Architecture

```
Region Review Submission (Service Team)
         │
         ▼
   Review Intake Pipeline
   ├── Service metadata extraction
   ├── Target region identification (GovCloud / Opt-In / China)
   ├── Review type classification (new service / expansion / config change)
   └── Dependency graph resolution (which upstream services does this depend on?)
         │
         ▼
   Control Requirement Lookup (RAG)
   ├── FedRAMP control baseline (High / Moderate / Low)
   ├── GovCloud-specific AWS service restrictions
   ├── China region data residency requirements
   ├── Opt-In region sovereignty controls
   └── Historical review decisions for similar service architectures
         │
         ▼
   LLM Evaluation Engine
   ├── Control gap analysis (submitted posture vs. required baseline)
   ├── Risk identification (what controls are missing or insufficient?)
   ├── Dependency risk propagation (do upstream service gaps affect this review?)
   └── Structured evaluation report generation
         │
         ▼
   Human Security Engineer Review
   ├── Review LLM-generated gap analysis
   ├── Apply judgment to ambiguous or novel controls
   ├── Approve / Conditional approve / Reject
   └── Override and correction feedback loop
         │
         ▼
   Outcome Recording + Baseline Update
```

**Key AWS services in the pipeline:**
- **Amazon S3** — Review submission storage and artefact management
- **AWS Lambda** — Review intake processing and dependency resolution
- **Amazon DynamoDB** — Review state management and audit trail
- **Amazon Bedrock** — LLM inference layer (Claude model)
- **Amazon Kendra** — Retrieval-augmented search over control documentation and historical reviews
- **AWS IAM** — Access control for review submission and outcome access
- **AWS CloudTrail** — Full audit trail of review actions and LLM invocations
- **Amazon EventBridge** — Review state transition events and notifications
- **AWS Secrets Manager** — Credentials for downstream integrations

---

## Security Controls Evaluated

The automation evaluated service submissions against the following control domains. For each domain, the LLM-generated evaluation was structured as: *current posture described → required baseline → gap identified → risk rating.*

### Control Domain 1 — Data Residency and Sovereignty

**GovCloud requirement:** All data processed by the service — customer data, control plane metadata, logging artefacts, backup data — must remain within the GovCloud boundary (us-gov-east-1 or us-gov-west-1). No data egress to standard commercial regions is permitted.

**What the automation evaluated:**
- S3 bucket configurations: Are bucket policies enforcing `aws:RequestedRegion` conditions? Are S3 replication rules configured, and if so, do they replicate only within the GovCloud boundary?
- CloudWatch Logs destinations: Are log groups configured to export to destinations outside GovCloud?
- Cross-region API calls: Does the service make API calls to commercial region endpoints?
- Backup and DR configurations: Are AWS Backup plans sending recovery points to commercial regions?
- Service dependency declarations: Do any declared upstream dependencies operate in commercial-only regions?

**China region additional requirement:** Data residency enforced within cn-north-1 (Beijing) or cn-northwest-1 (Ningxia); no cross-border data transfer without explicit MIIT approval documentation.

**Common gaps identified:** Services that had implemented data residency correctly for standard Opt-In regions frequently had CloudWatch Logs export configurations pointing to commercial region destinations — a subtle misconfiguration that was consistently flagged.

---

### Control Domain 2 — Encryption at Rest and in Transit

**GovCloud requirement (FedRAMP High):**
- Encryption at rest: FIPS 140-2 validated cryptographic modules required; AWS-managed keys insufficient for most data classifications — customer-managed KMS keys with appropriate key policies required
- Encryption in transit: TLS 1.2 minimum; TLS 1.3 preferred; explicit prohibition on deprecated cipher suites

**What the automation evaluated:**
- KMS key configuration: Are CMKs used for all data stores? Are key policies restricting access to GovCloud IAM principals only? Is key rotation enabled?
- DynamoDB encryption: Is encryption at rest configured with CMK rather than AWS-owned key?
- RDS/Aurora encryption: Is encryption at rest enabled and using CMK? Are automated backups also encrypted?
- S3 bucket default encryption: Is SSE-KMS with CMK configured as the default?
- API Gateway and ALB: Are TLS policies explicitly configured to exclude deprecated protocols?
- ECS/EKS inter-service communication: Is mTLS or equivalent configured for service-to-service calls?

**Risk accepted — documented decision:** Several services used AWS-managed keys (SSE-S3) for non-customer-data artefacts (build artefacts, internal metrics aggregations). After evaluating the data classification of these artefacts, the security team accepted this as a low-risk deviation with documented justification — the artefacts contained no customer data and were not subject to FedRAMP data-level controls. This decision was recorded in the review audit trail with rationale.

---

### Control Domain 3 — Access Control and IAM Posture

**GovCloud requirement:**
- IAM roles must be scoped to GovCloud ARNs — commercial region IAM principals cannot assume GovCloud roles
- No IAM wildcard permissions (`*`) on actions or resources for roles with access to customer data
- Service control policies (SCPs) must enforce GovCloud boundary
- MFA required for all human access to GovCloud environments

**What the automation evaluated:**
- IAM role trust policies: Do trust policies contain `arn:aws-us-gov:*` ARNs or commercial `arn:aws:*` ARNs?
- IAM role permission scope: Are there `*` wildcard actions on data-plane permissions?
- Cross-account access: Are there cross-account role assumptions that traverse the GovCloud / commercial boundary?
- Service-linked roles: Are service-linked roles scoped appropriately for GovCloud operation?
- AWS Organizations SCP configuration: Are appropriate deny SCPs in place at the GovCloud OU level?

**Most common high-severity gap found:** Services migrated from commercial regions to GovCloud frequently had IAM trust policies with commercial region account IDs in the principal — allowing commercial-region roles to assume GovCloud roles. This was flagged as a critical finding in every instance.

---

### Control Domain 4 — Audit Logging and SIEM Integration

**FedRAMP AU (Audit and Accountability) control family requirement:**
- CloudTrail enabled in all GovCloud regions with log file validation
- CloudTrail logs exported to a security-owned, immutable log archive
- VPC Flow Logs enabled for all VPCs with customer data
- S3 access logging enabled for all buckets with customer data
- Log retention: minimum 1 year hot, 3 years cold (FedRAMP High)
- SIEM integration: logs forwarded to the GovCloud SIEM within the boundary

**What the automation evaluated:**
- CloudTrail configuration: Multi-region trail enabled? Log file validation enabled? Is the S3 destination bucket in a security-owned account with appropriate bucket policy?
- CloudWatch Logs retention policies: Are retention periods meeting the minimum?
- S3 server access logging: Enabled on buckets containing customer data?
- VPC Flow Logs: Enabled and configured to capture REJECT traffic?
- Config Rules: Are relevant AWS Config rules enabled in the GovCloud account?

---

### Control Domain 5 — Incident Response and Operational Posture

**FedRAMP IR (Incident Response) control family:**
- Incident response plan documented and specific to GovCloud environment
- FedRAMP incident reporting timeline: US-CERT notification within 1 hour for major incidents
- GovCloud-specific runbooks distinct from commercial runbooks
- No use of commercial-region incident response tooling that would require data egress

**What the automation evaluated (limited — highest human review dependency):**
- Runbook completeness: Does the submitted documentation include GovCloud-specific runbooks?
- US-CERT notification procedure: Is the notification procedure documented and tested?
- Tooling inventory: Are any incident response tools listed that have commercial-region dependencies?

**Human review priority:** This control domain was flagged for mandatory human review on every submission — the automation generated a checklist of documentation gaps but did not attempt to assess the adequacy of IR plans, which requires security engineer judgment.

---

## Security Threats Identified — Pre-Deployment Evaluation of the Automation System

### Threat 1 — Prompt Injection via Review Submission Content

**Risk:** A service team submitting a region review could embed prompt injection payloads in their architecture documentation, service descriptions, or control narratives — attempting to cause the LLM to generate a favourable evaluation that does not reflect the actual security posture.

**Risk Score:** Critical (13.5)
- Likelihood: 3 (authenticated internal users; insider threat motivation exists — passing review unblocks revenue-generating service launches)
- Impact-Confidentiality: 2 (review content is internal)
- Impact-Integrity: 4 (a manipulated review outcome could allow a non-compliant service to operate in FedRAMP High environment — a regulatory and customer trust failure of the highest order)
- Exploitability: 3 (technique is documented; motivation is significant)

**Mitigations applied:**
- Structured output enforcement: LLM evaluation output is constrained to a strict JSON schema — gap present/absent, risk rating from an enumerated set, specific control reference. The LLM cannot output a free-form "this service passes" — it outputs structured gap data that a separate system renders into a report
- Human review mandatory for approve/reject decision: The LLM generates the gap analysis. A human security engineer makes the disposition decision. The automation cannot approve a service for GovCloud deployment
- Submission content isolation: Review submission content is injected into the LLM context with explicit XML delimiters marking it as untrusted data: `<submission_content>...</submission_content>` — and the system prompt explicitly instructs the model to treat content within these tags as data to evaluate, not as instructions
- Full logging of submission content and LLM output for retrospective audit

---

### Threat 2 — Historical Review Corpus Poisoning

**Risk:** The RAG system retrieves from historical review decisions to inform current evaluations. If historical review records are modified or if low-quality approvals accumulate in the corpus, the LLM may calibrate its gap analysis against a degraded baseline.

**Risk Score:** High (9.0)
- Likelihood: 2 (requires write access to historical review records; controlled access path)
- Impact-Integrity: 4 (baseline degradation affects all subsequent reviews)
- Impact-Confidentiality: 2
- Exploitability: 2 (requires insider access; not trivially exploitable)

**Mitigations applied:**
- Immutable review archive: Historical review records stored in S3 with Object Lock (WORM) — approved reviews cannot be modified after recording
- Write access restricted to security-owned pipeline: Only the automated pipeline can write to the historical review corpus; no human direct write access
- Retrieval source version pinning: The Kendra index is rebuilt from the immutable archive on a scheduled basis — not updated in real time from mutable sources

---

### Threat 3 — Regulatory Compliance Gap from LLM Miscalibration

**Risk:** If the LLM's evaluation of control requirements is incorrect — due to outdated control documentation in the RAG corpus, model misinterpretation, or edge cases — a service with genuine compliance gaps could receive an incorrect gap-free evaluation.

**Risk Score:** High (10.0)
- Likelihood: 3 (FedRAMP control requirements do change; model interpretation of complex regulatory language is imperfect)
- Impact-Integrity: 4 (FedRAMP non-compliance in GovCloud is a regulatory failure with contract and legal consequences)
- Impact-Confidentiality: 2
- Exploitability: 1 (not directly exploitable; systemic quality risk)

**Mitigations applied:**
- Control documentation version management: FedRAMP baseline documents in the Kendra index are versioned and updated on a defined schedule when NIST/FedRAMP publish revisions
- Mandatory human review for all high-risk control domains: IR, access control, and encryption control domains flagged for mandatory human review regardless of LLM gap analysis outcome
- Quarterly calibration: Sample of completed reviews evaluated against independent human security engineer assessment; agreement rate tracked; disagreements trigger corpus and prompt review

---

## Risks Formally Accepted — With Documented Rationale

The security team formally accepted the following risks during the system's security review:

| Risk | Rationale for Acceptance | Compensating Control |
|------|--------------------------|----------------------|
| LLM evaluation of IR plan adequacy | IR plan quality requires nuanced judgment the LLM cannot reliably provide | Mandatory human review for all IR control domain findings |
| AWS-managed keys for non-customer-data artefacts | Artefacts contain no customer data; not subject to FedRAMP data controls | Data classification documented in review record; re-evaluated if data classification changes |
| Indirect prompt injection via submitted documentation | Structured output schema and human disposition gate prevent injection from affecting outcomes | Injection monitoring; quarterly red team of submission injection vectors |
| LLM evaluation accuracy for novel service architectures | First-of-kind architectures may not have close historical precedents in the retrieval corpus | Novel architecture flag triggers mandatory senior security engineer review |

---

## Outcomes

**Quantified:**
- 77+ region reviews processed through the automated pipeline in 4 months post-deployment
- ~64 engineering hours/month saved on repeatable evaluation tasks
- Zero FedRAMP compliance gaps identified post-deployment audit (subsequent FedRAMP assessor review found no previously undetected compliance issues in reviewed services)
- Human review time per submission reduced from ~5 hours to ~45 minutes (LLM handles gap identification; human validates and decides)

**Qualitative:**
- Service teams received structured, consistent gap analysis rather than variable-quality manual reviews — improving the predictability and fairness of the review process
- Security engineers reported higher job satisfaction — repetitive checklist evaluation replaced by judgment-intensive review of LLM-identified gaps

---

## Lessons Applicable to Similar Systems

**1. Regulatory compliance automation requires an immutable audit trail.**
Every input, every LLM evaluation output, and every human disposition decision must be logged immutably. When a FedRAMP assessor asks "how was this control evaluated?", you need a complete, tamper-evident record. Design the audit trail before you design the automation.

**2. The LLM's role is gap identification, not compliance certification.**
The most important architectural decision: the automation identifies gaps; humans certify compliance. This boundary is not just a safety guardrail — it is the correct division of labour between machine pattern-matching and human regulatory judgment.

**3. Structured output schemas are a first-line injection defence in compliance contexts.**
When the LLM can only output a structured gap/no-gap evaluation against a predefined control list, the blast radius of a successful prompt injection is dramatically reduced. The attacker cannot inject a "this service complies" outcome — they can only attempt to affect which controls are flagged as gaps. Combined with human review, this makes injection manipulation extremely difficult.

**4. RAG corpus integrity is as important as LLM quality.**
A highly capable LLM evaluating against a degraded or outdated control corpus produces degraded evaluations. Treat the RAG knowledge base with the same rigour you would apply to any compliance-critical data source — version control, integrity verification, update governance.
