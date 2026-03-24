# docs/wip/

This directory holds documents in active investigative or research phase — recorded analysis, identified concerns, and assessed options where **no direction has yet been selected**.

"WIP" in this project means the thinking is ongoing. Documents here capture what has been learned and what the problem space looks like, but they do not represent settled decisions. They are working surfaces, not conclusions.

---

## What belongs here

- Security analysis and threat assessments being explored towards eventual remediation
- Candidate evaluations (storage backends, protocols, libraries) where no selection has been made
- Design explorations where tradeoffs are still being worked through
- Research notes that need to be preserved but aren't ready to drive a decision

---

## What does not belong here

- Documents where a direction has been chosen but action is deferred — those belong in `docs/pending/`
- Ephemeral scratch notes not worth preserving — those don't need to be tracked at all
- Finalised decisions — those belong in `docs/adr/`

---

## The distinction from `docs/pending/`

| | `wip/` | `pending/` |
|---|--------|------------|
| **Decision state** | No direction selected | Direction selected, not yet actionable |
| **Content** | Analysis, research, options | Settled documents awaiting a gate |
| **Next step** | More thinking, then a decision | Execute against a known condition |

A document graduates from `wip/` to `pending/` when a direction is chosen. It graduates from `pending/` to its permanent home when its promotion condition is met.

---

## Current documents

| File | Description | Status |
|------|-------------|--------|
| `security-analysis.md` | Assessment of security concerns and attack surface; ongoing exploration towards eventual remediation | Active research |
| `channel-provider-candidates.md` | Evaluation of potential new storage backend candidates; no selection made | Active research |

---

## On graduation

When a document in `wip/` reaches a decision point, move it to `docs/pending/` and add it to that directory's promotion table. Update the table above to reflect the change. Documents should not linger in `wip/` after a direction has been chosen — the distinction between "still thinking" and "decided but waiting" matters for anyone trying to understand the state of the project.
