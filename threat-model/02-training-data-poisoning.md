# Threat Category 02: Training Data Poisoning

**Severity:** High
**Applicability:** Type E (Fine-Tuned/Custom Model), Type B (LLM Platforms)
**OWASP LLM Top 10:** LLM03
**MITRE ATLAS:** AML.T0020

> Full content for this threat category is in progress. See the threat modeling README for methodology. Core concept: an attacker who can influence the training or fine-tuning data for a model can introduce backdoors, bias outputs toward specific behaviours, or degrade model performance on targeted inputs. Assessment questions: Who has write access to your training data pipeline? Is training data provenance auditable? Has the model been evaluated for anomalous behaviours on out-of-distribution inputs before deployment?
