# EWSP Protocol Specification v1.0

**Encrypted Wake-on-LAN Secure Protocol**

| Field         | Value                  |
|---------------|------------------------|
| Version       | 1.0                    |
| Status        | Stable                 |
| Date          | February 2026          |
| Author        | WakeLink Team          |
| License       | CC BY-SA 4.0           |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Security Model](#2-security-model)
3. [Cryptographic Primitives](#3-cryptographic-primitives)
4. [Key Management](#4-key-management)
5. [Packet Structure](#5-packet-structure)
6. [Blockchain Chaining](#6-blockchain-chaining)
7. [Session Management](#7-session-management)
8. [Command Protocol](#8-command-protocol)
9. [Transport Modes](#9-transport-modes)
10. [Security Considerations](#10-security-considerations)
11. [Appendix A: Test Vectors](#appendix-a-test-vectors)
12. [Appendix B: Reference Implementations](#appendix-b-reference-implementations)

---

## 1. Introduction

### 1.1 Purpose

EWSP (Encrypted Wake-on-LAN Secure Protocol) is a secure communication protocol designed for remote device management, with primary focus on Wake-on-LAN operations. The protocol provides:

- **End-to-end encryption** using XChaCha20-Poly1305 AEAD
- **Message authentication** via HMAC-SHA256
- **Replay protection** through blockchain-style packet chaining
- **Forward secrecy** via session key ratcheting
- **Mutual authentication** between client and device

### 1.2 Goals

| Goal                    | Solution                                      |
|-------------------------|-----------------------------------------------|
| Confidentiality         | XChaCha20-Poly1305 AEAD encryption            |
| Integrity               | HMAC-SHA256 signatures                        |
| Authentication          | Shared device token + mutual challenge-response |
| Replay Protection       | Blockchain chaining with sequence numbers     |
| Forward Secrecy         | HKDF key derivation + key ratcheting          |
| DoS Resistance          | Rate limiting + sequence jump limits          |

### 1.3 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          WakeLink System                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌──────────┐                                      ┌──────────────┐     │
│   │  Client  │◄──────── EWSP v1.0 ────────────────►│   Firmware   │     │
│   │ (Python/ │         TCP Direct                  │  (ESP8266/   │     │
│   │ Android/ │         or                          │   ESP32)     │     │
│   │ Desktop) │         WSS Relay                   │              │     │
│   └──────────┘                                      └──────────────┘     │
│        │                                                  │              │
│        │                 ┌──────────┐                     │              │
│        └────────────────►│  Server  │◄────────────────────┘              │
│                          │  (Relay) │                                    │
│                          └──────────┘                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.4 Protocol Layers

```
┌─────────────────────────────────────────┐
│           Application Layer             │   Commands (wake, ping, status)
├─────────────────────────────────────────┤
│            Session Layer                │   Key ratcheting, counters
├─────────────────────────────────────────┤
│            Packet Layer                 │   Blockchain chaining, JSON envelope
├─────────────────────────────────────────┤
│           Crypto Layer                  │   XChaCha20-Poly1305, HMAC, HKDF
├─────────────────────────────────────────┤
│          Transport Layer                │   TCP / WebSocket Secure (WSS)
└─────────────────────────────────────────┘
```

---

## 2. Security Model

### 2.1 Threat Model

EWSP v1.0 protects against:

| Threat                    | Mitigation                                      |
|---------------------------|-------------------------------------------------|
| Eavesdropping             | XChaCha20-Poly1305 encryption                   |
| Message tampering         | HMAC-SHA256 signatures + AEAD tags              |
| Replay attacks            | Blockchain chaining + monotonic sequence        |
| Man-in-the-middle         | Pre-shared device token + mutual authentication |
| Brute force               | 256-bit keys + rate limiting                    |
| Side-channel attacks      | Constant-time comparison functions              |
| Key compromise            | Forward secrecy via key ratcheting              |
| DoS via sequence jump     | Maximum sequence jump limit (100)               |

### 2.2 Trust Assumptions

1. **Device token** is securely provisioned during pairing
2. **Transport layer** (TLS 1.3 for WSS) provides in-transit security
3. **Random number generator** is cryptographically secure (CSPRNG)
4. **System time** is reasonably accurate (±1 day for session expiry)

### 2.3 Security Levels

| Level  | Key Size | Security Bits | Use Case                |
|--------|----------|---------------|-------------------------|
| EWSP   | 256-bit  | 128-bit       | Default for all devices |

---

## 3. Cryptographic Primitives

### 3.1 Hash Functions

#### SHA-256 (FIPS 180-4)

Used for:
- Packet hashing (blockchain)
- Key derivation (HKDF)
- HMAC construction

```
Output: 32 bytes (256 bits)
```

### 3.2 Message Authentication

#### HMAC-SHA256 (RFC 2104)

Used for:
- Packet signature verification
- OTA firmware signature

```
Key:    32 bytes
Input:  Variable length
Output: 32 bytes
```

**Signature Format:**

```
signature = HMAC-SHA256(auth_key, "v" || id || seq || prev || payload)
```

### 3.3 Key Derivation

#### HKDF-SHA256 (RFC 5869)

Used for:
- Deriving encryption and authentication keys from master token
- Session key derivation
- Key ratcheting

```
Salt:   Optional (32 bytes or empty)
IKM:    Input key material
Info:   Context/application-specific info
Output: Variable length key material
```

**Key Separation:**

```
enc_key  = HKDF(token, "", "wakelink_encryption_v1", 32)
auth_key = HKDF(token, "", "wakelink_authentication_v1", 32)
```

### 3.4 Encryption

#### XChaCha20-Poly1305 AEAD (RFC 7539 + XSalsa extension)

Used for:
- Payload encryption
- Session-encrypted messages

```
Key:            32 bytes
Nonce:          24 bytes (extended nonce)
Plaintext:      Variable length
Associated Data: Variable length (packet header fields)
Ciphertext:     len(plaintext) bytes
Tag:            16 bytes (appended to ciphertext)
```

**AEAD Construction:**

```
subkey = HChaCha20(key, nonce[0:16])
nonce' = 0x00000000 || nonce[16:24]
ciphertext = ChaCha20(subkey, nonce', 1, plaintext)
tag = Poly1305(poly_key, aad || ciphertext || lengths)
```

#### XChaCha20 (Stream Cipher)

Used for:
- Legacy payload encryption (with separate HMAC)
- Non-AEAD contexts

```
Key:    32 bytes
Nonce:  24 bytes
Counter: 32 bits (starting at 0 or 1)
```

### 3.5 Random Generation

All random values MUST be generated using a cryptographically secure PRNG:

| Platform      | Source                          |
|---------------|---------------------------------|
| Windows       | CryptGenRandom / BCryptGenRandom |
| Linux         | /dev/urandom / getrandom()      |
| ESP8266/ESP32 | esp_random()                    |
| Android       | SecureRandom                    |
| iOS           | SecRandomCopyBytes              |

---

## 4. Key Management

### 4.1 Device Token

The device token is a 32+ character secret shared between device and authorized clients.

**Requirements:**
- Minimum 32 characters
- Generated using CSPRNG
- Stored securely (encrypted on device, keychain on mobile)
- Never transmitted in plaintext

**Provisioning:**
1. Device generates token during first setup
2. Displayed as QR code on setup portal
3. Client scans QR and stores token

### 4.2 Key Derivation Tree

```
                    Device Token (master secret)
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         enc_key         auth_key        session_key
     (encryption)    (authentication)   (session mgmt)
              │
    ┌─────────┴─────────┐
    ▼                   ▼
 chacha_key        ota_key
(XChaCha20)    (OTA signing)
```

**Derivation Context Strings:**

| Key Purpose       | HKDF Info String                    |
|-------------------|-------------------------------------|
| Encryption        | `wakelink_encryption_v1`            |
| Authentication    | `wakelink_authentication_v1`        |
| Session           | `wakelink_session_v1`               |
| OTA Signing       | `wakelink_ota_signing_v1`           |
| Binding           | `wakelink_binding_v1`               |

### 4.3 Key Rotation

Key rotation is supported via the `key_rotate` command:

1. Server generates new encrypted token
2. Client sends `update_token` command
3. Device decrypts and stores new token
4. Device keeps previous token for 24h (rollback)
5. All sessions use new key after rotation

---

## 5. Packet Structure

### 5.1 Outer Packet (JSON)

Every EWSP v1.0 packet is a JSON object with the following structure:

```json
{
  "v": "1.0",
  "id": "WL35080814",
  "seq": 42,
  "prev": "a1b2c3d4e5f6...",
  "p": "base64_encrypted_payload...",
  "sig": "hex_hmac_signature..."
}
```

| Field  | Type   | Size       | Description                              |
|--------|--------|------------|------------------------------------------|
| `v`    | string | 3 chars    | Protocol version ("1.0")                 |
| `id`   | string | 10 chars   | Device identifier (WL + 8 hex)           |
| `seq`  | uint64 | 1-20 digits| Monotonic sequence number                |
| `prev` | string | 64 hex     | SHA256 hash of previous packet           |
| `p`    | string | variable   | Base64-encoded encrypted payload         |
| `sig`  | string | 64 hex     | HMAC-SHA256 signature                    |

**Field Order:** Fields SHOULD appear in the order shown for canonical serialization.

### 5.2 Inner Packet (Decrypted JSON)

The decrypted payload contains:

```json
{
  "cmd": "wake",
  "d": {
    "mac": "AA:BB:CC:DD:EE:FF",
    "port": 9
  },
  "rid": "X7K2M9P1"
}
```

| Field | Type   | Description                              |
|-------|--------|------------------------------------------|
| `cmd` | string | Command name (see §8)                    |
| `d`   | object | Command-specific data                    |
| `rid` | string | Request ID (8 alphanumeric chars)        |

### 5.3 Packet Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        OUTER PACKET (JSON)                       │
├─────────────────────────────────────────────────────────────────┤
│  {                                                               │
│    "v": "1.0",              ◄─── Protocol version               │
│    "id": "WL35080814",      ◄─── Device identifier              │
│    "seq": 42,               ◄─── Sequence number                │
│    "prev": "a1b2c3...",     ◄─── Previous packet hash (64 hex)  │
│    "p": "base64...",        ◄─── Encrypted inner packet         │
│    "sig": "hmac64..."       ◄─── HMAC signature (64 hex)        │
│  }                                                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ Decrypt(enc_key, nonce, p)
┌─────────────────────────────────────────────────────────────────┐
│                        INNER PACKET (JSON)                       │
├─────────────────────────────────────────────────────────────────┤
│  {                                                               │
│    "cmd": "wake",           ◄─── Command name                   │
│    "d": {                   ◄─── Command data                   │
│      "mac": "AA:BB:CC:DD:EE:FF"                                  │
│    },                                                            │
│    "rid": "X7K2M9P1"        ◄─── Request ID                     │
│  }                                                               │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Signature Calculation

The signature covers all routing fields to prevent tampering:

```
message = v || "|" || id || "|" || seq || "|" || prev || "|" || p
signature = HMAC-SHA256(auth_key, message)
```

**Verification MUST:**
1. Parse outer JSON
2. Reconstruct signature message from fields
3. Compute HMAC with auth_key
4. Use constant-time comparison with provided signature

### 5.5 Payload Encryption

**Encryption (client → device):**

```python
nonce = random_bytes(24)
ciphertext = XChaCha20(enc_key, nonce, 0, inner_json.encode())
payload = base64_encode(nonce + ciphertext)
```

**Decryption (device):**

```python
raw = base64_decode(payload)
nonce = raw[0:24]
ciphertext = raw[24:]
inner_json = XChaCha20(enc_key, nonce, 0, ciphertext)
```

---

## 6. Blockchain Chaining

### 6.1 Concept

Every packet contains the SHA256 hash of the previous packet in the same direction, forming a blockchain-like chain. This provides:

- **Replay protection**: Old packets cannot be replayed
- **Ordering guarantee**: Packets must arrive in sequence
- **Tamper detection**: Modifying any packet breaks the chain

### 6.2 Two-Way Chains

Each direction has an independent chain:

```
TX Chain (Client → Device):
  Genesis ──► Pkt1 ──► Pkt2 ──► Pkt3 ──► ...
     │          │         │
    seq=0     seq=1    seq=2

RX Chain (Device → Client):
  Genesis ──► Pkt1 ──► Pkt2 ──► Pkt3 ──► ...
     │          │         │
    seq=0     seq=1    seq=2
```

### 6.3 Genesis State

```
GENESIS_HASH = "0000000000000000000000000000000000000000000000000000000000000000"
GENESIS_SEQ  = 0
```

Both chains start at genesis. The first packet has:
- `seq = 1`
- `prev = GENESIS_HASH`

### 6.4 Chain Update Algorithm

**Sending a packet:**

```python
def send_packet(ctx, command, data):
    seq = ctx.tx.sequence + 1
    prev = ctx.tx.last_hash
    
    packet = create_packet(seq, prev, command, data)
    packet_hash = SHA256(packet.to_json())
    
    ctx.tx.sequence = seq
    ctx.tx.last_hash = packet_hash
    
    return packet
```

**Receiving a packet:**

```python
def receive_packet(ctx, packet):
    # Validate chain
    if packet.seq <= ctx.rx.sequence:
        return ERROR_REPLAY_DETECTED
    
    if packet.seq > ctx.rx.sequence + MAX_SEQ_JUMP:
        return ERROR_SEQUENCE_TOO_FAR
    
    if packet.prev != ctx.rx.last_hash:
        return ERROR_CHAIN_BROKEN
    
    # Verify signature
    if not verify_hmac(packet):
        return ERROR_INVALID_SIGNATURE
    
    # Update chain
    packet_hash = SHA256(packet.to_json())
    ctx.rx.sequence = packet.seq
    ctx.rx.last_hash = packet_hash
    ctx.last_received_hash = packet_hash
    
    # Decrypt and process
    return decrypt_and_process(packet)
```

### 6.5 Response Linking

Responses link back to the request they answer:

```
Request (TX):  seq=5, prev=hash(pkt4)    ───────┐
                                                │
Response (RX): seq=7, prev=hash(rsp6)   ◄──────┘
               The response KNOWS which request it answers
               via ctx.last_received_hash
```

### 6.6 Chain Recovery

If chains become desynchronized:

1. Either party can send `chain_reset` command
2. Both chains reset to genesis
3. Re-synchronization begins from seq=1

---

## 7. Session Management

### 7.1 Session Handshake

```
Client                                    Device
   │                                         │
   │── session_init ────────────────────────►│
   │   { client_random[32] }                 │
   │                                         │
   │                     Generate session_id │
   │                     Generate device_random │
   │                     Derive session keys │
   │                     Create device_proof │
   │                                         │
   │◄─────────────────── session_challenge ──│
   │   { session_id[16],                     │
   │     device_random[32],                  │
   │     device_proof[32],                   │
   │     expires_in }                        │
   │                                         │
   │  Verify device_proof                    │
   │  Derive session keys                    │
   │  Create client_proof                    │
   │                                         │
   │── session_confirm ─────────────────────►│
   │   { session_id[16],                     │
   │     client_proof[32] }                  │
   │                                         │
   │                     Verify client_proof │
   │                     Activate session    │
   │                                         │
   │◄─────────────── session_established ────│
   │   { binding_token[32],                  │
   │     expires }                           │
   │                                         │
   ═══════════ SESSION ACTIVE ══════════════
```

### 7.2 Key Derivation

**Session keys are derived using HKDF:**

```python
# Combine randoms
combined = client_random + device_random + master_token

# Extract PRK
prk = HKDF_Extract(salt=session_id, ikm=combined)

# Expand to individual keys
session_key = HKDF_Expand(prk, "ewsp_session_key", 32)
enc_key     = HKDF_Expand(prk, "ewsp_session_enc", 32)
auth_key    = HKDF_Expand(prk, "ewsp_session_auth", 32)
binding_key = HKDF_Expand(prk, "ewsp_session_bind", 32)
```

### 7.3 Proof Calculation

**Device proof (proves device knows token):**

```python
device_proof = HMAC-SHA256(
    binding_key,
    "device" + session_id + client_random + device_random
)
```

**Client proof (proves client knows token):**

```python
client_proof = HMAC-SHA256(
    binding_key,
    "client" + session_id + client_random + device_random
)
```

### 7.4 Session Encryption (AEAD)

After session establishment, messages use AEAD:

```python
def session_encrypt(session, plaintext):
    nonce = session.send_counter.to_bytes(24, 'little')
    session.send_counter += 1
    
    aad = session.session_id + nonce[:8]
    ciphertext, tag = AEAD_Encrypt(session.enc_key, nonce, aad, plaintext)
    
    return nonce + ciphertext + tag
```

### 7.5 Key Ratcheting

For forward secrecy, keys are ratcheted periodically:

```python
def ratchet_keys(session):
    new_key = HKDF(
        salt=session.ratchet_key,
        ikm=session.enc_key,
        info="ewsp_ratchet",
        length=32
    )
    
    session.ratchet_key = new_key
    session.enc_key = HKDF_Expand(new_key, "ewsp_enc", 32)
    session.auth_key = HKDF_Expand(new_key, "ewsp_auth", 32)
    session.ratchet_count = 0
```

Ratcheting occurs every `RATCHET_INTERVAL` (100) messages.

### 7.6 Replay Protection

64-bit counters + bitmap window prevent replay:

```python
def validate_counter(session, counter):
    if counter <= session.recv_counter - REPLAY_WINDOW:
        return False  # Too old
    
    if counter <= session.recv_counter:
        # Check bitmap
        bit_pos = session.recv_counter - counter
        if session.replay_bitmap & (1 << bit_pos):
            return False  # Already seen
        session.replay_bitmap |= (1 << bit_pos)
    else:
        # New high counter
        shift = counter - session.recv_counter
        session.replay_bitmap = (session.replay_bitmap << shift) | 1
        session.recv_counter = counter
    
    return True
```

---

## 8. Command Protocol

### 8.1 Command List

| Command           | Description                    | Data Fields                    |
|-------------------|--------------------------------|--------------------------------|
| `ping`            | Connection test                | —                              |
| `wake`            | Send Wake-on-LAN packet        | `mac`, `port`                  |
| `info`            | Get device information         | —                              |
| `status`          | Get device status              | —                              |
| `restart`         | Restart device                 | —                              |
| `ota_start`       | Enable OTA update mode         | —                              |
| `open_setup`      | Enter setup mode               | —                              |
| `update_token`    | Update device token            | `encrypted_token`              |
| `set_cloud`       | Configure cloud settings       | `host`, `port`, `enabled`      |
| `set_wifi`        | Configure WiFi                 | `ssid`, `password`             |
| `factory_reset`   | Reset to factory defaults      | —                              |
| `chain_sync`      | Synchronize chain state        | `tx_seq`, `rx_seq`, `hashes`   |
| `chain_reset`     | Reset chains to genesis        | —                              |
| `session_init`    | Initiate session handshake     | `client_random`                |
| `session_confirm` | Confirm session                | `session_id`, `client_proof`   |

### 8.2 Command Request Format

```json
{
  "cmd": "wake",
  "d": {
    "mac": "AA:BB:CC:DD:EE:FF",
    "port": 9
  },
  "rid": "X7K2M9P1"
}
```

### 8.3 Response Format

**Success:**

```json
{
  "status": "ok",
  "rid": "X7K2M9P1",
  "d": {
    "sent": true,
    "target": "AA:BB:CC:DD:EE:FF"
  }
}
```

**Error:**

```json
{
  "status": "error",
  "rid": "X7K2M9P1",
  "error": {
    "code": 1001,
    "message": "Invalid MAC address format"
  }
}
```

### 8.4 Request ID (rid)

- 8 characters: A-Z, 0-9
- Generated by client
- Echoed in response
- Used for request/response correlation

---

## 9. Transport Modes

### 9.1 TCP Direct (Local)

For local network communication:

```
Client ◄──── TCP/JSON ────► Device
         Port: 8080 (default)
         Keepalive: 30s
```

**Connection flow:**
1. Client connects to device IP:port
2. No additional handshake (EWSP handles auth)
3. Bidirectional JSON messages (newline-delimited)

### 9.2 WebSocket Secure (Cloud Relay)

For remote access via cloud server:

```
Client ◄──── WSS ────► Server ◄──── WSS ────► Device
                     (Relay)
```

**WSS Protocol:**
1. Client connects to `wss://server/wss`
2. Authenticates with API token
3. Subscribes to device channel
4. Server relays EWSP packets bidirectionally
5. Automatic reconnect with exponential backoff

**Message Format (WebSocket):**

```json
{
  "type": "command",
  "device_id": "WL35080814",
  "payload": { /* EWSP packet */ }
}
```

### 9.3 Transport Security

| Mode       | Transport Security      | EWSP Security           |
|------------|-------------------------|-------------------------|
| TCP Local  | None (trusted LAN)      | Full encryption + auth  |
| WSS Cloud  | TLS 1.3                 | Full encryption + auth  |

**Note:** EWSP encryption is applied regardless of transport. TLS is defense-in-depth.

---

## 10. Security Considerations

### 10.1 Implementation Requirements

| Requirement                          | Why                                      |
|--------------------------------------|------------------------------------------|
| Use constant-time comparison          | Prevent timing attacks on HMAC/tokens    |
| Zero keys after use                   | Prevent memory disclosure                |
| Validate all input lengths            | Prevent buffer overflows                 |
| Check sequence bounds                 | Prevent integer overflow                 |
| Limit max packet size                 | Prevent memory exhaustion                |
| Use CSPRNG for all random             | Prevent nonce/key prediction             |

### 10.2 Recommended Limits

| Parameter              | Value      | Rationale                    |
|------------------------|------------|------------------------------|
| Max packet size        | 64 KB      | Prevent memory exhaustion    |
| Max sequence jump      | 100        | Prevent DoS via fast-forward |
| Session timeout        | 5 min idle | Limit session hijack window  |
| Max session lifetime   | 24 hours   | Force periodic re-auth       |
| Auth failure lockout   | 5 attempts | Prevent brute force          |
| Rate limit             | 10 req/s   | Prevent DoS                  |

### 10.3 Error Handling

**Never reveal sensitive information in errors:**

```
✓ "Authentication failed"
✗ "Invalid token: expected 'abc123...'"
```

**Use unified error codes:**

| Range     | Category          |
|-----------|-------------------|
| 1xxx      | Protocol errors   |
| 2xxx      | Authentication    |
| 3xxx      | Command errors    |
| 4xxx      | Chain errors      |
| 5xxx      | Crypto errors     |

### 10.4 Logging Guidelines

**DO log:**
- Connection attempts (IP, device ID)
- Authentication failures
- Chain violations
- Rate limit triggers

**DO NOT log:**
- Tokens or keys
- Decrypted payloads
- Full packet contents

---

## Appendix A: Test Vectors

### A.1 SHA-256

```
Input:  "" (empty)
Output: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855

Input:  "hello"
Output: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

### A.2 HMAC-SHA256

```
Key:    "wakelink_test_key_32_characters!"
Input:  "test message"
Output: b1392a3a6768fdd5319792f45e7cc8922cc680db224e40da1898905d31a4d998
```

### A.3 HKDF-SHA256

```
IKM:    "input key material"
Salt:   "" (empty)
Info:   "wakelink_encryption_v1"
Length: 32
Output: [32 bytes of derived key]
```

### A.4 XChaCha20

```
Key:    [32 bytes of 0x00]
Nonce:  [24 bytes of 0x00]
Input:  [64 bytes of 0x00]
Output: [64 bytes of keystream]
```

### A.5 Packet Example

**Input:**

```
Device Token: "my_super_secret_device_token_32ch"
Device ID: "WL35080814"
Command: "wake"
Data: {"mac": "AA:BB:CC:DD:EE:FF"}
TX Sequence: 1
TX Last Hash: GENESIS_HASH
```

**Output Packet:**

```json
{
  "v": "1.0",
  "id": "WL35080814",
  "seq": 1,
  "prev": "0000000000000000000000000000000000000000000000000000000000000000",
  "p": "base64_of_nonce_and_encrypted_inner_json",
  "sig": "hmac_sha256_of_v_id_seq_prev_p"
}
```

---

## Appendix B: Reference Implementations

### B.1 C (ewsp-core)

```
Repository: ewsp-core/
Files:      src/ewsp_*.c, include/ewsp_*.h
Build:      CMake
Platforms:  Linux, Windows, macOS, ESP8266/ESP32
```

### B.2 Python

```
Repository: ewsp-core/bindings/python/
Package:    ewsp-core (pip installable)
API:        ewsp_core.CryptoManager, PacketManager
Fallback:   Pure Python (crypto_pure.py)
```

### B.3 Kotlin/JNI

```
Repository: ewsp-core/bindings/kotlin/
Package:    org.wakelink.ewsp
API:        Crypto.create(), Packet.create()
Fallback:   Pure Kotlin (CryptoPure.kt)
```

### B.4 Firmware

```
Repository: wakelink-firmware/WakeLink/
Platform:   ESP8266, ESP32
Framework:  Arduino
Files:      packet.cpp, SessionManager.cpp
```

---

## Changelog

| Version | Date       | Changes                                    |
|---------|------------|--------------------------------------------|
| 1.0     | 2026-02    | Initial release. Blockchain chaining, AEAD |

---

**End of EWSP Protocol Specification v1.0**

*This specification is released under CC BY-SA 4.0. Implementations welcome.*
