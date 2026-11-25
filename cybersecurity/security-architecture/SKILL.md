---
name: security-architecture
description: Design and review secure system architectures using threat modeling, trust boundaries, defense in depth, and security requirements.
---

# Security Architecture

## Architecture intake
Understand business flows, data classification, actors, deployment topology, integrations, identity model, tenancy, availability requirements, regulatory constraints, and operational ownership.

## Threat modeling
Use a suitable method such as STRIDE, attack trees, misuse cases, or data-flow analysis. Identify assets, entry points, trust boundaries, security assumptions, threats, mitigations, and residual risks. Do not treat a threat-model worksheet as proof that controls work.

## Architecture domains
- Identity: federation, MFA, workload identity, authorization, session lifecycle.
- Data: classification, minimization, encryption, key management, retention, deletion.
- Network: segmentation, ingress/egress, service-to-service trust, private connectivity.
- Application: secure defaults, input/output boundaries, tenant isolation.
- Platform: hardened compute, container isolation, patching, secrets.
- Resilience: backups, recovery, failure modes, abuse resistance.
- Observability: audit events, detection, traceability, privacy.
- Supply chain: build provenance, dependency controls, artifact integrity.

## Design principles
Least privilege, fail secure, explicit trust boundaries, secure-by-default, defense in depth, separation of duties, minimize attack surface, assume breach, and usable controls.

## Decision records
For each major decision, record context, options, selected design, rationale, security trade-offs, owner, and review trigger. Identify compensating controls and expiry for exceptions.

## Output
Context/data-flow diagrams, threat register, security requirements, architecture decisions, residual risks, and verification plan.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
