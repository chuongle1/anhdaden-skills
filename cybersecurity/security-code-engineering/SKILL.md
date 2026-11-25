---
name: security-code-engineering
description: Design, implement, and test secure production-quality security tooling and application code in Python, Go, shell, and SQL.
---

# Security Code Engineering

## Engineering contract
Before coding, inspect the repository conventions, runtime, dependency versions, interfaces, tests, and deployment model. Preserve existing behavior unless change is requested. Prefer complete working files over fragments when the user asks for implementation.

## Secure implementation checklist
- Validate input at trust boundaries; use allowlists and canonicalization appropriate to the format.
- Encode output for its destination; avoid shell, SQL, template, LDAP, and path injection.
- Apply least privilege and explicit authorization checks on every sensitive operation.
- Use secure random generators for tokens; standard vetted cryptographic libraries; never design custom crypto.
- Store secrets in a secret manager/environment injection; never hardcode or log them.
- Set connect/read/overall timeouts, response size limits, concurrency limits, and cancellation.
- Use TLS with certificate verification; do not normalize `verify=False` or insecure cipher settings.
- Use safe temporary files, restrictive permissions, atomic writes, and symlink-aware path handling.
- Avoid unsafe deserialization and untrusted dynamic code execution.
- Log security-relevant events without secrets or unnecessary personal data.
- Pin or constrain dependencies and document provenance.

## Language patterns
### Python
Use type hints, `pathlib`, context managers, `subprocess.run([...], shell=False, timeout=...)`, explicit encodings, bounded reads, and `pytest`. Catch specific exceptions. Do not use `eval`/`exec` on untrusted data.

### Go
Propagate `context.Context`; use `http.Client` with timeouts; close response bodies; limit body reads; check every error; use parameterized SQL; run `go test`, `go vet`, and race tests where appropriate.

### Shell
Use `set -euo pipefail` where compatible, quote expansions, use `--` before user-controlled paths, validate arguments, avoid `eval`, and require explicit confirmation for destructive operations. Never print secrets in `set -x` traces.

### SQL
Use parameterized statements, transaction boundaries, least-privilege roles, and safe migrations. Avoid string concatenation for query construction.

## Tests
Include unit tests for normal, boundary, malformed, unauthorized, timeout, and failure cases. Add integration tests only where environment and scope permit. Record tests actually run and results; do not claim unrun tests passed.

## Review output
Summarize design, files changed, security controls, test commands/results, compatibility risks, and remaining limitations.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
