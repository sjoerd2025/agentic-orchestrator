## Stage 2 Delivery: AgentField Squad — Agent Orchestration Workflow (BETA-27)

In fulfillment of the "new workflow for the squad next to the other" ask: the `@agentfield` squad's orchestration workflow, defined in this repository's Go-file structure (`agents/` + `skills/` embed contract), with the full loop and new subagents sourced from this repo. Existing team agents are unchanged — they remain agents; only the five node subagents are new, and their definitions live here.

---

### 1. New Subagents (from this repo, `agents/agentfield/`)

| Subagent | AGENT.md | Capability (native — no AgentField API) |
|---|---|---|
| swe-planner | `agents/agentfield/swe-planner/AGENT.md` | Plan → implement → verify → PR |
| sec-af | `agents/agentfield/sec-af/AGENT.md` | Security audit: recon → hunt → prove → remediate |
| cloudsecurity-af | `agents/agentfield/cloudsecurity-af/AGENT.md` | IaC static scan + optional live prove (node id `cloudsecurity`) |
| pr-af | `agents/agentfield/pr-af/AGENT.md` | Multi-axis PR review: intake → anatomy → adversary → verify → verdict |
| person-to-brief-af | `agents/agentfield/person-to-brief-af/AGENT.md` | Person intelligence: resolve → research → verify → cited brief |

All five run natively on the Hermes runtime (`e54c3957-90e1-48df-9836-db0f2e3a4012`) in Multica. The embed contract (`//go:embed *` / `all:*`) picks the directories up with zero edits — verified by `go build ./...` (exit 0).

### 2. Orchestration Workflow (`agents/agentfield/WORKFLOW.md`)

Conductor-and-workers orchestration, not a linear pipeline:

- **Conductor: Kiro** (unchanged team agent, squad leader) — classify, dispatch, gate handoffs, publish. Never executes node work.
- **Dispatch matrix** — task type → owning worker; independent workers (sec-af ∥ cloudsecurity-af) fan out in parallel.
- **Handoff protocol** — every transition gated on a verified artifact in format; gate-failure rule (retry twice, then escalate); no silent progression.
- **Full loop stages** — INTAKE → ROUTE → EXECUTE → VERIFY → AGGREGATE → PUBLISH → aftercare.

### 3. New Skills (`skills/`)

- `security-audit` (SKILL.md + format.md) — findings schema used by sec-af and cloudsecurity-af (severity-ranked, file:line:fix, investigated-not-proven list).
- `person-brief` (SKILL.md + format.md) — structured cited-brief schema (summary, priorities, recent signals, conversation starters, messaging angles, cautions).

### 4. Runtime Registration (Multica)

- Autopilot engine **`agentfield-squad`** `c4584fb1-c11c-414a-9498-3c937573db24` — assignee Kiro, run_only, bound to `https://github.com/sjoerd2025/agentic-orchestrator/` (main) + in-place path. (Scheduled/webhook triggers are platform-side, as with the feature-squad engine `35b2904d…`.)
- Loop spine skills bound to all five workers: `knowledge-reader`, `handoff`, `planning`, `lean-ctx`, `code-review`; plus `web-research` (person-to-brief-af) and `mr-review` (pr-af).
- Earlier live smoke (swe-planner, task `01a194fe`) returned `{"output": "integration ok"}` — claim + native run verified.

### 5. Ground Rules

- Disk: all scratch/worktrees/checkouts under `/Volumes/ExternalSSD` (boot disk ~133 MB free; never `/tmp`).
- Workers stop and name missing credentials (GitHub token for private repos, AWS keys for live scans) rather than guess.

### 6. Status

Delivered and verified (build green, registration live, smoke passed). Remaining: platform-side trigger scheduling for the `agentfield-squad` engine, and (optional) importing the two new skills into Multica or leaving them repo-side as the ground source. This issue awaits human approval in line with the project workflow.

### 7. Side note — TO REVISIT

This stage commits as a foundation, not a finished loop. Revisit items, in order of importance on the next pass:

1. Trigger scheduling for engine `c4584fb1` (scheduled + webhook) — platform-side, not configured yet; the engine is run_only until then.
2. First real multi-agent run — the orchestration (dispatch, parallel fan-out, gate escalation) is defined on paper; tune gates after one production run.
3. Import `security-audit` + `person-brief` skills into Multica, or deliberately keep them repo-side as ground source — decide and record.
4. Disk pressure — boot disk ~133 MB free; revisit if the ExternalSSD-only ground rule gets violated by any runtime.
5. Worker credentials (GitHub token, AWS keys) — provisioning path for `pr-af`/`cloudsecurity-af` when private repos / live scans are actually requested.