# Security / Cloud Findings — Structured Format

Both `sec-af` (security-findings.md) and `cloudsecurity-af` (cloud-findings.md) emit findings in this schema.

## Frontmatter

```yaml
target: <repo path or IaC path>
audit_type: code | iac
mode: static | live          # live only when cloud credentials were used
severity_threshold: <low|medium|high|critical>
compliance_frameworks: [<CIS|SOC2|HIPAA|...>]
fail_on_findings: <true|false>
scanned_at: <UTC timestamp>
```

## Findings (array)

For each finding, emit:

```yaml
- id: SEC-01
  title: <one line>
  severity: critical|high|medium|low
  location: <file:line>
  framework: <mapping, or none>
  evidence: <trace / reproduction path>
  impact: <what an attacker gains>
  fix: <concrete remediation>
  status: confirmed|investigated-not-proven
```

## Report body

1. **Summary line**: `N findings — X critical, Y high, Z medium, W low (A investigated but not proven)`.
2. **Confirmed findings** (severity-ranked, full fields above).
3. **Investigated, not proven** — listed but out of scope for blocking.
4. **Skipped items** — e.g. live pass skipped for missing credentials.
5. **Remediation order** — suggested sequence of fixes.

Never emit a finding whose `evidence` field is empty.