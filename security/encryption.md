# Encryption Details

Technical details of cryptographic primitives used in WakeLink.

## Algorithm Summary

| Purpose | Algorithm | Key Size | Notes |
|---------|-----------|----------|-------|
| Symmetric Encryption | XChaCha20-Poly1305 | 256-bit | AEAD cipher |
| Message Authentication | HMAC-SHA256 | 256-bit | Packet integrity |
| Key Derivation | HKDF-SHA256 | Variable | Session keys |
| Hashing | SHA-256 | 256-bit | Chain hashing |
| Password Hashing | Argon2id | Variable | User passwords |

---

## XChaCha20-Poly1305

### Why XChaCha20-Poly1305?

| Feature | Benefit |
|---------|---------|
| **AEAD** | Encryption + authentication in one |
| **256-bit key** | High security margin |
| **192-bit nonce** | Safe random nonce generation |
| **No padding** | Ciphertext = plaintext length |
| **Fast** | Optimized for software |
| **Safe** | Resistant to timing attacks |

### Comparison with AES-GCM

| Feature | XChaCha20-Poly1305 | AES-256-GCM |
|---------|---------------------|-------------|
| Nonce size | 192-bit | 96-bit |
| Nonce safety | Random OK | Counter required |
| Hardware | Software-optimized | Needs AES-NI |
| ESP8266 speed | Fast | Slow (no AES-NI) |

### Parameters

```
Key:         256 bits (32 bytes)
Nonce:       192 bits (24 bytes)
Tag:         128 bits (16 bytes)
Max message: 2^64 bytes
```

### Usage

```c
// Encryption
uint8_t nonce[24];
getrandom(nonce, 24);

uint8_t ciphertext[plaintext_len + 16];
xchacha20_poly1305_encrypt(
    ciphertext,      // output
    plaintext,       // input
    plaintext_len,
    key,             // 32 bytes
    nonce,           // 24 bytes
    NULL,            // additional data
    0
);
```

---

## HMAC-SHA256

Used for packet signatures (in addition to Poly1305):

```
signature = HMAC-SHA256(mac_key, canonical_message)
```

### Why Double Authentication?

1. **Poly1305** authenticates the encrypted payload
2. **HMAC** authenticates the entire packet structure

This provides:
- Defense in depth
- Independent verification layers
- Compatibility with relay servers

### Implementation

```c
uint8_t signature[32];
hmac_sha256(
    mac_key,         // 32 bytes
    message,
    message_len,
    signature        // output: 32 bytes
);
```

---

## Key Derivation (HKDF)

### Pre-Shared Key

Generated during device registration:
```
PSK = random(32)  // 256 bits of entropy
```

### Session Key Derivation

```
Session Key = HKDF-SHA256(
    IKM  = PSK,
    salt = SHA256(device_id || timestamp || server_nonce),
    info = "ewsp-v1-session",
    L    = 64
)

encryption_key = Session_Key[0:32]   // First 32 bytes
mac_key        = Session_Key[32:64]  // Last 32 bytes
```

### Why HKDF?

- **Extract**: Concentrates entropy from PSK
- **Expand**: Generates multiple keys safely
- **Standard**: RFC 5869, widely reviewed

---

## Chain Hashing (SHA-256)

Each packet linked to previous via hash:

```
packet_hash = SHA256(canonical_packet_bytes)
```

### Chain Structure

```
[Genesis] → [Packet 1] → [Packet 2] → [Packet 3]
  hash0   ←   prev    ←    prev    ←    prev
```

### Verification

```c
uint8_t expected_prev[32];
sha256(previous_packet, previous_len, expected_prev);

if (memcmp(current_packet.prev, expected_prev, 32) != 0) {
    return ERROR_CHAIN_BROKEN;
}
```

---

## Password Hashing (Argon2id)

Server-side, for user passwords:

### Parameters

```
Argon2id:
  Memory:     64 MB
  Time:       3 iterations
  Parallelism: 4
  Salt:       16 bytes (random)
  Output:     32 bytes
```

### Why Argon2id?

- **Memory-hard**: Resists GPU/ASIC attacks
- **Time-hard**: Resists brute force
- **Side-channel resistant**: Argon2id variant
- **Winner**: Password Hashing Competition

---

## Random Number Generation

### Requirements

All random numbers must be cryptographically secure:

| Platform | Source |
|----------|--------|
| ESP8266 | `esp_random()` (hardware RNG) |
| ESP32 | `esp_random()` (hardware RNG) |
| Linux | `/dev/urandom` / `getrandom()` |
| Python | `secrets.token_bytes()` |

### Verification

```c
// ESP8266/ESP32
uint32_t random_word = esp_random();

// Never use rand() or similar!
// ❌ rand()
// ❌ random()
// ❌ Math.random()
```

---

## Constant-Time Operations

To prevent timing attacks:

### Comparison

```c
// ✅ Constant time
int secure_compare(const uint8_t *a, const uint8_t *b, size_t len) {
    uint8_t diff = 0;
    for (size_t i = 0; i < len; i++) {
        diff |= a[i] ^ b[i];
    }
    return diff == 0;
}

// ❌ Variable time (vulnerable)
int insecure_compare(const uint8_t *a, const uint8_t *b, size_t len) {
    return memcmp(a, b, len) == 0;  // May return early
}
```

### Selection

```c
// ✅ Constant time selection
uint8_t select(uint8_t a, uint8_t b, int choose_a) {
    uint8_t mask = -(choose_a != 0);  // All 1s or all 0s
    return (a & mask) | (b & ~mask);
}
```

---

## Key Storage

### On ESP Device

```
Flash Memory (encrypted partition):
┌─────────────────────────────┐
│  Device Token (32 bytes)    │
│  PSK (32 bytes)             │ ← AES-256 encrypted
│  Session State (variable)   │
└─────────────────────────────┘
```

ESP32 supports hardware flash encryption.

### On Server

```
PostgreSQL (encrypted column):
┌─────────────────────────────┐
│  device_token_hash          │ ← Argon2 for verification
│  device_psk_encrypted       │ ← AES-256 with master key
└─────────────────────────────┘
```

### Key Hierarchy

```
Master Key (HSM/KMS)
    │
    ├── User Password Keys (Argon2id per user)
    │
    └── Device Encryption Keys (HKDF per device)
            │
            └── Session Keys (HKDF per session)
```

---

## Cryptographic Libraries

### ESP8266

- **BearSSL** (built-in)
- **Crypto** library by Rhys Weatherley

### ESP32

- **mbedTLS** (built-in)
- Hardware acceleration for SHA, AES

### Python (Server/CLI)

- **cryptography** library (PyCA)
- Uses OpenSSL backend

### Implementation References

```c
// ESP8266/ESP32
#include "CryptoManager.h"

ewsp_crypto_init();
ewsp_encrypt(key, nonce, plaintext, len, ciphertext);
ewsp_decrypt(key, nonce, ciphertext, len, plaintext);
ewsp_hmac(key, message, len, output);
ewsp_hkdf(ikm, salt, info, output, output_len);
```

---

## Security Auditing

The cryptographic implementation has been reviewed for:

- [ ] Correct algorithm usage
- [ ] Proper nonce handling
- [ ] Constant-time comparisons
- [ ] Secure random generation
- [ ] Key erasure after use
- [ ] Buffer overflow prevention

For security issues, see [Security Policy](https://github.com/wakelinkdev/wakelink/security/policy).
