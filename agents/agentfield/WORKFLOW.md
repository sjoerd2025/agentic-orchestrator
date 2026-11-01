# AgentField Squad — Agent Orchestration Workflow

Squad: `@agentfield` (Multica squad `1906884e-3372-4ef8-ab92-f73c96e8b341`, leader **Kiro** `81c02f47-f7df-4df0-bb05-b1d3fff37872`) · sits next to the `@feature-squad` workflow (see `reply_beta26.md`).

This workflow is about **orchestration**: one conductor, five specialized worker agents, artifacts as the currency between them, and explicit gates at every handoff. The workers run natively on the Hermes runtime — they are the capability, not wrappers around the AgentField control plane (no `af run`, no AgentField HTTP API).

## 1. The orchestration graph

```
                 ┌─────────────────────────────┐
   Task Request  │  Kiro — conductor / router  │
   ─────────────▶│  - classify, route, gate    │
                 │  - never does node work     │
                 └──────────────┬──────────────┘
                                │ route (dispatch matrix)
        ┌───────────────┬───────┼────────┬───────────────┐
        ▼               ▼       ▼        ▼               ▼
   swe-planner    sec-af   cloudsecurity pr-af    person-to-brief-af
   (engineer)   (auditor)   -af (IaC)  (reviewer)    (intel)
        │               │       │        │               │
        └───────────────┴───────┴────────┴───────────────┘
                                │ artifacts (verified by gate)
                                ▼
                 ┌─────────────────────────────┐
                 │ Kiro — verify & publish     │
                 │ supervisor aftercare        │
                 └─────────────────────────────┘
```

- **Conductor (Kiro)**: classifies the task, routes it to exactly one owning worker (or fans out to independent ones), gates each handoff on a verified artifact, aggregates, and publishes. Conductor never executes node work.
- **Workers**: each owns exactly one capability and its artifact contract (below). Workers do not route, gate, or publish — they produce artifacts.

## 2. Dispatch matrix (route by task type)

| Task type | Owning worker(s) | Parallel? | Artifact gate |
|---|---|---|---|
| engineering change / new feature | swe-planner | ✗ (single worker) | plan.md + passing checks + commits |
| security audit of a codebase | sec-af | with cloudsecurity-af: YES | security-findings.md |
| IaC / cloud config scan | cloudsecurity-af | with sec-af: YES | cloud-findings.md |
| PR / code review | pr-af | ✗ | review.md (evidence-verified) |
| person intelligence | person-to-brief-af | ✗ | person-brief.md (cited) |
| mixed / ambiguous | Kiro asks a clarifying question first | — | task.md restated |

Independent workers (sec-af ∥ cloudsecurity-af) run concurrently and return separate artifacts; the conductor merges them at the publish gate.

## 3. Handoff protocol (every transition gated)

```
1. INTAKE          Kiro restates task.md (goal, acceptance, scope). Gate: unambiguous.
2. ROUTE           Kiro picks the owning worker from the dispatch matrix.
3. EXECUTE         Worker produces its artifact in the loop's format
                   (see each AGENT.md output contract).
                   Worker self-checks: never report unproven findings.
4. VERIFY          Gate: artifact exists, is in format, every claim traced.
                   pr-af re-verifies review artifacts; conductor checks audits.
5. AGGREGATE       Conductor merges parallel artifacts (audit + scan → one report).
6. PUBLISH         report.md (or PR for swe-planner work) + aftercare notes.
```

**Gate failure rule**: a handoff that fails its gate goes back to the owning worker with the gate report; two failed retries escalate to the conductor, which reduces scope or asks a question. Never silently proceed on a failed gate.

## 4. Agent contracts (in this directory)

| Subagent | AGENT.md | Owns | Produces |
|---|---|---|---|
| swe-planner | `swe-planner/AGENT.md` | build | plan, commits, PR |
| sec-af | `sec-af/AGENT.md` | audit | security-findings.md |
| cloudsecurity-af | `cloudsecurity-af/AGENT.md` | IaC | cloud-findings.md |
| pr-af | `pr-af/AGENT.md` | review | review.md |
| person-to-brief-af | `person-to-brief-af/AGENT.md` | intel | person-brief.md |

All five run natively (Hermes runtime `e54c3957-90e1-48df-9836-db0f2e3a4012`) with the loop spine skills bound (`knowledge-reader`, `handoff`, `planning`, `lean-ctx`, `code-review`; plus `web-research` on the intel worker and `mr-review` on the reviewer).

## 5. Skills

Reuses the shared set (`build-knowledge-base`, `research-codebase`, `knowledge-reader`, `inquire`, `create-roadmap`, `design`, `plan-phase`, `implement`, `supervisor`, workspace-level `lean-ctx`, `handoff`, `planning`, `code-review`). Squad-specific: `security-audit` (sec-af, cloudsecurity-af), `person-brief` (person-to-brief-af) — defined under `skills/`.

## 6. Runtime registration & ground rules

- Autopilot engine: `agentfield-squad` (`c4584fb1-c11c-414a-9498-3c937573db24`), assignee Kiro, run_only; repo `https://github.com/sjoerd2025/agentic-orchestrator/` (main), in-place path `/Volumes/ExternalSSD/projects/Github/agentic-orchestrator`. (Scheduled/webhook triggers are configured platform-side, as for the feature-squad engine.)
- Disk rule: scratch, worktrees, and checkouts under `/Volumes/ExternalSSD` only — boot disk has ~133 MB free; never `/tmp`.
- Workers stop and name the missing credential (GitHub token for private repos, AWS keys for live scans) rather than guess; the conductor reports the blocker verbatim.