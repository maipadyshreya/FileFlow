# FileFlow requirements

## Product overview

FileFlow is a cross-platform file-transfer and folder-synchronization application for people who work with large numbers of files, including non-technical users.

The product should support batch file transfer and folder synchronization across:

- IP/network transfer
- Direct file access over cable
- Bluetooth as a stretch goal

## Delivery status

The repository contains planning documents and scaffold notes only. No transfer or synchronization functionality is implemented yet.

See docs/milestones.md for the complete staged roadmap.

### First PoC

The first implementation combines a Rust CLI, a minimal Flutter desktop shell, and a shared Rust path-based file operation engine. It copies files or directories between source and destination paths, reports progress, verifies checksums, and returns clear errors. It has no network component.

### Desktop MVP

The desktop MVP adds local-IP transfer through the Flutter UI and embedded Rust core accessed through FFI. It targets manually addressed local-IP transfer between FileFlow endpoints, with peer approval, progress, pause/resume, interrupted-transfer recovery, and basic authentication/encryption.

### Later goals

Continuous synchronization, conflict resolution, MTP, remote IP paths, remembered devices, search/filtering, mobile, SFTP, rsync interoperability, and Bluetooth are later goals unless explicitly promoted into the MVP plan.

### Experimental or out of scope for the MVP

QR-code carrier transport, Tailscale management, full rsync server compatibility, web support, and broad platform certification are not part of the initial MVP.

## Platform priority

1. Desktop
2. Mobile
3. Web

The team is cross-platform. Development and automated checks should support Linux, macOS, and Windows rather than making one operating system the architecture baseline.

## Functional requirements

| ID | Requirement | Priority | Delivery status |
| --- | --- | --- | --- |
| FR-01 | Transfer a file to a previously unknown peer device. | Must | Desktop MVP |
| FR-02 | Continue synchronizing a folder as changes occur on either device. | Must | Later |
| FR-03 | Resume a failed file transfer without starting over. | Must | Desktop MVP |
| FR-04 | Support network transfer, direct cable access, and Bluetooth. | Must / Stretch for Bluetooth | Local IP in MVP; others later |
| FR-05 | Allow users to pause and resume a transfer. | Must | Desktop MVP |
| FR-06 | Allow a transfer to continue over a different connection method. | Should | Later |
| FR-07 | Support common file types without requiring format-specific handling. | Must | First PoC and Desktop MVP |
| FR-08 | Remember devices that have previously been synced with or used for transfers. | Should | Later |
| FR-09 | Detect simultaneous edits and prevent one change from silently overwriting another. | Must | Later |
| FR-10 | Search for files by name and filter them by type. | Should | Later |

## Nonfunctional requirements

| ID | Requirement | Delivery status |
| --- | --- | --- |
| NFR-01 | Run on Windows, macOS/iOS, Android, and Linux, subject to platform capabilities. | Later; desktop development across Linux, macOS, and Windows first |
| NFR-02 | Make basic functions usable by non-technical users. | Desktop MVP |
| NFR-03 | Provide responsive feedback, including transfer progress indicators. | First PoC and Desktop MVP |
| NFR-04 | Start a requested process within 0.5 seconds under normal operating conditions. | Desktop MVP target; validate during implementation |

## User stories

These stories break the product goals into implementation-sized slices. The IDs can be used in issues, tests, and design notes.

