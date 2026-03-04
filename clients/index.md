[🇬🇧 English](index.md) | [🇷🇺 Русский](index_RU.md)

# Clients Overview

Multiple ways to interact with WakeLink.

## Available Clients

| Client | Platform | Use Case |
|--------|----------|----------|
| [Python CLI](cli.md) | Linux, macOS, Windows | Power users, automation, scripting |
| [Android App](android.md) | Android 8+ | Mobile users, quick access |
| [REST API](api.md) | Any | Integration, custom clients |

## Feature Comparison

| Feature | CLI | Android | API |
|---------|-----|---------|-----|
| Wake devices | ✅ | ✅ | ✅ |
| Device management | ✅ | ✅ | ✅ |
| Status monitoring | ✅ | ✅ | ✅ |
| Push notifications | ❌ | ✅ | ❌ |
| Home screen widget | ❌ | ✅ | ❌ |
| Batch operations | ✅ | ❌ | ✅ |
| Scripting | ✅ | ❌ | ✅ |
| Offline mode | ❌ | Partial | ❌ |

## Quick Links

<div class="grid cards" markdown>

-   :material-console:{ .lg .middle } __Python CLI__

    ---

    Full-featured command-line interface

    [:octicons-arrow-right-24: CLI Guide](cli.md)

-   :material-android:{ .lg .middle } __Android App__

    ---

    Wake devices from your phone

    [:octicons-arrow-right-24: Android Guide](android.md)

-   :material-api:{ .lg .middle } __REST API__

    ---

    Build your own integrations

    [:octicons-arrow-right-24: API Reference](api.md)

</div>

---

## Authentication

All clients use the same authentication mechanism:

1. **User Login**: Email + password → JWT token
2. **API Calls**: Include JWT in `Authorization: Bearer <token>` header
3. **Token Refresh**: Tokens expire after 24 hours, refresh automatically

### Getting API Token

=== "CLI"
    ```bash
    wakelink login
    # Token stored in ~/.wakelink/config.yaml
    ```

=== "API"
    ```bash
    curl -X POST https://wakelink-project.org/api/v1/auth/login \
      -H "Content-Type: application/json" \
      -d '{"email": "user@example.com", "password": "your-password"}'
    ```

    Response:
    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIs...",
      "token_type": "bearer",
      "expires_in": 86400
    }
    ```

---

## Device Identification

Devices can be referenced by:

| Method | Example | Notes |
|--------|---------|-------|
| Name | `office-pc` | Human-friendly, must be unique per user |
| ID | `d7a8f9b0-...` | UUID, globally unique |
| MAC | `AA:BB:CC:DD:EE:FF` | Hardware address |

```bash
# All equivalent
wakelink wake office-pc
wakelink wake d7a8f9b0-1234-5678-9abc-def012345678
wakelink wake --mac AA:BB:CC:DD:EE:FF
```

---

## Common Operations

### Wake a Device

=== "CLI"
    ```bash
    wakelink wake my-device
    ```

=== "Android"
    Tap device in list → Tap "Wake"

=== "API"
    ```bash
    curl -X POST https://wakelink-project.org/api/v1/devices/my-device/wake \
      -H "Authorization: Bearer $TOKEN"
    ```

### List Devices

=== "CLI"
    ```bash
    wakelink list
    ```

=== "API"
    ```bash
    curl https://wakelink-project.org/api/v1/devices \
      -H "Authorization: Bearer $TOKEN"
    ```

### Check Status

=== "CLI"
    ```bash
    wakelink status my-device
    ```

=== "API"
    ```bash
    curl https://wakelink-project.org/api/v1/devices/my-device \
      -H "Authorization: Bearer $TOKEN"
    ```

---

## Self-Hosted Configuration

Point clients to your own server:

=== "CLI"
    ```yaml
    # ~/.wakelink/config.yaml
    server:
      api_url: https://api.your-server.com
      ws_url: wss://ws.your-server.com
    ```

    Or via environment:
    ```bash
    export WAKELINK_API_URL=https://api.your-server.com
    ```

=== "Android"
    Settings → Server → Custom Server → Enter URL

=== "API"
    Replace `wakelink-project.org/api` with your server URL

---

## Error Handling

Common error responses:

| Code | Meaning | Action |
|------|---------|--------|
| 401 | Unauthorized | Re-login, refresh token |
| 403 | Forbidden | Check device ownership |
| 404 | Not found | Check device name/ID |
| 408 | Timeout | Device offline, retry |
| 429 | Rate limited | Slow down requests |
| 500 | Server error | Contact support |

---

## Rate Limits

Default limits:

| Endpoint | Limit |
|----------|-------|
| Login | 5/minute |
| Wake commands | 30/minute |
| Other API calls | 60/minute |

Rate limit headers:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1640000000
```
