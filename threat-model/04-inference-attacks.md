# Threat Category 04: Inference Attacks

**Severity:** Medium
**Applicability:** All types
**OWASP LLM Top 10:** LLM06 (Sensitive Information Disclosure)
**MITRE ATLAS:** AML.T0024, AML.T0025

> Full content for this threat category is in progress. Core concept: inference attacks use the model's outputs to extract information about its training data (membership inference, training data extraction) or internal structure (model inversion). Most relevant for models fine-tuned on sensitive data. Assessment questions: Was sensitive data used in fine-tuning? Has the model been tested for training data memorisation? Are there rate limits that would slow systematic inference probing?