| ID | User story | Related requirements | Delivery status |
| --- | --- | --- | --- |
| US-01 | As a sender, I want to select one or more files or a folder so that I can prepare a transfer. | FR-01, FR-02, FR-07 | First PoC for paths; Desktop MVP for UI |
| US-02 | As a sender, I want to discover a nearby new device and verify its identity so that I can transfer files to the intended recipient. | FR-01, FR-04 | Later; manual IP entry in Desktop MVP |
| US-03 | As a recipient, I want to see an incoming transfer request and accept or reject it so that I control what enters my device. | FR-01, FR-04, NFR-02 | Desktop MVP |
| US-04 | As a sender or recipient, I want to see transfer progress, estimated status, and errors so that I understand what the application is doing. | NFR-02, NFR-03 | First PoC and Desktop MVP |
| US-05 | As a sender or recipient, I want to pause and resume a transfer so that I can manage bandwidth and device resources. | FR-05, NFR-03 | Desktop MVP |
| US-06 | As a sender or recipient, I want a failed transfer to resume from the last completed portion so that I do not waste time retransmitting data. | FR-03, NFR-03 | Desktop MVP |
| US-07 | As a sender or recipient, I want to continue a transfer over another available connection method so that a changing connection does not force me to start over. | FR-04, FR-06 | Later |
| US-08 | As a user, I want to synchronize a selected folder continuously so that changes made on either device are reflected on the other device. | FR-02 | Later |
| US-09 | As a user, I want simultaneous edits to create a visible conflict with clear choices so that my changes are not silently lost. | FR-09, NFR-02 | Later |
| US-10 | As a user, I want previously used devices to be remembered and recognizable so that repeat transfers require less setup. | FR-08, NFR-02 | Later |
| US-11 | As a user, I want to search by filename and filter by file type so that I can find files quickly. | FR-10, NFR-02 | Later |
| US-12 | As a user, I want transfers to be protected in transit and devices to be authenticated so that private files are not exposed to unintended peers. | NFR-02 | Desktop MVP |

## Initial acceptance outcomes

These outcomes are intentionally platform-neutral. Detailed acceptance criteria should be added to implementation issues as each story is scheduled.

| Story | Initial outcome |
| --- | --- |
| US-01 | A user can select a single file, multiple files, and a folder, and can review the selection before starting. |
| US-02 | A new peer is presented with enough identifying information for the user to confirm the intended device before transfer. |
| US-03 | A recipient can accept or reject a request, and rejected files are not written to the destination. |
| US-04 | Every active transfer exposes state such as queued, transferring, paused, completed, failed, or cancelled. |
| US-05 | Pausing prevents additional transfer work, and resuming continues the same transfer. |
| US-06 | A retry does not retransmit verified completed chunks. |
| US-07 | A connection change preserves transfer identity and already verified progress. |
| US-08 | A folder change is detected and synchronized according to the selected synchronization policy. |
| US-09 | Conflicting changes remain recoverable and the application never silently discards either version. |
| US-10 | A returning device can be identified without requiring the user to repeat the entire setup flow. |
| US-11 | Search and type filters update the visible file results without modifying files. |
| US-12 | Transfers use the selected security protocol, and an untrusted peer cannot access transfer contents. |

The requirement IDs above are canonical for design notes, issues, tests, and implementation tasks.


## PoC operation semantics

The first PoC supports both copy and move modes. Destination handling is configurable and follows rsync-inspired principles:

- skip files that are already equivalent
- compare metadata before checksums where possible
- use checksums when configured or needed
- update or overwrite according to the selected setting
- handle newer destination files according to the selected setting
- report conflicts instead of making hidden destructive choices
- remove the source only after a move has completed and the destination has been verified

The CLI and Flutter shell use the same Rust operation API. Rust owns comparison, filesystem work, checksums, state, and errors. Flutter owns controls, path selection, progress display, status messages, and layout.


## Settled PoC defaults

- The initial Flutter/Rust bridge will use flutter_rust_bridge or its current successor.
- The PoC supports regular files, directories, and hidden files.
- The PoC preserves basic timestamps.
- Symbolic-link behavior and complete permission preservation are deferred.
- Copy is the default operation mode.
- Newer destination files are not overwritten by default.
- Files with matching size and modification time are skipped.
- Optional checksum comparison provides stronger verification.
- Overwriting requires an explicit setting.
- Move removes the source only after successful destination verification.
