# Case Study 03 — High-Severity Data Services Customer Escalation: Security Response Across 26 Dependent Services

**System Type:** N/A — This case study covers a security incident response and remediation program, not an LLM deployment
**Domain:** Product Security Incident Response — AWS Data Services
**Scale:** 26 dependent services; multiple enterprise customer commitments; cross-organisational remediation
**Outcome:** Structured data classification framework, negotiated remediation timeline, full customer assurance with zero unplanned data boundary violations post-remediation

---

## Why This Case Study Is in an LLM Security Framework

This case study does not involve an LLM system. It is included because it illustrates a security engineering pattern that is directly applicable to LLM deployments at scale: **how to manage a high-severity security finding that propagates across a complex service dependency graph, where remediating one service requires coordinated action across many others.**

In LLM security, this pattern appears when a vulnerability in a shared security framework, a common LLM library, or a shared RAG data source affects multiple downstream systems simultaneously. The response playbook — dependency mapping, data classification, tiered remediation, stakeholder communication — is the same.

---

## Background

AWS's data services portfolio — Glue, Lake Formation, OpenSearch Service, Athena, EMR, QuickSight, SMUS, DataZone — provides enterprise customers with the infrastructure for data ingestion, transformation, cataloguing, query, analytics, and governance at scale. These services are deeply interdependent: a data pipeline typically flows through multiple services in sequence, and a security control (or gap) in one service can propagate effects to all downstream services that consume its outputs.

A high-severity security finding was identified in a data handling component shared across this service portfolio. The finding had the potential to affect how customer data was classified, stored, and accessed across services that consumed data from the affected component — creating a risk of data boundary violations where customer data from one tenant could potentially be accessed in the context of another tenant's operations.

The finding triggered a customer escalation from an enterprise customer with contractual security commitments — specifically, a requirement for data isolation guarantees that the finding potentially undermined.

**The core challenge:** This was not a vulnerability in a single service that could be patched and deployed. It was a data handling pattern that had been adopted across 26 dependent services, each of which needed to be individually assessed, classified by risk level, and remediated according to a prioritised timeline — while simultaneously managing a customer whose confidence in the platform was shaken.

---

## Phase 1 — Dependency Mapping and Impact Assessment

The first 48 hours were spent answering one question: *which services are actually affected, and how?*

**Dependency graph construction:**

Using AWS service dependency data and direct interrogation of each service team's data flow documentation, a directed graph was constructed mapping:

- Which services ingested data from the affected component
- What transformations each service applied to that data
- Where each service's outputs went — to other internal services, to customer-accessible APIs, or to customer-controlled storage

```
Affected Component
      │
      ├──→ AWS Glue (ETL jobs — data transformation layer)
      │         └──→ AWS Lake Formation (data catalogue and governance)
      │                   ├──→ Amazon Athena (query layer over Lake Formation governed data)
      │                   ├──→ Amazon QuickSight (BI layer — customer-facing dashboards)
      │                   └──→ AWS DataZone (data marketplace — cross-account data sharing)
      │
      ├──→ Amazon OpenSearch Service (search and analytics index)
      │         └──→ Multiple customer application integrations
      │
      ├──→ Amazon EMR (large-scale data processing)
      │         ├──→ Amazon S3 (output storage — customer-controlled buckets)
      │         └──→ AWS Glue (catalogue integration)
      │
      └──→ AWS SMUS (storage management and unified search)
                └──→ Amazon S3 (storage backend)
```

**Finding at the end of Phase 1:**
- 26 services confirmed as having data flows through the affected component
- 8 services assessed as having **direct customer data exposure risk** — data they processed could potentially be affected in ways visible to customers
- 18 services assessed as having **indirect risk** — affected component was in their dependency chain but downstream data was not directly customer-accessible or was further transformed in ways that mitigated the original finding
- 3 services (DataZone, QuickSight, Lake Formation) assessed as **critical priority** — these services had cross-account data sharing capabilities where a data boundary violation would be most severe

---

## Phase 2 — Data Classification Framework

With 26 services in scope and finite remediation capacity, the team needed a principled way to prioritise — not just by service name but by the actual risk posed by each service's data handling in the context of the finding.

**The data classification framework developed:**

Each affected service's data was classified across two dimensions:

**Dimension 1 — Data Sensitivity**

