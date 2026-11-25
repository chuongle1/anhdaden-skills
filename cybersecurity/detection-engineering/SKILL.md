---
name: detection-engineering
description: Design, implement, test, and tune SIEM/EDR detections with ATT&CK mapping and measurable operational quality.
---

# Detection Engineering

## Detection lifecycle
Define behavior and threat hypothesis → identify data sources → model telemetry → write analytic → test against positive/negative cases → deploy safely → monitor quality → tune and version.

## Rule requirements
- Clear name, purpose, author, version, status, severity, confidence.
- Data source and required fields; parser/schema assumptions.
- Query/rule logic with time window and aggregation semantics.
- ATT&CK technique mapping only when behavior supports it.
- Positive test events, negative/benign cases, and expected results.
- False-positive causes, exclusions, and tuning guidance.
- Response steps, triage pivots, and telemetry health checks.

## Quality metrics
Measure true/known-positive coverage, false-positive rate, alert volume, precision where labels exist, time-to-detect, data-source availability, and rule execution health. Avoid optimizing only for low alert volume.

## Sigma/SIEM
Use portable Sigma where suitable, then validate backend translation and field mappings. For Splunk/KQL/Elastic/Sentinel, document query dialect and index/table assumptions. Do not invent field names; inspect schemas.

## Testing
Use synthetic events or approved replay datasets. Test time boundaries, missing fields, case sensitivity, nulls, aggregation, allowlists, and evasion-adjacent variants without building stealth tooling.

## Output
Detection spec, rule source, test corpus, expected outputs, ATT&CK mapping, tuning notes, deployment plan, rollback, and monitoring metrics.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
