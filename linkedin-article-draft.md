# LinkedIn Article — Draft

**Title:** What I Learned Securing GenAI Systems at Scale — And Why I Built an Open Framework

---

Two years ago, my security team at AWS faced a problem I had not seen documented anywhere: how do you maintain a consistent security bar when your own engineers are building and deploying GenAI tools faster than traditional security review processes can keep up?

We were not evaluating whether to use AI — it was already in production. Engineers across the organisation were using LLM-powered tools for security escalation triage, automated risk assessments, and region review automation for regulated deployments like GovCloud and FedRAMP environments. Each system had a different threat profile. Prompt injection risk in a tool that could auto-route security escalations was categorically different from prompt injection risk in a customer support chatbot. A RAG system retrieving from historical security reviews needed a different access control model than one retrieving from public documentation. We needed a way to evaluate these systems consistently — not just for the first deployment, but as a repeatable practice across a growing portfolio of GenAI tooling.

What we built was not elegant at first. It was a set of evaluation checklists, a dependency-aware risk scoring model, and a classification framework for the data these systems touched — developed through real security reviews of real production systems. Over time, it became the foundation for how our team approached every new GenAI deployment: classify the system type, map the trust boundaries, enumerate the threats specific to that architecture, score them, and build mitigations proportional to actual risk. When we applied this to our GenAI-powered escalation triage system — which reduced investigation time by 60% across our security organisation — we identified and mitigated a prompt injection risk that could have allowed a malicious actor to systematically miscategorise security findings. When we applied it to GovCloud region review automation, we caught a data residency gap that would have been a FedRAMP compliance failure. The framework paid for itself before we finished writing it.

I have open-sourced this as the LLMTrust Layer because the security community needs practitioner-grade material on GenAI security — not vendor whitepapers, not theoretical academic models, but the actual evaluation criteria that security engineers can pick up and use on Monday morning. The framework covers seven threat categories (with particular depth on prompt injection and agentic workflow risks), a pre-deployment evaluation checklist with 50+ structured items, a practical risk scoring model, mitigation pattern libraries, and three case studies drawn from real production systems. It is not finished — the LLM threat landscape moves too fast for any framework to be finished — but it is real, and it is yours to use, adapt, and improve.

The repository is at: **github.com/amarwakh/LLMTrust-Layer**

If you are building or evaluating GenAI systems and want to compare notes on what the real security challenges look like in production — I would genuinely enjoy the conversation.

---

**Suggested hashtags:** #LLMSecurity #AISecuity #GenAI #ApplicationSecurity #CloudSecurity #AWSecurity #AIRisk #CyberSecurity #SecurityEngineering

**Suggested image:** A clean diagram of the framework structure (the directory tree from the README rendered as a visual) or a simple graphic showing the 7 threat categories.

**Optimal posting time:** Tuesday or Wednesday morning Pacific time — highest LinkedIn engagement for technical content.

**One additional action:** When you post this, tag 2–3 people from the OWASP AI Security project or well-known LLM security researchers. This triggers notifications, extends reach, and signals to recruiters that you are engaged with the relevant community — not just publishing into a void.
