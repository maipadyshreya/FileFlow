# Agent guidance for FileFlow

## Project context

FileFlow is a Flutter application with an embedded Rust core accessed through FFI. The delivery priority is desktop, then mobile, then web. Read docs/requirements.md and docs/architecture.md before proposing implementation work.

## Repository boundaries

- apps/desktop/ — Flutter UI and desktop application integration.
- crates/ — Rust core libraries for transfer, synchronization, conflict handling, and security.
- bindings/ — generated or handwritten Flutter/Rust FFI boundary, when introduced.
- docs/ — requirements and technical design notes.

Do not introduce a separate local service unless the team explicitly revises the architecture. Keep the Rust core independent from Flutter so it can later support mobile bindings and WebAssembly where platform capabilities allow.

## Working rules

- Keep changes small and explain the reason for each architectural choice.
- Treat FR-01 through FR-10 and NFR-01 through NFR-04 as canonical requirement IDs.
- Prefer a thin vertical slice over broad placeholder code once implementation begins.
- Preserve resumable transfers, conflict safety, clear progress feedback, and end-to-end security as first-class concerns.
- Do not claim support for a platform or transfer medium until it is implemented and tested.
- Update README.md or the relevant document in docs/ when setup steps, decisions, or supported scope changes.

## Before changing code

1. Inspect the current repository and existing documentation.
2. Identify the relevant requirement IDs.
3. State assumptions when an open decision affects the implementation.
4. Look for existing tests and run the narrowest relevant checks before and after the change.
5. Keep platform-specific code at the application boundary where possible.

## Documentation and design

Use docs/requirements.md for product requirements and docs/architecture.md for technical decisions. Record settled architecture decisions with their rationale, alternatives considered, and affected platforms. Keep unresolved questions in the architecture document open-decisions section rather than silently choosing for the team.

## Git and file hygiene

- Avoid destructive Git commands and do not overwrite unrelated work.
- Do not commit secrets, credentials, generated build output, or large transfer fixtures.
- Add project-specific ignore rules when the first toolchain is selected.
- Keep filenames and links stable unless a rename is necessary.
