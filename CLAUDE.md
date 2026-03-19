# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MeshCore is a lightweight C++ library for embedded systems enabling multi-hop packet routing over LoRa and other packet radios. It targets Arduino-compatible microcontrollers (ESP32, nRF52, RP2040, STM32) using the PlatformIO build system.

## Build Commands

```bash
# List available firmware targets
sh build.sh list

# Build a specific firmware target
sh build.sh build-firmware <target_name>
# e.g.: sh build.sh build-firmware RAK_4631_repeater

# Build all firmwares by category
sh build.sh build-firmwares
sh build.sh build-companion-firmwares
sh build.sh build-repeater-firmwares
sh build.sh build-room-server-firmwares

# Build directly with PlatformIO
pio run -e <environment-name>

# Optional env vars for builds
export FIRMWARE_VERSION=v1.0.0
export DISABLE_DEBUG=1
```

Device variant configurations live in `variants/<device-name>/platformio.ini`.

## Code Style

Formatting is enforced via `.clang-format`:
- 2-space indentation (no tabs)
- 110-column line limit
- K&R brace style, pointer alignment right

**Do NOT retroactively reformat existing code** — it creates unnecessary diffs and review noise. Only format code you are actively writing or modifying.

## Architecture

### Core Protocol Layer (`src/`)

- **`Dispatcher`** — Base packet reception/transmission layer. Manages packet lifecycle, coordinates with Radio and Clock abstractions.
- **`Mesh`** (extends Dispatcher) — Multi-hop routing logic: filtering, forwarding decisions, decryption, peer verification.
- **`Packet`** — Packet data structure with payload, headers, and path information.
- **`Identity`** — Ed25519 public/private key pairs and ECDH shared secret generation.
- **`Utils`** — Shared utility functions.

### Helper Modules (`src/helpers/`)

- **`BaseChatMesh`** — Base class for encrypted peer-to-peer chat applications.
- **`CommonCLI`** — Serial command-line interface for device configuration.
- **`ClientACL`** — Per-device access control lists.
- **`IdentityStore`** — Key/identity persistence layer.
- **`radiolib/`** — RadioLib hardware abstraction wrappers.
- **`bridges/`** — Bridge implementations (BLE, USB, WiFi, serial).
- **`sensors/`** — Sensor integrations (CayenneLPP encoding).
- **`ui/`** — UI components (buttons, display abstractions).
- **`esp32/`, `nrf52/`, `stm32/`** — Platform-specific implementations.

### Example Applications (`examples/`)

Six reference firmware applications, each compiled into many device variants:
- **`companion_radio/`** — BLE/USB/WiFi connected client node
- **`simple_repeater/`** — Network relay (forwards all packets)
- **`simple_room_server/`** — BBS-style persistent message server
- **`simple_secure_chat/`** — Terminal encrypted chat
- **`simple_sensor/`** — Remote telemetry node
- **`kiss_modem/`** — Serial KISS protocol bridge

### Device Variants (`variants/`)

69 device-specific configurations. Each variant folder contains:
- `platformio.ini` — Build flags, pin definitions, platform selection
- `target.h/cpp` — Device driver setup and hardware abstraction

### Key Design Constraints

- **No dynamic memory allocation** except during `setup()`/`begin()` — this is a hard rule for embedded stability.
- The two node roles: **Repeater** nodes forward all packets; **Companion** nodes do not repeat.
- Packet deduplication is handled by `MeshTables` (abstract, implemented per application).

## Contributing

- Submit PRs against the **`dev`** branch, not `main`.
- For significant changes, open an Issue first for discussion before implementing.

## Testing

There is no unit test framework. Validation happens via GitHub Actions CI (`.github/workflows/pr-build-check.yml`) which compiles representative targets across all supported platforms on every PR.

## Documentation

Protocol and API documentation is in `docs/` and published via MkDocs to GitHub Pages. Key references:
- `docs/packet_format.md` — V1 protocol packet structure
- `docs/companion_protocol.md` — Companion device communication protocol
- `docs/cli_commands.md` — CLI reference
