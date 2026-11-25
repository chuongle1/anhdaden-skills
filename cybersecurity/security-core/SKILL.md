---
name: security-core
description: Apply authorization, evidence discipline, security methodology, scope control, and safe validation across cybersecurity tasks.
---

# Security Core

## Purpose
This is the mandatory foundation for any security task. Apply it before domain-specific work.

## Intake and scope
Establish:
- Objective and decision the work must support.
- Asset owner, environment, exact targets, exclusions, and third-party dependencies.
- Written authorization, permitted methods, time window, rate limits, and prohibited actions.
- Data classification, retention, evidence storage, notification path, emergency contact, and stop conditions.
- Whether testing is passive, authenticated, active, destructive, or production-impacting.

If authorization is absent or ambiguous, continue with passive analysis, design review, lab reproduction, or a proposed test plan. Do not initiate active testing.

## Evidence model
Tag each material statement:
- `OBSERVED`: directly present in supplied artifacts or tool output.
- `REPRODUCED`: independently reproduced under documented conditions.
- `INFERRED`: follows from observed facts but was not directly tested.
- `HYPOTHESIS`: plausible explanation requiring validation.
- `UNKNOWN`: unavailable or contradictory.

Preserve original evidence. Record source, collector, UTC timestamp, timezone, hash, environment, command/tool version, and chain-of-custody notes where relevant. Never fabricate missing output.

## Workflow
1. Frame objective, scope, assumptions, and success criteria.
2. Inventory assets and trust boundaries.
3. Inspect evidence and establish baseline.
4. Build attack/defense hypotheses.
5. Select the least invasive discriminating test.
6. Execute only within scope; capture raw output and timestamps.
7. Evaluate impact, likelihood, confidence, and alternatives.
8. Recommend root-cause remediation and compensating controls.
9. Verify fixes and document residual risk.

## Safety gates
Stop and ask for approval before:
- Destructive or irreversible changes.
- Denial-of-service, load testing, mass scanning, or broad fuzzing.
- Accessing, modifying, or exporting real user data.
- Persistence, lateral movement, credential harvesting, or evasion in a live environment.
- Testing third-party systems not explicitly included in scope.

## Output
Produce a short scope statement, methodology, evidence ledger, findings or negative results, limitations, and next steps. A lack of findings is not proof of security; state coverage and blind spots.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
