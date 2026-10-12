# ProtoLattice Design Document
*A Modern, Rust-Powered Engine for Multiplayer Tile & Voxel Worlds*

---

## 1. Project Overview

**ProtoLattice** is a modern, performance-focused engine designed to spiritualize and advance the core paradigm of BYOND (*Build Your Own Net Dream*). It provides a unified, networked framework for building multiplayer, tile- and voxel-based virtual worlds, games, and persistent environments.

ProtoLattice leverages Rust across the entire technology stack to guarantee memory safety, high performance, and concurrency. Server-side game logic and assets are bundled together into a unified, redistributable archive (`.plc`) that the server loads and dynamically streams to connecting clients, allowing friction-free user onboarding without manual asset installation.

---

## 2. Core Goals & Vision

* **Zero-Friction Multiplayer Access:** Clients connect to a server IP/port and automatically download/stream all necessary assets and dynamic game state required to render and play.
* **Unified Rust & Bevy Ecosystem:** Utilize the Bevy Entity Component System (ECS) for both client rendering and server-side world simulation.
* **WASM-Driven Game Code Packaging:** Sandboxed execution of server-side game logic using WebAssembly (WASM), decoupling host system operations from community-provided game code.
* **Flexible Dimension Support:** Native, built-in support for both 2D tilemaps and 3D voxel-based grid worlds.
* **Moddable & Package-Driven Distribution:** Entire games are shipped as single `.plc` packages containing compiled WASM binaries and all required media assets.

---

## 3. System Architecture & Technical Research

### 3.1 Stack Breakdown

| Component | Technology | Role |
| :--- | :--- | :--- |
| **Language** | Rust | Core system implementation (Client, Server Host, Tooling). |
| **ECS Framework** | Bevy Engine | Entity and world management on client and server. |
| **Server Logic Sandbox** | WASM (`wasm32-unknown-unknown`) | Encapsulates game simulation logic within host server. |
| **WASM Runtime** | `wasmtime` / `wasmer` | Embeddable host server execution engine for `.wasm` modules. |
| **Package Format** | `.plc` (Zip container) | Redistributable container holding `.wasm` binaries + `assets/`. |
| **Networking** | QUIC / UDP (`renet`, `quinn`) | Reliable/unreliable channel networking, resource streaming, state replication. |

### 3.2 The `.plc` Package Specification

The `.plc` (*ProtoLattice Package*) format is a standard ZIP archive structure renamed with the `.plc` extension.

```text
my_game.plc
├── manifest.ron         # Metadata, package title, entry points, engine version
├── server.wasm          # WebAssembly module containing game logic & systems
└── assets/              # Raw game assets
    ├── textures/        # 2D tileset atlases & UI textures
    ├── models/          # 3D voxel models / mesh data
    ├── sounds/          # Audio files (SFX, music)
    └── data/            # RON/JSON data configurations
```

### 3.3 Server & WASM Execution Flow

1. **Initialization:** The ProtoLattice Host application reads and unpacks the `.plc` archive.
2. **Module Instantiation:** `server.wasm` is loaded into the host environment (`wasmtime`).
3. **Loop & Ticking:** The server host drives the game loop at a fixed tick rate (e.g., 20 Hz). Each tick invokes exported functions in the WASM module to update the Bevy world state.
4. **Client Connection:** Upon player connection, the host server negotiates asset manifests and streams required `assets/` files to the client before initiating world state synchronization.

---

## 4. Minimum Viable Product (MVP) Requirements

The goal of the MVP is to establish a complete **vertical slice**: a dedicated server loading a `.plc` package, streaming resources to a connecting client, and rendering a synchronized, playable player character on a tile or voxel grid.

### 4.1 Functional Requirements

* **Package Management (`proto-lattice-pack`):**
  * CLI tool to pack a project folder into a `.plc` archive and validate manifest formatting.
* **Server Runtime (`proto-lattice-server`):**
  * Headless server executable capable of unarchiving `.plc` files.
  * Embedded WASM runner capable of invoking Bevy tick systems.
  * Server-to-client asset streaming protocol over UDP/QUIC.
* **Client Application (`proto-lattice-client`):**
  * Native Bevy-based client executable.
  * Asset cache pipeline to store downloaded `.plc` assets during runtime.
  * 2D Tilemap / 3D Voxel rendering layer driven by server state broadcasts.
* **Network & State Replication:**
  * Handshake and manifest exchange.
  * Server-authoritative player movement synchronization.
  * Basic client-side prediction and server reconciliation.

### 4.2 Non-Functional Requirements

* **Security:** WASM execution sandbox must prohibit host system file access or arbitrary network sockets.
* **Cross-Platform:** Native compilation targets for Windows, macOS, and Linux for host, client, and developer CLI.

---

## 5. Development Phases to MVP

### Phase 1: Foundation & Package Format Specification
* Define project crate structure (`proto-lattice-common`, `proto-lattice-server`, `proto-lattice-client`, `proto-lattice-cli`).
* Implement the `.plc` archive reader/writer using Rust ZIP/compression libraries.
* Establish shared serializable network packets and Bevy component types.

### Phase 2: Headless Server Host & WASM Sandbox Integration
* Integrate `wasmtime` or `wasmer` into the server host binary.
* Establish Host <-> WASM bindings (FFI/Component Model) for system ticking and event queueing.
* Validate a minimal Rust/Bevy script compiling to `wasm32-unknown-unknown` running inside the headless host loop.

### Phase 3: Networking & Asset Streaming Protocol
* Implement networking using `renet` or `quinn`.
* Build the asset manifest handshake: Client requests asset hashes, Server streams missing files from `.plc/assets/`.
* Implement local client caching of received asset files.

### Phase 4: Client Rendering & World Synchronization
* Implement basic Bevy rendering on the client for 2D tilemaps and 3D voxel chunks.
* Connect input events (WASD / Movement) from client to server.
* Wire state replication from WASM server logic to client rendering entities.

### Phase 5: End-to-End Vertical Slice Demo
* Construct a sample `.plc` test game package (*e.g., a simple multi-user grid walk demo*).
* Launch the server with the test package.
* Connect multiple client instances to verify resource streaming, player joining, movement, and visual rendering synchronization.