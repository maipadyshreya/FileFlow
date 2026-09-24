# FileFlow architecture

## Chosen direction

FileFlow uses a Flutter client with an embedded Rust core accessed through FFI.

- Flutter owns the UI, navigation, user interaction, progress display, and platform presentation.
- Rust owns transfer, resumability, synchronization, conflict detection, and security-sensitive domain logic.
- FFI defines the narrow boundary between the Flutter presentation layer and the Rust core.
- The initial deployment model is one desktop application process, with no separate local service.

This keeps the first desktop release straightforward while allowing the Rust core to be reused for mobile and adapted to WebAssembly for web support later.

## Repository boundaries

- apps/desktop/ — Flutter UI and desktop application integration
- crates/ — Rust core libraries
- bindings/ — generated or handwritten Flutter/Rust FFI boundary, when introduced
- docs/ — requirements and technical design notes

## User and data flow

1. The user selects a file or folder to share.
2. FileFlow discovers a new device or connects to a known device over an available medium.
3. The Flutter layer requests operations from the Rust core through FFI.
4. The Rust core establishes the transfer, reports progress, and supports resumability.
5. Simultaneous edits produce a visible conflict instead of silently overwriting data.
6. The Flutter layer presents transfer state, conflicts, search, and filtering.

## Current technology direction

- Client framework: Flutter
- Core language: Rust
- Desktop integration: Flutter desktop plugins and native platform adapters as needed
- Mobile integration: platform-appropriate Flutter/Rust bindings
- Web direction: WebAssembly where browser capabilities permit
- Transfer security: end-to-end encryption; the concrete protocol and library remain open decisions

## Open decisions

- Which desktop platform should receive the first development build: Windows, macOS, or Linux?
- Should the first transfer path be local network, cable, or both?
- Which FFI and binding-generation approach should be used?
- Which synchronization and conflict-resolution model should be used?
- Which encryption, device-authentication, and key-management approach is appropriate?
- Which persistence layer is needed for known devices and transfer history?
