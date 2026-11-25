---
name: devsecops-automation
description: Integrate security checks into CI/CD and automate vulnerability, secrets, dependency, container, and IaC workflows.
---

# DevSecOps Automation

## Pipeline threat model
Identify source control, runners, build identities, artifact stores, registries, deployment credentials, environments, approvals, and trust boundaries. Treat CI runners and build artifacts as high-value assets.

## Controls
- Branch protection, code review, signed commits/tags where appropriate.
- Least-privilege short-lived workload identity; avoid long-lived static tokens.
- Isolated ephemeral runners; restrict untrusted PR access to secrets.
- Dependency pinning, lockfiles, provenance, SBOM, signature verification.
- SAST, secret scanning, SCA, container scanning, IaC scanning, DAST where scoped.
- Policy gates based on exploitability, reachability, severity, and ownership.
- Artifact signing/attestation and deployment verification.
- Secret masking, log hygiene, protected environments, auditable approvals.

## Automation design
Make jobs deterministic, idempotent, bounded, and observable. Define timeouts, retries, rate limits, failure semantics, and rollback. Separate scanner execution errors from clean results. Preserve raw machine-readable output and normalize findings without losing provenance.

## Triage
Deduplicate by stable fingerprint; map to repository, commit, file, line, package, owner, and lifecycle status. Avoid treating scanner presence as confirmed exploitability. Define exception owner, justification, expiry, and compensating controls.

## Output
Pipeline diagram, control matrix, sample workflow/config, policy thresholds, artifact handling, metrics, failure modes, and rollout plan.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
