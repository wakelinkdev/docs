[🇬🇧 English](changelog.md) | [🇷🇺 Русский](changelog_RU.md)

# Changelog

All notable changes to WakeLink project.

## [Unreleased]

### Added
- EWSP Protocol v1.0 with enhanced security
- Comprehensive user documentation (MkDocs)
- Hotfix process documentation
- Post-mortem templates

### Changed
- Updated encryption to XChaCha20-Poly1305
- Improved error handling across all components

---

## [1.0.0] - 2024-XX-XX

### Overview
Initial stable release of WakeLink ecosystem.

### Firmware (v1.0.0)
- ESP8266/ESP32 support
- EWSP protocol implementation
- OTA updates via secure channel
- Web-based provisioning mode
- Local network operation mode
- WoL packet generation
- Status LED indication
- Deep sleep support

### Server (v1.0.0)
- FastAPI-based relay server
- WebSocket connections (EWSP)
- REST API for management
- JWT authentication
- PostgreSQL storage
- Redis for sessions
- Docker deployment
- Prometheus metrics
- Alembic migrations

### CLI (v1.0.0)
- Device discovery and pairing
- Wake commands with status
- Device management
- Configuration management
- Multiple output formats
- Shell completion

### Android App (v1.0.0)
- Kotlin Multiplatform (KMP)
- Device management UI
- One-tap wake widgets
- Push notifications
- Material Design 3
- Biometric authentication

### Security
- End-to-end encryption (EWSP)
- XChaCha20-Poly1305 AEAD
- HMAC-SHA256 authentication
- HKDF key derivation
- Replay attack protection
- Chain hashing for integrity

---

## Version Scheme

WakeLink follows [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking API/protocol changes
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes, security patches

### Component Versioning

Each component has independent versioning:
- Firmware: `wakelink-firmware/VERSION`
- Server: `wakelink-server/pyproject.toml`
- CLI: `wakelink-client/pyproject.toml`
- Android: `wakelink-android/app/build.gradle.kts`

### EWSP Protocol Version

Protocol version is negotiated during handshake:
- Current: **v1.0**
- Minimum supported: v1.0

---

## Upgrade Guides

### Firmware Update

```bash
# OTA (recommended)
wakelink ota --device <name>

# Manual flash
esptool.py write_flash 0x0 firmware.bin
```

### Server Update

```bash
docker-compose pull
docker-compose up -d
docker-compose exec server alembic upgrade head
```

### CLI Update

```bash
pip install --upgrade wakelink-cli
```

---

## Security Advisories

Security issues are documented in `SECURITY.md`.

Report vulnerabilities: security@wakelink.io (PGP key available)

---

## Links

- [Releases](https://github.com/wakelinkdev/releases)
- [Migration Guides](migration.md)
- [Security Policy](../security/index.md)
