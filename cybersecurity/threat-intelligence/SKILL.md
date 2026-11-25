---
name: threat-intelligence
description: Collect, evaluate, contextualize, and operationalize cyber threat intelligence without overstating attribution.
---

# Threat Intelligence

## Intelligence requirements
Clarify decision, audience, sector/geography, time horizon, and required confidence. Distinguish strategic, operational, and tactical intelligence.

## Source evaluation
Assess provenance, recency, corroboration, collection method, potential bias, and confidence. Prefer primary advisories, vendor research with transparent methods, CERTs, and validated telemetry. Treat open feeds and social posts as leads, not ground truth.

## IOC handling
Normalize type/value, defang for display, record source, first/last seen, confidence, context, expiration, and false-positive considerations. Consider shared hosting, CDNs, sinkholes, VPNs, and dynamic infrastructure before blocking.

## Attribution
Do not attribute an intrusion based only on malware family, infrastructure, language, or a single TTP. Use confidence levels and competing hypotheses. Separate actor assessment from observed behavior.

## Operationalization
Translate intelligence into hunt hypotheses, detection logic, risk advisories, patch priorities, and stakeholder actions. Track whether indicators remain useful and retire stale items.

## Output
Executive assessment, key judgments with confidence, evidence and sourcing, observed TTPs, affected sectors/technologies, defensive actions, collection gaps, and review date.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
