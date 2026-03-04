# Firmware Overview

The WakeLink firmware runs on ESP8266 and ESP32 microcontrollers, enabling secure Wake-on-LAN over the Internet.

## Features

- 🔐 **End-to-end encryption** — XChaCha20-Poly1305 AEAD
- 📡 **Persistent WebSocket** — Always connected, instant wake
- 🔄 **OTA Updates** — Update firmware without physical access
- 🏠 **Local Mode** — Works without server on local network
- 💤 **Power Efficient** — Deep sleep support for battery operation
- 🔧 **Web Provisioning** — Easy WiFi setup via captive portal

## Supported Hardware

| Board | Status | Notes |
|-------|--------|-------|
| ESP8266 (NodeMCU) | ✅ Recommended | Lowest cost, proven stable |
| ESP8266 (Wemos D1 Mini) | ✅ Supported | Compact form factor |
| ESP32 | ✅ Supported | More RAM/flash, faster |
| ESP32-S2 | ✅ Supported | USB OTG support |
| ESP32-C3 | ✅ Supported | RISC-V based |

## Quick Links

<div class="grid cards" markdown>

-   :material-download:{ .lg .middle } __Flashing Guide__

    ---

    Install firmware on your ESP board

    [:octicons-arrow-right-24: Flashing](flashing.md)

-   :material-cog:{ .lg .middle } __Configuration__

    ---

    WiFi, server, and device settings

    [:octicons-arrow-right-24: Configuration](configuration.md)

-   :material-update:{ .lg .middle } __OTA Updates__

    ---

    Remote firmware updates

    [:octicons-arrow-right-24: OTA Updates](ota.md)

-   :material-wrench:{ .lg .middle } __Troubleshooting__

    ---

    Common issues and solutions

    [:octicons-arrow-right-24: Troubleshooting](troubleshooting.md)

</div>

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    WakeLink Firmware                     │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │  WebSocket      │  │  Local HTTP     │              │
│  │  Client         │  │  Server         │              │
│  └────────┬────────┘  └────────┬────────┘              │
│           │                    │                        │
│  ┌────────▼────────────────────▼────────┐              │
│  │         Session Manager              │              │
│  │   - Connection state                  │              │
│  │   - Reconnection logic                │              │
│  └────────────────┬─────────────────────┘              │
│                   │                                     │
│  ┌────────────────▼─────────────────────┐              │
│  │         Command Handler              │              │
│  │   - Parse commands                    │              │
│  │   - Execute actions                   │              │
│  └────────────────┬─────────────────────┘              │
│                   │                                     │
│  ┌────────────────▼─────────────────────┐              │
│  │         Crypto Manager               │              │
│  │   - XChaCha20-Poly1305               │              │
│  │   - HMAC-SHA256                       │              │
│  │   - Key derivation                    │              │
│  └────────────────┬─────────────────────┘              │
│                   │                                     │
│  ┌────────────────▼─────────────────────┐              │
│  │         WoL Sender                   │              │
│  │   - Build magic packet               │              │
│  │   - UDP broadcast                     │              │
│  └──────────────────────────────────────┘              │
├─────────────────────────────────────────────────────────┤
│                    Platform Layer                        │
│   WiFi │ Flash Storage │ Timer │ LED │ OTA │ mDNS       │
└─────────────────────────────────────────────────────────┘
```

## LED Status Codes

| Pattern | Meaning |
|---------|---------|
| Slow blink (1Hz) | Connecting to WiFi |
| Fast blink (4Hz) | Connecting to server |
| Solid ON | Connected and ready |
| Double blink | Command received |
| Triple blink | Error (check logs) |
| OFF | Deep sleep or no power |

## Power Consumption

| State | ESP8266 | ESP32 |
|-------|---------|-------|
| Active (WiFi TX) | ~170mA | ~240mA |
| Active (idle) | ~70mA | ~80mA |
| Light sleep | ~1mA | ~0.8mA |
| Deep sleep | ~20μA | ~10μA |

!!! tip "Battery Operation"
    For battery-powered setups:
    
    1. Enable deep sleep in config
    2. Set wake interval (e.g., check every 5 min)
    3. Use 18650 cell + TP4056 charger
    4. Expected life: ~30 days on 3000mAh

## Memory Usage

| Resource | ESP8266 | ESP32 |
|----------|---------|-------|
| Flash (firmware) | ~450KB | ~600KB |
| Flash (config) | 4KB | 4KB |
| RAM (heap free) | ~25KB | ~200KB |

## Source Code

The firmware source is available at:

- GitHub: [wakelink/wakelink-firmware](https://github.com/wakelinkdev/wakelink-firmware)
- License: MIT

### Directory Structure

```
wakelink-firmware/
├── WakeLink/
│   ├── WakeLink.ino      # Main entry point
│   ├── config.cpp/h      # Configuration management
│   ├── cloud.cpp/h       # WebSocket client
│   ├── command.cpp/h     # Command parser
│   ├── packet.cpp/h      # Packet builder
│   ├── CryptoManager.cpp/h # Encryption
│   ├── SessionManager.cpp/h # Session state
│   ├── provisioning.cpp/h # WiFi setup
│   ├── ota_manager.cpp/h # OTA updates
│   └── platform.h        # Platform abstraction
├── platformio.ini        # PlatformIO config
└── README.md
```

---

Continue to [Flashing Guide →](flashing.md)
