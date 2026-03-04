[🇬🇧 English](index.md) | [🇷🇺 Русский](index_RU.md)

# Security Overview

WakeLink is designed with security as a core principle.

## Security Model

```
┌────────────────────────────────────────────────────────────┐
│                    Security Layers                          │
├────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Transport Security (TLS 1.3)                 │  │
│  │    • Server authentication (certificates)            │  │
│  │    • Perfect forward secrecy                         │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Application Encryption (XChaCha20)           │  │
│  │    • End-to-end encryption                           │  │
│  │    • Server cannot read commands                     │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Message Integrity (HMAC-SHA256)              │  │
│  │    • Packet authenticity                             │  │
│  │    • Tamper detection                                │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Replay Protection (Chain Hashing)            │  │
│  │    • Sequence numbers                                │  │
│  │    • Packet chaining                                 │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

## Key Features

| Feature | Implementation | Purpose |
|---------|---------------|---------|
| **E2E Encryption** | XChaCha20-Poly1305 | Server can't read commands |
| **Authentication** | JWT + Device Tokens | Verify identity |
| **Integrity** | HMAC-SHA256 | Detect tampering |
| **Anti-Replay** | Sequence + Chain | Prevent replay attacks |
| **Transport Security** | TLS 1.3 | Protect in transit |
| **Key Derivation** | HKDF-SHA256 | Secure key expansion |

---

## Threat Model

### What WakeLink Protects Against

✅ **Eavesdropping** — Commands encrypted end-to-end
✅ **Man-in-the-Middle** — TLS + certificate validation
✅ **Replay Attacks** — Sequence numbers + chain hashing
✅ **Command Injection** — Strict command validation
✅ **Unauthorized Wake** — Token authentication
✅ **Server Compromise** — E2E encryption (server can't read payloads)

### What WakeLink Does NOT Protect Against

❌ **Physical access to ESP device** — Attacker could extract keys
❌ **Compromised client device** — Malware could steal tokens
❌ **DDoS attacks** — Mitigated but not prevented
❌ **Rubber-hose cryptanalysis** — Physical coercion

---

## Protocol Security (EWSP v1.0)

### Encryption

- **Algorithm**: XChaCha20-Poly1305 (AEAD)
- **Nonce**: 192-bit, unique per message
- **Key Size**: 256-bit

XChaCha20-Poly1305 provides:
- Confidentiality (encryption)
- Integrity (MAC)
- Authenticity (AEAD)

### Key Derivation

```
Pre-Shared Key (32 bytes)
        │
        ▼
    HKDF-SHA256
        │
    ┌───┴───┐
    ▼       ▼
Encryption  MAC
   Key      Key
(32 bytes) (32 bytes)
```

### Replay Protection

Two mechanisms work together:

1. **Sequence Numbers**
   - Monotonically increasing
   - Never reset during session
   - Rejected if not greater than last seen

2. **Chain Hashing**
   - Each packet includes hash of previous packet
   - Creates cryptographic chain
   - Breaks if any packet modified or replayed

```
Packet 1: prev=NULL, hash=SHA256(packet1)
Packet 2: prev=hash1, hash=SHA256(packet2)
Packet 3: prev=hash2, hash=SHA256(packet3)
```

---

## Authentication

### User Authentication

- **Method**: Email + Password → JWT
- **Password Storage**: Argon2id (memory-hard)
- **Token Expiry**: 24 hours (configurable)
- **Refresh**: Automatic with valid token

### Device Authentication

- **Method**: Pre-shared device token
- **Token Format**: 32-byte random, hex-encoded
- **Storage**: Encrypted in device flash
- **Regeneration**: Available via API

---

## Data Security

### Data at Rest

| Data | Protection |
|------|------------|
| Passwords | Argon2id hashing |
| Device tokens | Encrypted in DB |
| Session keys | Memory only, not persisted |
| Logs | No sensitive data logged |

### Data in Transit

| Channel | Protection |
|---------|------------|
| API calls | TLS 1.3 |
| WebSocket | WSS (TLS) |
| Commands | TLS + XChaCha20 |

---

## Network Security

### Required Ports

| Port | Direction | Purpose |
|------|-----------|---------|
| 443 | Outbound | HTTPS/WSS to server |

No inbound ports required on home network.

### Firewall Recommendations

```bash
# Allow only outbound HTTPS
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
```

### DNS Security

- Use DNS-over-HTTPS or DNS-over-TLS
- Validate certificates strictly
- Pin certificates for high-security deployments

---

## Quick Links

<div class="grid cards" markdown>

-   :material-protocol:{ .lg .middle } __EWSP Protocol__

    ---

    Technical specification of the encryption protocol

    [:octicons-arrow-right-24: EWSP Details](ewsp.md)

-   :material-lock:{ .lg .middle } __Encryption Details__

    ---

    Cryptographic algorithms and implementation

    [:octicons-arrow-right-24: Encryption](encryption.md)

-   :material-shield-check:{ .lg .middle } __Best Practices__

    ---

    Recommendations for secure deployment

    [:octicons-arrow-right-24: Best Practices](best-practices.md)

</div>
