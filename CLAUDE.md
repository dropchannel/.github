# CLAUDE.md — dropchannel/.github

This repo is the GitHub org-level community health repository for `github.com/dropchannel`.
It holds the org profile page. It does **not** contain runnable code or tests.

**Links note:** Cross-repo links (to other repos in the DropChannel org) use
<https://github.com/dropchannel/> as the URL root. Intra-repo links use relative URLs.

---

## What lives here

| Path | Purpose |
|------|---------|
| `profile/README.md` | Org profile page (rendered by GitHub at `github.com/dropchannel`) |

---

## Vocabulary

The system uses a **Waterway metaphor**. Use these terms consistently in all docs.

| Term | Meaning | Retired term |
|------|---------|--------------|
| **Channel** | Named path between two fixed endpoints. Namespace for Waterways. | `channel_id` (as param) |
| **Waterway** | A flow path within a Channel. Protocol-typed via name prefix. | slot, reach |
| **Raft** | Forwarding participant. Crypto-blind, passive, no independent agency. | Node |
| **Dock / DockProvider** | A storage backend (GCS, Dropbox, local, httprelay). | ChannelProvider |
| **Upper Dock** | Dock on the initiator's side. Fixed — never swaps. | recv_backend |
| **Lower Dock** | Dock on the responder's side. Fixed — never swaps. | send_backend |
| **Raft ID** | Stable unique identifier for a Raft/host. | node_id |
| **Agent** | Host-level supervisor process. One per physical host. | — |
| **Worker** | Runtime subprocess that runs a Raft. | — |
| **Endpoint** | Terminating participant (Client A/B). Holds the shared secret. | — |
| **Relay** | Network service exposing a Dock over HTTP. | — |

---

## Protocol registry

| Prefix | Protocol | Behavior |
|--------|----------|---------|
| `tideway-` | Tideway | Turn-passing, bidirectional, hold-and-cascade, ACK cascade. Replaces Winch + Piston. |
| `riverway-` | Riverway | Unidirectional, overwrite-always, no ACK. Replaces Conveyer. |
| `telemetry-` | Riverway (functional alias) | External monitoring side-channel. Status TBD — see ADR-0037 open item 1. |
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

## DockProvider interface

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

## Working in this repository

**Keeping CLAUDE.md current:** When updating profile/README.md, update CLAUDE.md to reflect the new state.

The governance/ADR repository was previously named `spec/` and is now named `dropchannel/` — links should use `https://github.com/dropchannel/dropchannel`.

---

## Encryption (reference)

AES-256-GCM. Wire format: `base64( nonce[12 bytes] + ciphertext + tag[16 bytes] )`.
PSK distributed out-of-band. Both directions of a Channel share the same key.
Known gap: no replay protection (no sequence number in AAD).
Heartbeat metadata is **plaintext** — encryption deferred.