| Class | Definition | Examples in Context |
|-------|------------|-------------------|
| S1 — Critical | Customer data that if exposed cross-tenant would constitute a data breach | Customer PII in DataZone catalogues; QuickSight dashboard data containing customer records |
| S2 — Confidential | Internal operational data that if cross-contaminated would violate customer isolation but not constitute regulated data exposure | Glue job metadata; EMR cluster configuration data; Lake Formation permission metadata |
| S3 — Internal | Service operational data with no direct customer data content | Service health metrics; internal audit logs; configuration state |
| S4 — Public | Data with no confidentiality requirement | Public documentation indices; public-facing OpenSearch indices |

**Dimension 2 — Exposure Vector**

| Vector | Definition |
|--------|------------|
| V1 — Direct customer API | Service exposes this data directly through customer-callable APIs |
| V2 — Cross-account sharing | Service can share this data across AWS account boundaries (DataZone, Lake Formation cross-account grants) |
| V3 — Customer-controlled storage | Data is written to customer-controlled S3 buckets or equivalent |
| V4 — Internal only | Data remains within service boundary; not customer-accessible |

**Prioritisation matrix:**

| | V1 — Direct API | V2 — Cross-account | V3 — Customer storage | V4 — Internal |
|--|-----------------|-------------------|----------------------|----------------|
| **S1 — Critical** | P0 — Immediate | P0 — Immediate | P1 — 14 days | P2 — 30 days |
| **S2 — Confidential** | P1 — 14 days | P1 — 14 days | P2 — 30 days | P3 — 90 days |
| **S3 — Internal** | P2 — 30 days | P2 — 30 days | P3 — 90 days | P4 — Next cycle |
| **S4 — Public** | P4 — Next cycle | P4 — Next cycle | P4 — Next cycle | No action |

**Application to the 26 services:**

| Priority | Services | Count |
|----------|----------|-------|
| P0 — Immediate remediation | DataZone, Lake Formation (cross-account grants), QuickSight (direct customer data) | 3 |
| P1 — 14 days | OpenSearch (customer API), Athena (direct query results), EMR (customer bucket writes) | 5 |
| P2 — 30 days | Glue (ETL output metadata), SMUS (customer-accessible search), remaining direct API services | 8 |
| P3 — 90 days | Internal pipeline services, metadata stores, monitoring integrations | 7 |
| P4 — Next planning cycle | Internal operational services with no customer data exposure | 3 |

---

## Phase 3 — Customer Communication and Expectation Management

The escalating enterprise customer had a contractual security commitment that included data isolation guarantees. They were aware a finding had been identified and wanted two things: assurance that their data had not been compromised, and a commitment to a remediation timeline.

**What we could honestly say at the time of first communication:**
- The finding had been identified proactively through internal security review, not through a customer-detected incident
- No evidence of actual customer data boundary violation had been found at the time of communication
- A structured impact assessment was underway
- P0 services would be remediated within 72 hours; full remediation on a defined timeline

**What we could not say:**
- That we could guarantee no data had been affected prior to detection (honest — we could not make this assertion while the dependency assessment was still in progress)
- That the timeline the customer initially demanded (all 26 services in 14 days) was achievable without compromising remediation quality

**The timeline negotiation:**

The customer's initial demand was complete remediation across all 26 services within 14 days. This was not achievable — remediating all 26 services in 14 days would require cutting corners on the remediation quality in lower-priority services, potentially introducing new issues.

The security team's position:
- P0 services: 72 hours (non-negotiable — these are the actual risk)
- P1 services: 14 days (achievable with dedicated engineering resources)
- P2 services: 30 days (achievable with normal sprint allocation)
- P3/P4 services: 90 days / next planning cycle (these have no direct customer data exposure — expediting them provides no additional protection to the customer)

**The argument made to the customer:**

*"Demanding all 26 services remediated in 14 days is not in your interest. The 3 services that actually handle your data in ways that matter to your isolation guarantee will be remediated in 72 hours. The remaining 23 services are either indirect dependencies or handle data in ways that do not affect your customer data boundary. Rushing their remediation to meet an arbitrary deadline increases the risk of introducing new issues in services that are currently low-risk. The right ask is: P0 in 72 hours, P1 in 14 days — and we commit to that."*

The customer accepted this framing. The data classification framework was shared with them (with appropriate detail level) so they could understand the prioritisation logic rather than perceiving it as us managing down their expectations.

---

## Phase 4 — Remediation Execution and Tracking

**P0 remediation (72 hours):**

The three P0 services required emergency remediation sprints:

