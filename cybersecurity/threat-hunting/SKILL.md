---
name: threat-hunting
description: Develop hypothesis-driven hunts across endpoint, identity, network, cloud, and application telemetry.
---

# Threat Hunting

## Hunt design
Start with a threat behavior or control gap, not a random query. State hypothesis, rationale, target population, time range, required telemetry, expected artifacts, and what would disprove the hypothesis.

## Data readiness
Check time synchronization, retention, field normalization, ingestion gaps, identity resolution, asset inventory, and query cost. Define a bounded time window and sampling strategy.

## Hunt loop
1. Build baseline and expected benign behavior.
2. Query broad enough to identify candidates but constrain resource cost.
3. Pivot across user, host, process, IP, domain, cloud principal, and application.
4. Validate candidates using independent telemetry.
5. Document false positives and alternative explanations.
6. Escalate confirmed incidents; create detections for repeatable behavior.
7. Record coverage gaps and follow-up hunts.

## Analytical patterns
Unusual parent-child process relationships, rare binaries, anomalous authentication, privilege changes, new persistence, abnormal DNS, unusual egress, suspicious cloud API sequences, and deviations from service-account baselines.

## Output
Hypothesis, data sources, queries, result counts, sample evidence, validation, conclusion, limitations, detection candidates, and next hunt.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
