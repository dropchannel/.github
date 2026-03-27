# CLAUDE.md — dropchannel/.github

This repo is the GitHub org-level community health repository for `github.com/dropchannel`.
It holds the org profile page, architectural decision records, and planning documents.
It does **not** contain runnable code or tests.

**Naming note:** Numerous aspects of the DropChannel system and protocols have been
renamed as part of ADR-037. All ADR-related content specific to versions 000-036
use old names — do not update them.

---

## What lives here

| Path | Purpose |
|------|---------|
| `profile/README.md` | Org profile page (rendered by GitHub at `github.com/dropchannel`) |
| `docs/adr/README.md` | Master ADR index — all 37 ADRs, grouped by spec version |
| `docs/adr/ADR-NNN.md` | Individual ADR documents (only a few written so far) |
| `docs/channel-provider-candidates.md` | Candidate storage backends under consideration |

---

## Vocabulary (ADR-037)

The system uses a **Waterway metaphor**. Use these terms consistently in all docs and ADRs.

| Term | Meaning |
|------|---------|
| **Channel** | Named path between two fixed endpoints. Namespace for Waterways. |
| **Waterway** | A flow path within a Channel. Protocol-typed via name prefix. |
| **Raft** | Forwarding participant. Crypto-blind, passive, no independent agency. (Old: Node) |
| **Dock / DockProvider** | A storage backend (GCS, Dropbox, local, httprelay). (Old: ChannelProvider) |
| **Upper Dock** | Dock on the initiator's side. Fixed — never swaps. (Old: recv_backend) |
| **Lower Dock** | Dock on the responder's side. Fixed — never swaps. (Old: send_backend) |
| **Raft ID** | Stable unique identifier for a Raft/host. (Old: node_id) |
| **Agent** | Host-level supervisor process. One per physical host. Unchanged. |
| **Worker** | Runtime subprocess that runs a Raft. Unchanged. |
| **Endpoint** | Terminating participant (Client A/B). Holds the shared secret. Unchanged. |
| **Relay** | Network service exposing a Dock over HTTP. Unchanged. |

**Do not use** old terms: slot, node (as forwarding participant), channel_id (as param name),
recv_backend, send_backend, node_id, winch, conveyer, ring, piston.

---

## Protocol registry

| Prefix | Protocol | Behavior |
|--------|----------|---------|
| `tideway-` | Tideway | Turn-passing, bidirectional, hold-and-cascade, ACK cascade. Replaces Winch + Piston. |
| `riverway-` | Riverway | Unidirectional, overwrite-always, no ACK. Replaces Conveyer. |
| `telemetry-` | Riverway (functional alias) | External monitoring side-channel. Status TBD — see ADR-037 open item 1. |
| `heartbeat-` | Meta Waterway | Per-hop liveness chain on meta Waterways. Plaintext. |

Unrecognized prefix → halt and log. Never silently skip.

---

## Key design invariants

- **Storage layer is blind.** Docks store opaque bytes. No Dock implementation may depend on the `cryptography` package.
- **Topology is configuration.** Endpoints don't know hop count. Rafts don't know the endpoints.
- **Crash safety is inherent.** Any participant reconstructs state from its two Docks on restart. No WAL needed.
- **Unrecognized prefix halts.** Never a default protocol, never a silent skip.
- **Reach/Waterway names are invariant.** The same name at every Dock across the pipeline.
- **Workers receive only their two Dock configs** at spawn via stdin — never the full backend map or credentials file.

---

## DockProvider interface (ADR-037)

```python
# Primary operations
write(channel, waterway, filename, data) → bool      # False if filename already present
read(channel, waterway, filename) → bytes|None        # Destructive. Terminating endpoint only.
peek(channel, waterway, filename) → bytes|None        # Non-destructive.
exists(channel, waterway, filename) → bool            # Primary polling operation.
delete(channel, waterway, filename) → None            # Idempotent.

# Meta operations (heartbeat/observability only)
meta_write(channel, waterway, filename, data) → bool
meta_read(channel, waterway, filename) → bytes|None
meta_delete(channel, waterway, filename) → None
meta_list(channel, waterway) → list[str]
```

---

## ADR format

Standard sections: **Title, Status, Date, Deciders, Context, Decision, Consequences.**

The three tiers:

- **Full ADR** — load-bearing decisions where the Consequences section is the point. Write as individual `ADR-NNN.md`.
- **Short ADR** — clear rationale, fits in 2–3 paragraphs. Individual file.
- **Log entry** — table row in `docs/adr/README.md` is sufficient.

Priority candidates for full ADR treatment: 008, 010, 017, 021, 033.

Individual ADR files (`ADR-001.md` through `ADR-035.md`) have not been written yet —
`docs/adr/README.md` (the master index) is the current extent of ADR content.
ADR-036 and ADR-037 have individual files.

The 35 earlier ADRs have not been triaged into full / short / log-entry tiers.

**Starting a new ADR-writing session:** provide `docs/adr/README.md`,
then specify which ADRs to write and at what depth. Example:
> "Read CLAUDE.md and docs/adr/README.md. Write full ADR documents for ADRs 008, 010,
> and 017. Use the standard ADR format: Title, Status, Context, Decision, Consequences."

---

## Current branch

`waterway-naming-changes` — implementing ADR-037 vocabulary rename across this repo.
Recent commits renamed Tide→Tideway, Current→Riverway, Run/head/tail→Waterway/Upper/Lower,
removed ring/piston references, renamed Winch→Tide (now Tideway).

---

## Encryption (reference)

AES-256-GCM. Wire format: `base64( nonce[12 bytes] + ciphertext + tag[16 bytes] )`.
PSK distributed out-of-band. Both directions of a Channel share the same key.
Known gap: no replay protection (no sequence number in AAD).
Heartbeat metadata is **plaintext** — encryption deferred.