- **DataZone:** Cross-account data sharing grants audited; affected sharing configurations suspended pending validation; customer notified of temporary service impact with estimated restoration time
- **Lake Formation:** Cross-account grants involving affected data classifications reviewed; permissions tightened to explicit allow-lists while remediation was in progress
- **QuickSight:** Dashboard data refresh suspended for affected data sources; customer-facing dashboards displaying potentially affected data temporarily disabled with customer notification

**P1 remediation (14 days):**

- **OpenSearch:** Re-indexing of affected indices with remediated data handling; index access controls reviewed and tightened
- **Athena:** Query execution path for affected Lake Formation governed tables reviewed; query result caching cleared and disabled temporarily
- **EMR:** Job output validation added to detect any data classification boundary violations in job outputs before write to customer buckets

**Tracking mechanism:**

A dedicated remediation tracker was maintained with:
- Service name
- Priority classification and rationale
- Assigned engineering team and DRI (Directly Responsible Individual)
- Committed remediation date
- Validation criteria (how will we know the remediation is complete and correct?)
- Status (Not started / In progress / Remediated / Validated)
- Customer communication sent (Y/N)

This tracker was reviewed in a daily 30-minute standup across all service teams for the first 14 days, then moved to every-other-day as P0 and P1 services were validated.

---

## Security Findings From the Incident Response

### Finding 1 — Insufficient Data Classification in Service Dependency Documentation

**Finding:** Of the 26 affected services, only 7 had up-to-date documentation of the sensitivity classification of data they processed. The remaining 19 required manual investigation to determine whether they handled customer data, what classification, and through what exposure vectors.

**Impact:** The dependency assessment took 48 hours instead of the 4–6 hours it should have taken with complete documentation.

**Remediation:** Data classification documentation added as a mandatory field in AWS service security bar review — services cannot pass their annual security review without documented data classification for all data they process.

---

### Finding 2 — Cross-Service Security Dependency Visibility

**Finding:** No single team had visibility into the full dependency graph of which services consumed from the affected component. Each team knew their own upstream dependencies but not the aggregate downstream impact of a finding in a shared component.

**Impact:** Dependency graph construction required manual interrogation of 26 service teams rather than automated retrieval.

**Remediation:** Service dependency graph tooling updated to include security classification of data flows — allowing security teams to automatically identify all services downstream of a given component that process customer data above a specified sensitivity class.

---

### Finding 3 — Customer Communication Playbook Gap

**Finding:** No pre-defined playbook existed for communicating a data handling finding to enterprise customers with contractual security commitments before a complete impact assessment was available.

**Impact:** The first customer communication was delayed by 6 hours while the security team worked through what could and could not be said at the time.

**Remediation:** Customer security escalation communication playbook developed — defining what to say at T+2 hours (acknowledgement and process), T+24 hours (preliminary impact assessment), T+72 hours (P0 remediation confirmation), and ongoing (weekly status until full remediation).

---

## Lessons Applicable to LLM Security at Scale

**1. Build the dependency graph before you need it in an incident.**
In this incident, the dependency graph had to be constructed under pressure. For LLM systems: document which downstream services consume outputs from each LLM component, what data classification those outputs carry, and what the exposure vector is — before an incident, not during one.

**2. Data classification drives prioritisation — not intuition.**
Without the classification framework, the natural pressure would have been to remediate all 26 services simultaneously, spreading remediation capacity too thin. The framework allowed the security team to make a defensible argument: these 3 services are the actual risk; remediating the other 23 first provides no additional protection. The same logic applies to LLM security: classify what data is in the LLM's context and what the exposure vector is before deciding which controls are urgent.

**3. Honest, structured customer communication is better than confident-sounding vague reassurance.**
The customer initially wanted to hear "your data is safe." The honest answer was "we don't have complete evidence of impact yet, here is what we know, here is our timeline, here is the data that makes our prioritisation defensible." That answer built more trust than a premature assurance would have.

**4. Unreasonable remediation timelines produce lower-quality remediations.**
Accepting the customer's 14-day-for-all-26-services demand would have required rushing P3/P4 services that had no actual customer data exposure — creating the risk of introducing new issues in services that were currently low-risk. Negotiating from a data-driven position allowed the team to deliver high-quality remediations on the services that actually mattered while managing the customer's expectations against the actual risk profile.

**5. Post-incident documentation improvements compound over time.**
The two structural improvements from this incident — mandatory data classification in security reviews, and automated dependency graph tooling — made every subsequent security review faster and every subsequent incident response more efficient. Invest in the systemic improvement, not just the immediate fix.
