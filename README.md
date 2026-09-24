# FileFlow

FileFlow is a cross-platform file-transfer and folder-synchronization application.

## Project direction

The implementation priority is:

1. Desktop
2. Mobile
3. Web

The initial architecture is a Flutter application with an embedded Rust core accessed through FFI. Rust owns transfer, synchronization, conflict handling, and security-sensitive logic. Flutter owns the user interface and platform presentation.

## Architecture rule

FileFlow has one reusable Rust core, multiple front ends, and transport/platform adapters behind stable interfaces.

The Rust core must not depend directly on Flutter, desktop APIs, Android, iOS, or browser APIs. The CLI, Flutter desktop/mobile clients, and future web client should all use the same core where their platform capabilities allow.

## Delivery status

### Working now

- Project documentation and requirement tracking
- Flutter/Rust repository boundary documentation
- Initial desktop and Rust core directory placeholders

No file transfer, synchronization, device discovery, or FFI implementation works yet.

### First PoC target

The first PoC will combine:

- A Rust CLI for fast testing and automation
- A basic Flutter desktop shell for the UI team
- A shared Rust path-based file operation engine
- Local source and destination paths only
- Progress, checksum verification, and clear success or error states

The PoC intentionally has no network, discovery, encryption, synchronization, or transport adapters.

### MVP target

The first desktop MVP should support local-IP file transfer through a Flutter desktop application backed by the shared Rust core. It will include file/folder selection, manual endpoint entry, peer approval, progress, pause/resume, interrupted-transfer recovery, and basic security.

## Client capability profiles

- **CLI:** testing, automation, local path operations, and future daemon or administrative modes.
- **Desktop:** richest initial client, including local paths, removable devices, local IP, and future platform-specific transports.
- **Mobile:** shared Rust core with platform-specific storage, permissions, and device adapters.
- **Web:** intentionally narrower client focused primarily on IP-based transfers and browser-compatible workflows. It should not be expected to support MTP, Bluetooth, native device storage access, or every desktop transport.

## Project documents

- [Requirements](docs/requirements.md) — product scope, requirements, user stories, and delivery status
- [Architecture](docs/architecture.md) — system boundaries, transport model, and open decisions
- [Milestones](docs/milestones.md) — staged implementation roadmap
- [Agent guidance](AGENTS.md) — repository conventions for coding agents
- [Desktop app scaffold](apps/desktop/README.md)
- [Rust core scaffold](crates/README.md)

## Current status

The repository is in the planning and architecture stage. The directory layout establishes intended ownership boundaries, but application code and bindings have not been implemented yet.

## Development

Setup and test commands will be added with the first Flutter and Rust scaffolds.
