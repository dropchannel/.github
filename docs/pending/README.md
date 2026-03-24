# docs/pending/

This directory holds documents that are **decided but not yet actionable** — content that has been worked through and is considered correct, but cannot be committed to its permanent home until a specific condition is met.

"Pending" in this project means waiting on a defined gate, not waiting on further design work. Documents here are not drafts. If a document is still being worked out, it doesn't belong here yet.

---

## Promotion criteria

Each document in this directory should have a clear condition under which it gets promoted to its permanent location. That condition is noted in the document itself or in the table below.

| File | Permanent home | Promotion condition |
|------|---------------|-------------------|
| `ADR-XXXX-versioning-strategy.md` | `docs/adr/ADR-NNNN-versioning-strategy.md` | Org-wide 1.0.0 remediation complete; assign ADR number at commit time |
| `versioning.md` | `docs/conventions/versioning.md` | Org-wide 1.0.0 remediation complete |
| `versioning-remediation-checklist.md` | Close/archive (or convert to GitHub Issue) | All checklist items complete |

---

## What does not belong here

- Documents still under active design — keep those in working notes or a session handoff file until decisions are settled
- Permanent reference material that is already actionable — commit it directly to its home
- Ephemeral working notes — these don't need to be tracked here at all

---

## On promotion

When a document is promoted, remove its row from the table above and delete or archive the file from this directory. The `pending/` directory should trend toward empty over time, not accumulate indefinitely.
