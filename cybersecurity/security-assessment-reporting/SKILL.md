---
name: security-assessment-reporting
description: Produce evidence-backed security findings, severity ratings, remediation plans, and professional assessment reports.
---

# Security Assessment Reporting

## Finding standard
Each finding must contain:
- ID and concise title.
- Severity, rationale, and confidence (separate dimensions).
- Affected asset, endpoint/file/line, component and version when known.
- Preconditions and attacker capabilities.
- Evidence with timestamps and redaction.
- Minimal reproducible steps and expected vs actual behavior.
- Technical impact and realistic business impact.
- CWE/CVE only when accurately mapped and verified.
- Root-cause remediation, defense-in-depth controls, and validation test.
- References and status (`confirmed`, `needs-validation`, `accepted-risk`, `fixed`, `retest-passed`).

## Severity
Use CVSS 4.0 where sufficient data exists; record vector and assumptions. Do not derive a precise score from missing facts. Explain exploitability, exposure, privileges, user interaction, scope, and impact. Business risk may differ from CVSS; show both rather than forcing one to replace the other.

## Evidence quality
Prefer raw request/response pairs, sanitized logs, source excerpts, configuration, screenshots, hashes, and reproducible tests. Redact credentials, session identifiers, personal data, and unrelated records. Keep original evidence in a controlled location and reference its ID/hash.

## Report structure
1. Executive summary and overall risk.
2. Scope, exclusions, dates, and rules of engagement.
3. Methodology, tools, versions, and limitations.
4. Findings summary and risk distribution.
5. Detailed findings.
6. Positive controls and observations.
7. Remediation roadmap (immediate, short-term, strategic).
8. Retest results and residual risk.
9. Appendices: assets, evidence index, glossary.

## Quality gates
- No unsupported claims or duplicate findings.
- Severity matches demonstrated impact and prerequisites.
- Reproduction steps are safe and scoped.
- Remediation addresses root cause, not only symptoms.
- Each finding has a testable acceptance criterion.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
