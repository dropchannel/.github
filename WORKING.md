# .github — Working Notes

## What this repo is

The org-level community health repository for `github.com/dropchannel`. GitHub renders
`profile/README.md` as the org profile page. Other files here serve as org-wide defaults
and planning/decision records.

## Current state

The following files have been drafted and are ready to commit:

| File | Status |
|------|--------|
| `profile/README.md` | Draft complete — org profile page |
| `docs/adr/README.md` | Master ADR list (35 ADRs, grouped by spec version) — ready to commit as index |

## What does not exist yet

Individual ADR documents (`ADR-001-*.md` through `ADR-035-*.md`) have not been written.
The master list in `docs/adr/README.md` is the current extent of ADR content.

## ADR triage (not yet done)

The 35 ADRs in the master list have not been triaged into:

- Full ADR (load-bearing architectural decisions where consequences section is the point)
- Short ADR (clear rationale, fits in 2–3 paragraphs)
- Log entry (table row is sufficient)

Suggested candidates for full ADR treatment:

- ADR 008: ChannelProvider abstraction + HTTP relay as one provider
- ADR 010: Delete-as-ACK cascade
- ADR 017: Crypto isolation at package boundary
- ADR 021: Heartbeat on meta slots, separate from primary slots
- ADR 033: Channel name prefix as protocol dispatch mechanism

## Pending tasks (in rough priority order)

1. Triage the 35 ADRs into full / short / log-entry
2. Write individual ADR files for full and short tiers
3. Generate `spec/README.md` (see `spec/WORKING.md`)
4. Write `CONTRIBUTING.md`

## How to start a new session here

For ADR writing, paste this file and `docs/adr/README.md` (the master list). Specify
which ADRs to write and at what depth.

Typical prompt:
> "Read WORKING.md and docs/adr/README.md. Write full ADR documents for ADRs 008, 010,
> and 017. Use the standard ADR format: Title, Status, Context, Decision, Consequences."

For the org profile or other top-level files, paste this file and the relevant draft.
