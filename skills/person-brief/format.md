# Person Brief — Structured Format

`person-to-brief-af` emits `person-brief.md` in this shape.

## Frontmatter

```yaml
subject: <full name>
company: <company, if known>
identity_status: resolved|partially-resolved|ambiguous
researched_at: <UTC timestamp>
sources_used: <count>
```

## Sections (all required; empty sections say "none found")

1. **Executive summary** — 3–5 lines: who they are, why they matter here.
2. **Priorities** — what matters for the requesting context (deal, hire, partnership, threat).
3. **Recent signals** — dated items, each with citation.
4. **Conversation starters** — 3–5 openers grounded in cited signals.
5. **Messaging angles** — 2–4 angles, each with supporting citation.
6. **Cautions** — verification gaps, contradictory signals, do-not-assume items.

## Citation rule

Every claim in sections 2–5 carries a Markdown link to its source. A claim that cannot be cited goes in Cautions, not in the body. Identity ambiguity must be stated in the frontmatter and reflected in the summary.