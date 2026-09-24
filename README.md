# FileFlow

FileFlow is a cross-platform file-transfer and folder-synchronization application.

## Project direction

The implementation priority is:

1. Desktop
2. Mobile
3. Web

The initial architecture is a Flutter application with an embedded Rust core accessed through FFI. Rust owns transfer, synchronization, conflict handling, and security-sensitive logic. Flutter owns the user interface and platform presentation.

## Project documents

- [Requirements](docs/requirements.md) — product scope, requirements, and user stories
- [Architecture](docs/architecture.md) — system boundaries, data flow, and open decisions
- [Agent guidance](AGENTS.md) — repository conventions for coding agents
- [Desktop app scaffold](apps/desktop/README.md)
- [Rust core scaffold](crates/README.md)

## Current status

The repository is in the definition and architecture stage. The directory layout establishes the intended ownership boundaries, but application code and bindings have not been implemented yet.

## Development

Setup and test commands will be added with the first Flutter and Rust scaffolds.
