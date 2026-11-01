# security-audit

Run a full security audit of a repository yourself and produce a severity-ranked finding report. Owned by `sec-af`; the same output schema is used by `cloudsecurity-af` for its findings.

## When to use
- A repository or codebase change needs a security audit (recon → hunt → prove → remediate).
- An IaC or cloud configuration needs a scan (static, or live when credentials are provided).

## Procedure
1. Recon: architecture map, dependency audit for known-vulnerable versions, configuration scan.
2. Hunt: trace each candidate from input to sink across the code.
3. Prove: confirm exploitability with a concrete path or minimal reproduction; set aside what you cannot prove.
4. Remediate: file, line, and fix for every confirmed finding.

## Non-negotiables
- Never report a finding you have not traced to evidence.
- Live cloud pass only with provided credentials; otherwise static scan only and say the live pass was skipped.
- Ranked by severity against the compliance frameworks in force.

## Output
`security-findings.md` (or `cloud-findings.md`) in the format defined by `format.md`.