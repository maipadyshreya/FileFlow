# FileFlow architecture

## Architecture status

This is the planned architecture. No Flutter application, Rust core, transport adapter, or FFI binding is implemented yet. See docs/milestones.md for the staged implementation plan.

## Core architecture rule

FileFlow has one multiplatform Flutter application, one reusable Rust core, and transport/platform adapters behind stable interfaces. The CLI is a separate developer and automation interface.

The Rust core must not directly depend on Flutter, desktop APIs, Android, iOS, or browser APIs. Platform-specific behavior belongs behind interfaces or adapters.

This preserves:

- One shared Flutter application codebase targeting desktop, mobile, and web
- A working CLI for testing, automation, and future daemon modes
- Flutter clients using the same Rust core through platform-appropriate bridges
- A narrower web target using the same platform-independent logic
- Transport implementations that can vary by platform without rewriting synchronization or transfer logic

## Chosen direction

FileFlow uses a Flutter client with an embedded Rust core accessed through FFI.

- Flutter owns the UI, navigation, user interaction, progress display, and platform presentation.
- Rust owns file operations, transfer, resumability, synchronization, conflict detection, and security-sensitive domain logic.
- FFI defines the narrow boundary between the Flutter presentation layer and the Rust core.
- The initial deployment model is one desktop application process, with no separate local service.
- The CLI and Flutter shell use the same Rust core.

The recommended binding approach is flutter_rust_bridge or an equivalent generated binding solution. Handwritten FFI remains an alternative if the generated approach does not meet project needs.

## Suggested Rust crate boundaries

These are logical boundaries for implementation and do not require all crates to exist immediately.

- core — platform-independent file, transfer, synchronization, and protocol logic
- transports — IP, MTP, SFTP, rsync, Bluetooth, and future transport adapters
- platform — OS-specific filesystem, permissions, storage, and key-management integrations
- cli — command-line interface and future administrative or daemon modes
- bindings — Flutter/Rust bridge and generated binding support

## Flutter platform targets

### CLI

The CLI is separate from the Flutter application and is the first development and testing interface. It should remain useful for path operations, protocol tests, automation, diagnostics, and possible future daemon modes.

### Desktop Flutter

Desktop is the richest initial client. It can use local filesystem paths, removable devices, mounted paths, local IP, and future platform-specific adapters.

### Mobile Flutter

Mobile reuses the Rust core through platform-appropriate bindings. Storage permissions, background transfer behavior, MTP-like device access, Bluetooth, and secure key storage require mobile platform adapters.

### Web Flutter

The web client is intentionally narrower. It should focus primarily on IP-based FileFlow transfers and browser-compatible workflows. It should not be expected to support MTP, Bluetooth, unrestricted filesystem paths, native key storage, or every desktop transport.

WebAssembly can provide the platform-independent parts of the Rust core, but browser APIs must provide the actual file selection, network, and storage capabilities.

## Implementation layers

### File operation engine

The lowest layer operates on source and destination paths. It handles traversal, metadata, copying, checksums, progress, and filesystem errors. This layer is the focus of the first PoC and can work with local folders, removable storage, mounted network shares, and mounted device paths.

### Transfer and session engine

This layer adds a session abstraction over file operations. It handles metadata exchange, chunking, hashing, pause/resume, interruption recovery, cancellation, and transfer state.

### Transport adapters

The transfer engine is transport-agnostic. Each adapter presents a common session interface for discovery or connection, authentication, metadata exchange, data transfer, progress, and resumability.

### Presentation and command interfaces

The Rust CLI is the first command interface for validating the core. A minimal Flutter desktop shell is added in the same PoC to validate the FFI boundary and provide a working UI surface. Both interfaces call the same Rust core and neither duplicates transfer logic.

## PoC architecture

The first PoC has no network component:

Rust CLI ────────┐
                 ├── Rust file operation engine
Flutter desktop ─┘       └── source path and destination path

The Flutter shell only needs source selection, destination selection, start, progress, and result states. It is intentionally not the final product UI.

## MVP architecture

The desktop MVP will implement a manually addressed local-IP path:

Flutter desktop UI
  -> FFI boundary
  -> Rust transfer/session core
  -> local IP transport
  -> Rust transfer/session core
  -> FFI boundary
  -> Flutter desktop UI

The Rust CLI remains available as a development and troubleshooting tool.

## Device identity and metadata

Each endpoint should have a persistent identity key pair and stable device identifier. A metadata exchange should communicate:

- device identifier
- display name
- identity public key
- protocol version
- software version
- supported transports
- supported capabilities

Device recognition should rely on the stable identity, not on the connection medium. Metadata may travel over IP, a file exposed through MTP, QR pairing, Bluetooth, or another future mechanism.

## Pairing and security

The first connection should require explicit approval. A configuration link or QR code may provide a device identifier, public key, connection details, and optional reachability hints.

IP streams should use modern established encryption such as TLS with authentication tied to the device identity. The project should use established cryptographic libraries and protocols rather than inventing a custom encryption protocol.

## Transport status

| Adapter | Role | Status |
| --- | --- | --- |
| Local filesystem paths | File operation PoC, including mounted paths. | First PoC |
| Local IP | FileFlow-to-FileFlow transfer on a local network. | Desktop MVP and primary web path |
| Remote IP | FileFlow-to-FileFlow transfer over an already reachable IP path. | Future |
| Cable/MTP | Local access to a connected device. | Future desktop/mobile |
| Bluetooth | Short-range transfer where supported by the platform. | Future desktop/mobile |
| QR carrier | Experimental ordered data transfer through rotating QR codes. | Experimental |
| Rsync daemon | Interoperate with an rsync-compatible daemon on the remote endpoint. | Future |
| SFTP | Transfer through an SSH/SFTP server. | Future |

MTP is treated as a local device adapter. It may expose device storage as a path or provider to one FileFlow application; a second FileFlow client is not required on the connected device.

Remote reachability is separate from transport. An overlay such as Tailscale may provide an IP path, but FileFlow should not make overlay networking a requirement of the transfer engine. Tailscale setup, accounts, relays, and reachability management are outside the MVP.

## Future interoperability

The Rust core should support two related but distinct capabilities:

1. FileFlow-native synchronization, using FileFlow session protocols and transport adapters.
2. External endpoint adapters, including rsync daemon and SFTP connections.

An rsync adapter may connect to a daemon running on the other endpoint and use its negotiated protocol for efficient delta transfer. A future FileFlow companion agent could also run an rsync-compatible daemon, allowing other clients to connect to FileFlow-managed files.

SFTP is an interoperability and remote-file-access adapter. It does not provide the same peer discovery, continuous synchronization, or conflict semantics as a FileFlow-native endpoint. Those differences should be visible in the UI and capability model.

QR transport is experimental and should not influence MVP interfaces beyond keeping transport capabilities extensible.

## Open decisions

- Which FFI and binding-generation approach should be used in the implementation?
- Which desktop packaging and build strategy should be used across Linux, macOS, and Windows?
- Which synchronization and conflict-resolution model should be used?
- Which encryption, device-authentication, and key-management details are appropriate?
- Which persistence layer is needed for known devices and transfer history?
- Which local-IP discovery method should be added after manual endpoint entry works?
