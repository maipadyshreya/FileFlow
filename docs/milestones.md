# FileFlow milestones

This roadmap separates what is implemented, what is being built next, and what remains a future goal.

## Milestone 0 — Path-based file operation PoC

**Status:** Planned first implementation

**Purpose:** Validate the Rust file operation engine, the Flutter/Rust boundary, and the multi-front-end structure without networking or transport protocol complexity.

### Rust core and CLI

Example command:

    fileflow copy SOURCE_PATH DESTINATION_PATH

Scope:

- Copy one file
- Copy a directory recursively
- Preserve names and directory structure
- Report progress
- Verify copied data with checksums
- Return clear errors
- Operate on any path exposed by the operating system
- Provide a CLI harness for repeatable testing and automation

### Basic Flutter desktop shell

The Flutter PoC should provide a small but real user flow:

1. Select a source file or directory.
2. Select a destination path.
3. Start the copy operation.
4. Display progress and current status.
5. Display checksum verification and success or error results.

The UI should call the Rust path operation through the planned FFI boundary. It may be visually simple; the purpose is to validate the integration and give the UI team a working surface for layout and interaction design.

The CLI and Flutter shell must call the same Rust core. Neither should contain a separate copy implementation.

### Client boundary goals

- Keep the Rust core independent of Flutter and operating-system UI APIs.
- Keep the CLI working as a first-class interface.
- Keep platform-specific filesystem and permission logic behind adapters.
- Use the same domain and file-operation APIs for desktop, mobile, and future web integration where possible.

Out of scope:

- Network communication
- Device discovery
- Peer approval
- Encryption
- Continuous synchronization
- Conflict resolution
- Transport switching
- Production-quality visual design

Deliverables:

- Rust core library for path-based file operations
- Rust CLI that exercises the core
- Minimal Flutter desktop application
- Flutter/Rust binding proof of concept
- Unit tests for copying, checksums, errors, and progress
- A basic Flutter UI test for operation states

## Milestone 1 — Local-IP CLI transfer PoC

**Status:** Planned after Milestone 0

**Purpose:** Prove the first FileFlow-native transport while keeping the interface simple.

Example commands:

    fileflow receive --listen 0.0.0.0:PORT
    fileflow send --to 192.168.1.20:PORT ./example.txt

Scope:

- Manually supplied IP address and port
- One file transfer
- Basic metadata exchange
- Transfer checksum verification
- Clear success and failure reporting
- Two CLI processes using the shared Rust core

Initially excluded:

- Automatic discovery
- Folder synchronization
- Multiple simultaneous transfers
- Transport switching
- Mobile and web support

## Milestone 2 — Secure local-IP transfer

**Status:** Planned

**Purpose:** Add the minimum trust and security model before building a user-facing desktop application.

Scope:

- Persistent device identity key pairs
- Initial key exchange
- Explicit first-connection approval
- Standard encrypted IP transport, expected to use modern TLS or an equivalent established protocol
- Device identity and protocol metadata
- Configuration link or QR pairing as an optional out-of-band setup method

The project should use established cryptographic libraries and protocols rather than inventing encryption.

## Milestone 3 — Desktop MVP

**Status:** Target product milestone

**Platforms:** Linux, macOS, and Windows development support

Scope:

- Flutter desktop application
- Embedded Rust core accessed through flutter_rust_bridge or an equivalent generated binding approach
- File and folder selection
- Manual local-IP endpoint entry, with discovery deferred
- Peer approval
- Progress display
- Pause and resume
- Interrupted-transfer recovery using verified progress
- Basic authentication and encryption
- Common file types

The first desktop release should use the same Rust core proven by the CLI and the PoC Flutter shell. The Flutter application should be a presentation layer over that core, not a replacement implementation.

## Milestone 4 — Transfer usability and local-device support

**Status:** Future

Scope:

- Automatic local-IP discovery
- Remembered devices
- Search and file-type filtering
- Better transfer history and error recovery
- Direct cable and MTP support
- Path/device capability reporting

MTP is treated as a local device adapter. A single FileFlow application may read from or write to a connected device; a second FileFlow client is not required on the MTP device.

## Milestone 5 — Synchronization

**Status:** Future

Scope:

- Continuous folder synchronization
- Change detection
- Synchronization policies
- Conflict detection
- Conflict recovery that never silently discards either version
- Resumable synchronization sessions

This milestone is separate from one-time file or folder transfer.

## Milestone 6 — External transport interoperability

**Status:** Future

Scope:

- SFTP adapter for existing SSH/SFTP servers
- Rsync adapter for connecting to an rsync-compatible daemon
- Optional FileFlow companion agent that can expose an rsync-compatible daemon
- Capability-aware UI for transports that do not support native peer discovery or continuous synchronization

The Rust transfer engine remains shared, but external protocols must retain their own protocol and capability limitations.

## Milestone 7 — Additional connection methods

**Status:** Future or experimental

Potential scope:

- Remote IP over an already reachable path
- Bluetooth transport
- Transport switching while preserving verified progress
- Tailscale as an optional reachability provider

Tailscale accounts, relays, and reachability management are outside the core project scope.

Rotating QR-code data transfer is experimental. QR pairing and configuration exchange may be useful earlier, but QR as the actual data carrier should not constrain the PoC or MVP interfaces.

## Milestone 8 — Mobile and web

**Status:** Future

Mobile:

- Reuse the Rust core through platform-appropriate bindings
- Add platform-specific storage and permission adapters
- Preserve the same capability and identity model

Web:

- Reuse platform-independent Rust logic through WebAssembly where browser capabilities permit
- Focus primarily on IP-based FileFlow transfers
- Define browser-specific file selection, network, and storage limitations
- Do not assume desktop FFI or filesystem behavior maps directly to browsers
- Do not require MTP, Bluetooth, unrestricted filesystem paths, or every desktop transport

## Cross-milestone testing

- Rust unit tests for path operations, chunking, hashing, resume logic, and conflict rules
- CLI integration tests between two local processes
- Transport tests using temporary directories and controlled interruptions
- Flutter UI tests for transfer states and errors
- Desktop smoke tests on Linux, macOS, and Windows
- Corrupted-data, interrupted-transfer, permission, and insufficient-space tests


## Approved PoC operation decisions

The path-based PoC supports both copy and move operations. Its operation API accepts source, destination, mode, and configurable settings.

Destination behavior is rsync-inspired:

- equivalent files can be skipped
- metadata is compared before checksums where possible
- update and overwrite behavior is configurable
- newer destination files are handled according to settings
- conflicts are reported rather than hidden
- move removes the source only after successful destination verification

The CLI calls the Rust API directly. Flutter calls the same API through the generated bridge. Rust owns operation state and filesystem behavior; Flutter owns controls and presentation.


## Settled PoC defaults

- Use flutter_rust_bridge or its current successor for generated Flutter/Rust bindings.
- Support regular files, directories, and hidden files.
- Preserve basic timestamps.
- Defer symbolic-link policy and complete permission preservation.
- Default to copy mode.
- Do not overwrite newer destination files by default.
- Skip files with matching size and modification time.
- Allow optional checksum comparison.
- Require an explicit overwrite setting.
- Remove a source only after a move destination has been successfully written and verified.
