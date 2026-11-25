---
name: appsec-code-review
description: Perform source-code and architecture security reviews, tracing vulnerabilities from entry point to security impact.
---

# AppSec Code Review

## Review setup
Identify language/framework, build system, entry points, trust boundaries, authentication model, deployment config, dependencies, and security requirements. Read relevant code and tests before making claims.

## Review method
1. Map sources, transformations, sinks, and authorization boundaries.
2. Trace user-controlled data across layers.
3. Inspect security-critical logic and negative paths.
4. Review dependency and configuration risks.
5. Validate suspected flaws with minimal tests or static reasoning.
6. Search for variants and missing regression tests.
7. Produce evidence-backed findings.

## Priority areas
- Authentication/session/token lifecycle.
- Authorization at object, function, tenant, and data layers.
- Injection and unsafe output encoding.
- Path/file/archive handling and SSRF.
- Cryptography, randomness, secret handling.
- Deserialization, template engines, dynamic evaluation.
- Race conditions, concurrency, transaction integrity.
- Logging, error handling, privacy, and audit trails.
- Dependency, build, CI/CD, container, and deployment configuration.

## Evidence
Cite file path, line range, call chain, relevant configuration, and conditions. Explain why a control is missing or bypassable. Distinguish reachable vulnerabilities from dead code or theoretical patterns.

## Remediation
Prefer root-cause changes, central controls, secure framework APIs, regression tests, and defense-in-depth. Avoid merely suppressing a scanner warning.

## Output
Architecture summary, review coverage, findings, false-positive/uncertain items, prioritized fixes, and tests not run.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
