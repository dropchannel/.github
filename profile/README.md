# DropChannel

DropChannel is a multi-protocol coordination runtime for encrypted, asynchronous messaging between two parties over shared storage.

The core idea: two endpoints exchange end-to-end encrypted blobs through a shared storage location — a cloud bucket, a synced folder, an HTTP relay — without trusting that location with anything meaningful. The storage provider sees only opaque ciphertext. Delivery confirmation, backpressure, and crash safety fall out of the protocol itself, with no additional state or mechanism required.

## How it works

A DropChannel pipeline is a directed sequence of storage hops between two endpoints. Each hop is a `DockProvider` — an abstraction over any shared location that supports named-blob read, write, and delete. Endpoints encrypt and decrypt. Intermediate Rafts forward blobs without ever touching the key.

A **Channel** is the named path between two fixed endpoints. Within a Channel, one or more **Waterways** carry payload flows — each Waterway is protocol-typed via its name prefix.

The **Tideway protocol** (`tideway-`) governs turn-passing payload propagation: a blob accumulates at every hop during the forward pass, and the terminating endpoint's read triggers a deletion cascade backward through the chain. The originating endpoint's Waterway stays occupied until the full cascade completes — structural backpressure with implicit delivery confirmation, no extra messages needed.

The **Riverway protocol** (`riverway-`) is unidirectional and overwrite-always: the producer continuously pushes the latest value Upper → Lower with no ACK and no backpressure. Used for telemetry feeds, sensor data, and any producer-driven flow where only the latest value matters.

Each hop is managed at runtime by a **Worker** subprocess. Workers are coordinated per host by an **Agent**. Workers spawn and run a **Raft** — the passive forwarding mechanism that carries payload files between Docks. Rafts are crypto-blind; they never hold the key or read the content.

Providers currently implemented: GCS, Dropbox (filesystem sync), local filesystem, HTTP relay (for Cloud Run deployment).

## Repositories

| Repo | Description |
|------|-------------|
| [`spec`](https://github.com/dropchannel/spec) | System specification: DockProvider interface, protocol registry, dispatch rules, security model |
| [`tideway-protocol`](https://github.com/dropchannel/tideway-protocol) | Tideway protocol spec: hold-and-cascade propagation, turn-taking w/ ACK cascade |
| [`riverway-protocol`](https://github.com/dropchannel/riverway-protocol) | Riverway protocol spec: unidirectional, overwrite-always, no ACK |
| [`dropchannel-py`](https://github.com/dropchannel/dropchannel-py) | Python reference implementation: four packages (`dropchannel-channel`, `dropchannel-endpoint`, `dropchannel-raft`, `dropchannel-server`) |

## Design principles

**The storage layer is blind.** Providers store and serve opaque bytes. No provider implementation has a dependency on the cryptography library. This is enforced at the package level, not by convention.

**Topology is configuration.** Endpoints have no knowledge of how many hops their pipeline contains. Rafts have no knowledge of the endpoints they serve. Everything is wired through `.env` files.

**Protocols are identified by Waterway name prefix.** A Waterway named `tideway-commands` uses the Tideway protocol. The Raft reads the prefix and instantiates the appropriate state machine. An unrecognized prefix halts the Raft — it is never silently ignored.

**Crash safety is inherent.** Any participant can reconstruct its exact state by inspecting its two Docks on restart. No external coordination or write-ahead log is required.

## Status

Active development. The Python implementation (`dropchannel-py`) has full test coverage across all four packages. Protocol specs are in progress. Not yet published to PyPI.

## Decisions

Significant architectural decisions are recorded in [`docs/adr/`](../docs/adr/README.md).
