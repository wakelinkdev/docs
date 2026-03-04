# Introduction

WakeLink enables secure Wake-on-LAN over the Internet using end-to-end encrypted communication between your devices.

## Why WakeLink?

Traditional Wake-on-LAN (WoL) has a fundamental limitation: magic packets only work on local networks. If you want to wake your home PC while traveling, you typically need:

- Port forwarding on your router (security risk)
- VPN setup (complex, always-on requirement)
- Third-party services (often closed-source, privacy concerns)

**WakeLink solves this** with a simple, secure, open-source approach:

| Challenge | WakeLink Solution |
|-----------|-------------------|
| Magic packets are local-only | WebSocket relay server bridges the gap |
| Security concerns | End-to-end encryption (XChaCha20-Poly1305) |
| Router configuration | No port forwarding needed |
| Privacy | Self-hostable, open source |
| Complexity | 5-minute setup |

## Key Features

### 🔐 End-to-End Encryption

All communication between your client and ESP device is encrypted. The relay server **cannot read your commands** — it only forwards encrypted packets.

- **Algorithm**: XChaCha20-Poly1305 (AEAD)
- **Key Exchange**: Pre-shared key + HKDF derivation
- **Replay Protection**: Sequence numbers + blockchain-style chaining

### 🌐 Works From Anywhere

Wake your computer from your office, while traveling, or from another country. As long as you have Internet access, you can send wake commands.

### 📡 Multiple Platforms

- **ESP8266/ESP32**: Low-cost, low-power devices (~$3-5)
- **CLI**: Full-featured command-line tool
- **Android**: Native mobile app
- **API**: REST endpoints for integration

### 🏠 Self-Hostable

Run your own relay server with Docker. Complete control over your data and infrastructure.

## Architecture Overview

```
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│   Client    │◄────────►│   Server    │◄────────►│ ESP Device  │
│  (CLI/App)  │   HTTPS  │   (Relay)   │    WSS   │  (Firmware) │
└─────────────┘          └─────────────┘          └──────┬──────┘
                                                         │
                                                    WoL Packet
                                                         │
                                                         ▼
                                                  ┌─────────────┐
                                                  │  Target PC  │
                                                  └─────────────┘
```

**Communication Flow**:

1. Client authenticates with server
2. Client sends encrypted wake command
3. Server relays to ESP device via persistent WebSocket
4. ESP decrypts and verifies command signature
5. ESP sends WoL magic packet on local network
6. Target PC wakes up
7. ESP sends encrypted acknowledgment

## Protocol: EWSP v1.0

WakeLink uses the **Encrypted Wake Signaling Protocol (EWSP)** — a custom protocol designed for:

- **Confidentiality**: All payloads encrypted
- **Integrity**: HMAC verification on every packet
- **Freshness**: Replay protection via chaining
- **Efficiency**: Minimal overhead for constrained devices

[Learn more about EWSP →](../security/ewsp.md)

## Next Steps

<div class="grid cards" markdown>

-   :material-clipboard-check:{ .lg .middle } __Check Requirements__

    ---

    Make sure you have the necessary hardware and software

    [:octicons-arrow-right-24: Requirements](requirements.md)

-   :material-rocket-launch:{ .lg .middle } __Quick Start__

    ---

    Get your first device online in 5 minutes

    [:octicons-arrow-right-24: Quick Start](quickstart.md)

</div>
