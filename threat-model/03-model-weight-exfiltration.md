# Threat Category 03: Model Weight Exfiltration

**Severity:** High
**Applicability:** Type B (LLM Platforms), Type E (Custom Models)
**OWASP LLM Top 10:** LLM10 (Model Theft)
**MITRE ATLAS:** AML.T0044

> Full content for this threat category is in progress. Core concept: proprietary model weights represent significant IP value and, for safety-critical models, contain capabilities that should not be widely accessible. Exfiltration can occur via direct API access to inference infrastructure, insider access to model storage, or supply chain compromise. Assessment questions: Where are model weights stored and who has access? Are weights encrypted at rest and in transit? Is there monitoring for unusual inference patterns that could indicate model extraction via API?
