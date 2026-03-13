# Using LLMTrust Layer (A LLM Security Evaluation Framework - LSEF) in Your Organisation

This guide covers everything your team needs to adopt the LLM Security Evaluation Framework (LSEF) — from a single practitioner reading files on GitHub to a full organisational deployment with automated documentation builds and integrated security review workflows.

Choose the adoption path that matches your team's size, tooling maturity, and how seriously you want to embed LLM security into your development lifecycle.

---

## Adoption Paths at a Glance

| Path | Setup Time | Best For | What You Get |
|------|------------|----------|-------------|
| [1. Direct GitHub Reference](#path-1--direct-github-reference) | 0 minutes | Individual practitioners, initial evaluation | Framework access with no overhead |
| [2. Wiki Import](#path-2--confluence--notion-import) | 30–60 minutes | Small–medium security teams | Searchable, collaborative internal documentation |
| [3. MkDocs Documentation Site](#path-3--mkdocs-internal-documentation-site) | 1–2 hours | Teams wanting a professional internal security portal | Branded, searchable, auto-updating documentation site |
| [4. Fork and Customise](#path-4--fork-and-customise) | 2–4 hours | Teams wanting long-term ownership | Organisation-specific version you control and evolve |
| [5. Security Review Workflow Integration](#path-5--security-review-workflow-integration) | 1–3 days | Mature security teams with existing tooling | Framework embedded into Jira, ServiceNow, or review platforms |

---

## Path 1 — Direct GitHub Reference

**The zero-setup option.** Use the framework files directly from GitHub during security reviews.

### What to bookmark

| Need | File |
|------|------|
| Starting a new LLM security review | [`threat-model/README.md`](threat-model/README.md) |
| Pre-deployment sign-off | [`evaluation-checklists/pre-deployment-checklist.md`](evaluation-checklists/pre-deployment-checklist.md) |
| Scoring identified risks | [`risk-scoring/risk-scoring-model.md`](risk-scoring/risk-scoring-model.md) |
| Evaluating prompt injection risk | [`threat-model/01-prompt-injection.md`](threat-model/01-prompt-injection.md) |
| Evaluating an agentic system | [`threat-model/07-agentic-workflow-risks.md`](threat-model/07-agentic-workflow-risks.md) |
| Designing mitigations | [`mitigations/prompt-injection-mitigations.md`](mitigations/prompt-injection-mitigations.md) |
| Running an organisational LLM security program | [`mitigations/organisational-controls.md`](mitigations/organisational-controls.md) |

### How to use during a review

```
1. Open threat-model/threat-model-template.md
2. Copy the raw Markdown into a Google Doc, Confluence page, or Notion page
3. Fill in the system details and work through each threat category
4. Use the risk scoring model to score each identified threat
5. Document mitigations and owners
6. Record the disposition decision
```

### Stay updated

Star the repository on GitHub to receive notifications when new threat categories, checklists, or case studies are added.

---

## Path 2 — Confluence / Notion Import

Import the framework into your existing internal wiki to make it searchable, collaborative, and version-controlled alongside your other security documentation.

### Confluence

**Option A — Manual import (fastest):**

```
For each .md file you want to import:

1. Create a new Confluence page in your security space
2. Click Edit → click "..." menu → Insert Markup
3. Select "Markdown" from the dropdown
4. Paste the .md file contents
5. Click Insert → Save

Recommended Confluence space structure:
LLM Security Framework (space root)
├── Overview and Quick Start       ← README.md
├── Threat Model
│   ├── Methodology                ← threat-model/README.md
│   ├── 01 Prompt Injection        ← threat-model/01-prompt-injection.md
│   ├── 02 Training Data Poisoning
│   ├── 03 Model Weight Exfiltration
│   ├── 04 Inference Attacks
│   ├── 05 Supply Chain
│   ├── 06 Data Exfiltration
│   └── 07 Agentic Workflow Risks
├── Evaluation Checklists
│   └── Pre-Deployment Checklist
├── Risk Scoring Model
├── Mitigations
│   ├── Prompt Injection Mitigations
│   └── Organisational Controls
└── Case Studies
    ├── CS-01 Escalation Triage
    ├── CS-02 GovCloud Automation
    └── CS-03 Customer Escalation
```

**Option B — Confluence CLI (for teams with admin access):**

```bash
# Install Confluence CLI
pip install atlassian-python-api

# Bulk import script
python3 << 'EOF'
from atlassian import Confluence
import os

conf = Confluence(
    url='https://your-org.atlassian.net',
    username='your-email@org.com',
    password='your-api-token'
)

SPACE_KEY = 'LSEC'
PARENT_PAGE_ID = '123456'  # Your framework root page ID

files = [
    ('Threat Model — Prompt Injection', 'threat-model/01-prompt-injection.md'),
    ('Threat Model — Agentic Workflow Risks', 'threat-model/07-agentic-workflow-risks.md'),
    ('Pre-Deployment Checklist', 'evaluation-checklists/pre-deployment-checklist.md'),
    ('Risk Scoring Model', 'risk-scoring/risk-scoring-model.md'),
    ('Mitigations — Prompt Injection', 'mitigations/prompt-injection-mitigations.md'),
    ('Organisational Controls', 'mitigations/organisational-controls.md'),
]

for title, filepath in files:
    with open(filepath, 'r') as f:
        content = f.read()
    conf.create_page(
        space=SPACE_KEY,
        title=title,
        body=content,
        parent_id=PARENT_PAGE_ID,
        representation='wiki'
    )
    print(f"Created: {title}")
EOF
```

**Keeping Confluence in sync with framework updates:**

When the upstream framework publishes new content, re-run the import for updated files. For teams that want automated sync, set up a GitHub Action that triggers the import script on every new release tag.

---

### Notion

Notion is the most seamless import path — its Markdown import preserves tables, headers, and most importantly converts checklist items into interactive Notion checkboxes.

```
1. Create a new Notion page: "LLM Security Evaluation Framework"

2. Import all .md files:
   Settings & Members → Import → Text & Markdown
   Select all .md files from the framework directory

3. Organise into a Notion database for review tracking:
   Create a new Database (Table view) with properties:
   ├── System Name (text)
   ├── System Type (select: A/B/C/D/E)
   ├── Review Date (date)
   ├── Reviewer (person)
   ├── Disposition (select: Approved / Conditional / Hold / In Progress)
   ├── Risk Score — Highest (number)
   ├── Outstanding Items (number)
   └── Review Record (link to the filled-in checklist page)

4. Template setup:
   Create a Notion template from the filled threat-model-template.md
   Each new LLM system review = new database entry from template
```

**The pre-deployment checklist in Notion renders as a live interactive checklist** — reviewers can tick items directly in the review session rather than editing a document. This is the most effective collaborative review experience of any import option.

---

## Path 3 — MkDocs Internal Documentation Site

For teams that want a professional, searchable, auto-updating documentation portal — the same tooling used by major open-source projects and internal developer portals.

### Prerequisites

```bash
python 3.8+
pip
git
```

### Setup

```bash
# 1. Clone the framework
git clone https://github.com/amarwakh/LLMTrust-Layer
cd LLMTrust-Layer

# 2. Install MkDocs with Material theme
pip install mkdocs mkdocs-material

# 3. Create mkdocs.yml
cat > mkdocs.yml << 'EOF'
site_name: LLM Security Evaluation Framework
site_description: Practitioner-grade security evaluation for LLM and GenAI systems
site_author: Your Security Team

theme:
  name: material
  palette:
    - scheme: default
      primary: deep orange
      accent: orange
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      primary: deep orange
      accent: orange
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - search.highlight
    - search.suggest
    - content.code.copy

plugins:
  - search

nav:
  - Home: README.md
  - Threat Model:
    - Methodology: threat-model/README.md
    - 01 — Prompt Injection: threat-model/01-prompt-injection.md
    - 02 — Training Data Poisoning: threat-model/02-training-data-poisoning.md
    - 03 — Model Weight Exfiltration: threat-model/03-model-weight-exfiltration.md
    - 04 — Inference Attacks: threat-model/04-inference-attacks.md
    - 05 — Supply Chain: threat-model/05-supply-chain.md
    - 06 — Data Exfiltration: threat-model/06-data-exfiltration.md
    - 07 — Agentic Workflow Risks: threat-model/07-agentic-workflow-risks.md
    - Blank Template: threat-model/threat-model-template.md
  - Evaluation Checklists:
    - How to Use: evaluation-checklists/README.md
    - Pre-Deployment: evaluation-checklists/pre-deployment-checklist.md
  - Risk Scoring: risk-scoring/risk-scoring-model.md
  - Mitigations:
    - Prompt Injection: mitigations/prompt-injection-mitigations.md
    - Organisational Controls: mitigations/organisational-controls.md
  - Case Studies:
    - Overview: case-studies/README.md
    - CS-01 — Escalation Triage: case-studies/cs-01-llm-powered-escalation-triage.md
    - CS-02 — GovCloud Automation: case-studies/cs-02-govcloud-region-review-automation.md
    - CS-03 — Customer Escalation: case-studies/cs-03-data-services-customer-escalation.md
  - Contributing: CONTRIBUTING.md
EOF

# 4. Preview locally
mkdocs serve
# Open http://127.0.0.1:8000
# Full search, navigation, dark mode — all working

# 5. Build static site for deployment
mkdocs build
# Static site output in ./site/ directory
```

### Deployment options

**GitHub Pages (public, free, automatic):**

```yaml
# Create .github/workflows/deploy-docs.yml

name: Deploy Documentation Site
on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: pip install mkdocs mkdocs-material

      - name: Deploy to GitHub Pages
        run: mkdocs gh-deploy --force
```

Every push to `main` triggers an automatic rebuild and deployment. Your documentation site stays in sync with the repository at all times.

**AWS S3 + CloudFront (private internal hosting):**

```bash
# Build the site
mkdocs build

# Sync to S3 bucket (internal, private)
aws s3 sync ./site/ s3://your-internal-security-docs-bucket/llm-security-framework/ \
  --delete \
  --cache-control "max-age=3600"

# CloudFront distribution handles HTTPS and access control
# Restrict access to your corporate VPN or SSO via CloudFront signed URLs
```

**Self-hosted (Nginx):**

```bash
mkdocs build
scp -r ./site/* your-server:/var/www/security-docs/llm-framework/
```

---

## Path 4 — Fork and Customise

The recommended approach for any team planning long-term adoption. Fork the repository, make it yours, and evolve it alongside your organisation's specific needs while staying connected to upstream updates.

### Initial fork setup

```bash
# 1. Fork on GitHub:
#    github.com/amarwakh/LLMTrust-Layer
#    → Click Fork → your-org/LLMTrust-Layer

# 2. Clone your fork
git clone https://github.com/your-org/LLMTrust-Layer
cd LLMTrust-Layer

# 3. Add upstream remote to pull future updates from the original
git remote add upstream https://github.com/amarwakh/LLMTrust-Layer

# Verify remotes
git remote -v
# origin    https://github.com/your-org/LLMTrust-Layer (your fork)
# upstream  https://github.com/amarwakh/LLMTrust-Layer (original)
```

### What to customise in your fork

**1. Add your organisation's threat categories**

Create new files in `threat-model/` for threats specific to your architecture:

```
threat-model/
├── 08-internal-threat-[your-specific-system].md
├── 09-[your-regulatory-environment]-specific-risks.md
```

**2. Add internal case studies**

Unlike the public case studies which are anonymised, your internal fork can reference actual system names, real metrics, and real teams:

```
case-studies/
├── cs-internal-01-[your-system-name].md
├── cs-internal-02-[your-incident-name].md
```

**3. Add organisation-specific control requirements**

If your organisation operates under specific compliance frameworks (FedRAMP, HIPAA, SOC2 Type II, PCI-DSS), add requirement overlays:

```
compliance-overlays/
├── fedramp-high-overlay.md       ← Additional controls for FedRAMP High
├── hipaa-overlay.md              ← PHI-specific requirements
├── pci-dss-overlay.md            ← Cardholder data requirements
└── internal-security-bar.md      ← Your organisation's internal security bar
```

**4. Customise the risk scoring model**

Your organisation may weigh impacts differently — for example, reputational impact may be more critical than the default scoring implies:

```markdown
# In risk-scoring/risk-scoring-model.md, add:

## Organisation-Specific Calibration

Our scoring applies the following adjustments to the base model:
- Regulatory data (S1) Impact-Confidentiality: always score minimum 4
- Consumer-facing systems: add 1 to base Exploitability score
- Systems with >1M daily active users: multiply final score by 1.5
```

### Pulling upstream updates into your fork

```bash
# Fetch latest from the original framework
git fetch upstream

# Merge upstream changes into your main branch
git checkout main
git merge upstream/main

# If there are conflicts (your customisations vs upstream changes):
# Git will mark conflict sections — resolve manually, then:
git add .
git commit -m "Merge upstream framework updates v0.2.0"
git push origin main
```

**Recommended practice:** Subscribe to releases on the upstream repository. When a new version is tagged, review the changelog, then pull the update into your fork.

---

## Path 5 — Security Review Workflow Integration

For teams with established security review tooling — integrate the framework directly into Jira, ServiceNow, or custom review platforms so it is part of your existing workflow rather than a separate step.

### Jira — Security Review Epic Template

Create a reusable Jira Epic template for LLM system security reviews. Each new LLM system that needs review gets its own Epic from this template.

```
Epic: LLM Security Review — [System Name]
Labels: llm-security-review
Components: Security Engineering

Stories (created automatically from template):

┌─────────────────────────────────────────────────────────┐
│ Story 1: System Characterisation                         │
│ Description: Complete system characterisation section    │
│ of threat-model-template.md                             │
│ Acceptance criteria:                                     │
│   □ System type classified (A/B/C/D/E)                  │
│   □ Data inventory documented                            │
│   □ Trust boundaries mapped                             │
│   □ Actor set defined                                    │
│   □ Tool/action inventory complete (agentic systems)    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Story 2: Threat Model                                    │
│ Description: Work through all applicable threat          │
│ categories from lsef threat-model/                       │
│ Acceptance criteria:                                     │
│   □ All 7 threat categories assessed                    │
│   □ Applicable threats scored using risk-scoring-model  │
│   □ Risk owners assigned for all High/Critical threats  │
│   □ Threat model document linked in this ticket         │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Story 3: Pre-Deployment Checklist                        │
│ Description: Complete pre-deployment-checklist.md        │
│ Acceptance criteria:                                     │
│   □ All CRITICAL items: Resolved or formally accepted   │
│   □ All HIGH items: Resolved or remediation plan dated  │
│   □ Checklist document linked in this ticket            │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Story 4: Disposition Decision                            │
│ Description: Security team disposition                   │
│ Acceptance criteria:                                     │
│   □ Disposition recorded: Approved / Conditional / Hold │
│   □ All outstanding items tracked with owners and dates │
│   □ Next review date set                                │
│   □ Stakeholders notified                               │
└─────────────────────────────────────────────────────────┘
```

**Jira Automation rule to enforce the gate:**

```
Trigger: Issue transitioned to "Ready for Production"
Condition: Issue type = "Feature" AND label contains "llm-feature"
Action: Check if linked Epic "LLM Security Review" exists
        AND Epic status = "Done"
If not: Block transition + comment "LLM security review required before production"
```

---

### ServiceNow GRC — Risk Register Integration

The LSEF risk scoring model maps directly to ServiceNow GRC's risk register structure.

```
ServiceNow Risk Record fields → LSEF mapping:

Risk Name          → Threat category name (e.g., "Prompt Injection — Direct")
Risk Description   → Threat description from threat-model/
Likelihood         → LSEF Likelihood score (1-4)
Impact             → LSEF composite Impact score ((IC + IA) / 2)
Risk Score         → LSEF Raw Score (L × Impact × E)
Risk Rating        → LSEF Rating (Low / Medium / High / Critical)
Risk Owner         → Assigned security engineer
Treatment Plan     → Linked mitigations from mitigations/
Review Date        → Next review date per organisational cadence
```

**Import script for bulk risk register population:**

```python
import pysnow
import json

client = pysnow.Client(
    instance='your-instance',
    user='service-account',
    password='password'
)

# Load your completed risk scoring template
with open('risk-scoring/completed-assessment-[system].json') as f:
    risks = json.load(f)

risk_table = client.resource(api_path='/table/sn_risk_risk')

for risk in risks:
    risk_table.create(payload={
        'name': risk['threat'],
        'description': risk['description'],
        'likelihood': risk['likelihood'],
        'impact': risk['impact_composite'],
        'score': risk['raw_score'],
        'category': 'LLM Security',
        'subcategory': risk['threat_category'],
        'owner': risk['owner_email'],
        'treatment': risk['mitigations_applied'],
        'review_date': risk['next_review_date']
    })
    print(f"Created risk record: {risk['threat']}")
```

---

### Custom Security Review Platform — REST API Integration

If your organisation has a custom security review platform or uses a tool like Drata, Vanta, or Tugboat Logic, the framework content can be integrated via their APIs or custom control libraries.

**General pattern:**

```python
# Generic pattern for importing LSEF checklist items as controls
# into any GRC or review platform with a REST API

import requests
import re

def parse_checklist_items(filepath):
    """
    Parse LSEF checklist markdown into structured control objects.
    Returns list of dicts with: text, priority, section
    """
    controls = []
    current_section = None
    
    with open(filepath) as f:
        for line in f:
            # Detect section headers
            if line.startswith('## Section'):
                current_section = line.strip('# \n')
            
            # Detect checklist items with priority tags
            match = re.match(
                r'- \[ \] \*\*\[(CRITICAL|HIGH|MEDIUM|LOW)\]\*\* (.+)', 
                line
            )
            if match:
                controls.append({
                    'priority': match.group(1),
                    'text': match.group(2).strip(),
                    'section': current_section,
                    'source': 'LSEF Pre-Deployment Checklist'
                })
    
    return controls

# Parse the checklist
controls = parse_checklist_items(
    'evaluation-checklists/pre-deployment-checklist.md'
)

# POST to your platform's API
for control in controls:
    response = requests.post(
        'https://your-review-platform.com/api/controls',
        headers={'Authorization': 'Bearer YOUR_API_KEY'},
        json={
            'name': control['text'][:100],
            'description': control['text'],
            'priority': control['priority'],
            'category': control['section'],
            'framework': 'LSEF',
            'framework_version': '0.1.0'
        }
    )
    print(f"Imported: [{control['priority']}] {control['text'][:60]}...")

print(f"\nTotal controls imported: {len(controls)}")
```

---

## Keeping Your Deployment Current

The LLM threat landscape changes rapidly. Framework updates will be tagged as releases on GitHub. Here is how to stay current for each adoption path:

| Path | Update Mechanism |
|------|-----------------|
| Direct GitHub reference | Star the repo; watch for release notifications |
| Confluence / Notion | Re-import updated files when new versions are tagged |
| MkDocs with GitHub Actions | Automatic — every push to main rebuilds the site |
| Forked repo | `git fetch upstream && git merge upstream/main` on each release |
| Jira / ServiceNow | Re-run import scripts against updated checklist files |

**Recommended:** Subscribe to repository releases (GitHub → Watch → Custom → Releases) to receive an email notification for each new version. The changelog will document what changed and whether your deployment needs updating.

---

## Questions and Support

If you have questions about adoption, find a gap in the framework, or want to contribute content from your own experience:

- **Open an issue:** For questions, bug reports, or content gaps
- **Submit a pull request:** For contributions — see [CONTRIBUTING.md](CONTRIBUTING.md)
- **LinkedIn:** Connect with the framework author for direct conversation on adoption challenges

The framework gets better when practitioners who have deployed it in real organisations share what they found.
