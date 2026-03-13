# Threat Category 06: Sensitive Data Exfiltration via LLM

**Severity:** High
**Applicability:** Type A, Type C, Type D
**OWASP LLM Top 10:** LLM06

> Full content for this threat category is in progress. Core concept: LLMs with access to sensitive data (via RAG, tool access, or context injection) can be manipulated to output that data — either through direct extraction attempts or by embedding sensitive data in outputs that are then transmitted to an attacker-controlled channel (particularly in agentic systems). Assessment questions: What sensitive data could appear in the LLM's context? Are there output content classifiers that would detect sensitive data patterns in LLM outputs? In agentic systems, could the LLM be caused to transmit data to an external endpoint?
