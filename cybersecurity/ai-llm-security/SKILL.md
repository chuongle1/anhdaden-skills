---
name: ai-llm-security
description: Threat-model and assess AI/LLM applications, agents, RAG pipelines, tool use, model interfaces, and AI supply chains.
---

# AI and LLM Security

## System map
Document model/provider, system/developer prompts, user inputs, tools/functions, agent loop, memory, retrieval corpus, vector store, plugins/connectors, output consumers, logging, and human approval points. Identify where untrusted content crosses trust boundaries.

## Threat areas
- Direct and indirect prompt injection.
- Tool/function abuse, excessive agency, confused deputy, and unsafe delegation.
- RAG poisoning, retrieval manipulation, cross-tenant leakage, and source spoofing.
- Sensitive data disclosure through prompts, context, logs, memory, or outputs.
- Insecure output handling: HTML/SQL/shell/code injection downstream.
- Authentication/authorization gaps in tools and connectors.
- Model supply chain, untrusted models/adapters, datasets, and dependencies.
- Denial of wallet/service, resource exhaustion, and runaway agent loops.
- Overreliance, unsafe automation, and insufficient human oversight.

## Assessment method
1. Define assets, users, trust boundaries, model/tool permissions, and impact.
2. Create adversarial test cases mapped to a threat taxonomy (including OWASP LLM guidance where relevant).
3. Test controls with synthetic data and non-destructive tool mocks.
4. Verify authorization at the tool boundary, not only in prompt text.
5. Check output validation/encoding and downstream execution.
6. Test retrieval access control and tenant isolation.
7. Evaluate logging, retention, redaction, rate limits, budgets, and circuit breakers.
8. Document residual risk and human approval requirements.

## Design controls
Treat model output and retrieved content as untrusted. Enforce policy in deterministic code. Use least-privilege scoped tools, explicit user confirmation for consequential actions, schema validation, sandboxing, egress restrictions, secrets isolation, provenance, and audit trails. Do not rely on prompt instructions as the only security boundary.

## Output
System/data-flow diagram, threat register, test corpus, observed behavior, control gaps, mitigations, and regression tests. Clearly distinguish model refusal from actual access control.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
