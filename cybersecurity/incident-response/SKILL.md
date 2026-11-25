---
name: incident-response
description: Coordinate incident handling from preparation and containment through eradication, recovery, and lessons learned.
---

# Incident Response

## First principles
Preserve life/safety, service continuity, evidence integrity, and legal obligations. Follow the organization's incident command and communications plan. Do not improvise destructive containment.

## Lifecycle
1. Preparation: contacts, authority, backups, logging, forensic readiness, playbooks.
2. Identification: validate incident, classify, establish incident commander and severity.
3. Containment: choose reversible controls; document business impact and approval.
4. Evidence preservation: collect volatile data where appropriate, logs, images, cloud audit records, and hashes.
5. Eradication: remove root cause, persistence, compromised identities, and vulnerable paths.
6. Recovery: restore from trusted sources, rotate credentials, monitor heightened signals.
7. Lessons learned: timeline, control gaps, detection changes, owners, due dates, retest.

## Evidence handling
Record who collected what, when, from which system, using which method/tool/version, hash, storage location, and transfer history. Preserve originals. Minimize access to personal or regulated data.

## Containment decision
Assess attacker access, persistence, lateral movement, exfiltration risk, business criticality, and evidence volatility. Coordinate with system owners. Isolate or disable only under authority and with rollback/continuity plans.

## Communications
Use a single source of truth. Separate confirmed facts, estimates, and unknowns. Avoid speculative attribution. Coordinate legal, privacy, HR, communications, vendors, and regulators through approved channels.

## Deliverables
Incident summary, severity, timeline, affected assets/accounts/data, evidence index, containment/eradication/recovery log, notification decisions, root cause, residual risk, and action tracker.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
