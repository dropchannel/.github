# DropChannel

DropChannel is a multi-protocol coordination runtime for encrypted, asynchronous messaging between two parties over shared storage.

The core idea: two endpoints exchange end-to-end encrypted blobs through a shared storage location — a cloud bucket, a synced folder, an HTTP relay — without trusting that location with anything meaningful. The storage provider sees only opaque ciphertext. Delivery confirmation, backpressure, and crash safety fall out of the protocol itself, with no additional state or mechanism required.

## How it works

A DropChannel pipeline is a directed sequence of storage hops between two endpoints. Each hop is a `ChannelProvider` — an abstraction over any shared location that supports named-blob read, write, and delete. Endpoints encrypt and decrypt. Intermediate nodes forward blobs without ever touching the key.

The **Winch protocol** governs payload propagation: a blob accumulates at every hop during the forward pass, and the terminating endpoint's read triggers a deletion cascade backward through the chain. The originating endpoint's send slot stays occupied until the full cascade completes — structural backpressure with implicit delivery confirmation, no extra messages needed.

Providers currently implemented: GCS, Dropbox (filesystem sync), local filesystem, HTTP relay (for Cloud Run deployment).

## Repositories

| Repo | Description |
|------|-------------|
| [`spec`](https://github.com/dropchannel/spec) | System specification: ChannelProvider interface, protocol registry, dispatch rules, security model |
| [`winch-protocol`](https://github.com/dropchannel/winch-protocol) | Winch protocol spec: hold-and-cascade propagation, ACK cascade, node state machine |
| `ring-protocol` | Ring protocol spec: rolling window, no ACK — for streaming and telemetry use cases (not yet created) |
| [`dropchannel-py`](https://github.com/dropchannel/dropchannel-py) | Python reference implementation: four packages (`dropchannel-channel`, `dropchannel-endpoint`, `dropchannel-node`, `dropchannel-server`) |

## Design principles

**The storage layer is blind.** Providers store and serve opaque bytes. No provider implementation has a dependency on the cryptography library. This is enforced at the package level, not by convention.

**Topology is configuration.** Endpoints have no knowledge of how many hops their pipeline contains. Nodes have no knowledge of the endpoints they serve. Everything is wired through `.env` files.

**Protocols are identified by channel name prefix.** A channel named `winch-commands` uses the Winch protocol; `ring-telemetry` uses Ring. The node reads the prefix and instantiates the appropriate state machine. An unrecognized prefix halts the node — it is never silently ignored.

**Crash safety is inherent.** Any participant can reconstruct its exact state by inspecting its two slots on restart. No external coordination or write-ahead log is required.

## Status

Active development. The Python implementation (`dropchannel-py`) has full test coverage across all four packages. Protocol specs are in progress. Not yet published to PyPI.

## Decisions

Significant architectural decisions are recorded in [`docs/adr/`](../docs/adr/README.md).
