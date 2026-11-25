---
name: red-team-operations
description: Plan controlled adversary emulation, attack-path analysis, and purple-team exercises under explicit rules of engagement.
---

# Red Team Operations

## Governance
Require signed rules of engagement covering objectives, target scope, permitted TTPs, safety limits, data handling, persistence policy, social engineering boundaries, third-party approvals, notification/escalation, and stop conditions. Define a deconfliction method with defenders.

## Plan
Translate business objectives into measurable outcomes. Map relevant adversary behaviors to MITRE ATT&CK techniques. Define exercise phases, safety controls, success criteria, and evidence requirements. Prefer emulation plans over unconstrained intrusion.

## Execute safely
Use dedicated test identities and infrastructure where possible. Keep actions reversible and minimally invasive. Do not deploy persistence, collect credentials, access sensitive data, evade safety controls, or move laterally unless those exact activities are approved. Never exceed the approved objective.

## Operational records
Maintain an activity log: timestamp, operator, source, target, technique, command/action, expected signal, observed result, and cleanup status. Protect payloads and collected data. Stop on out-of-scope access or unexpected production impact.

## Purple-team loop
For each technique: objective → controlled action → telemetry observed → detection result → analyst response → gap → tuning → retest. Preserve both attacker and defender evidence.

## Deliverables
Executive narrative, attack-path diagram, ATT&CK mapping, control/detection gaps, timeline, safety/deconfliction notes, cleanup verification, and prioritized improvements. Clearly distinguish simulated effects from actual compromise.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
