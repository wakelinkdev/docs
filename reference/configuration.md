[🇬🇧 English](configuration.md) | [🇷🇺 Русский](configuration_RU.md)

# Configuration Reference

Complete configuration options for all WakeLink components.

## Firmware Configuration

### Device Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `device.name` | string | — | Device display name (required) |
| `device.token` | string | — | Authentication token (required) |

### WiFi Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `wifi.ssid` | string | — | WiFi network name (required) |
| `wifi.password` | string | — | WiFi password (required) |
| `wifi.static_ip` | string | — | Static IP address |
| `wifi.gateway` | string | — | Default gateway |
| `wifi.subnet` | string | — | Subnet mask |
| `wifi.dns` | string | — | DNS server |

### Server Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `server.url` | string | `wss://relay.wakelink.io` | WebSocket server URL |
| `server.port` | int | 443 | Server port |
| `server.tls` | bool | true | Use TLS |
| `server.ping_interval` | int | 30000 | Ping interval (ms) |

### Target Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `target.mac` | string | — | Target MAC address (required) |
| `target.broadcast` | string | `255.255.255.255` | Broadcast address |
| `target.port` | int | 9 | WoL port |

### Local Mode

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `local.enabled` | bool | false | Enable local HTTP server |
| `local.port` | int | 80 | Local server port |
| `local.auth_required` | bool | true | Require authentication |
| `local.auth_token` | string | — | Local auth token |

### Power Management

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `power.deep_sleep` | bool | false | Enable deep sleep |
| `power.sleep_interval` | int | 300 | Sleep duration (seconds) |

### Features

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `features.led_enabled` | bool | true | Status LED |
| `features.ota_enabled` | bool | true | OTA updates |
| `features.debug_mode` | bool | false | Debug logging |

---

## Server Configuration

### Core Settings

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `SERVER_HOST` | string | `0.0.0.0` | Bind address |
| `SERVER_PORT` | int | 8000 | API port |
| `WS_PORT` | int | 8001 | WebSocket port |
| `DEBUG` | bool | false | Debug mode |
| `WORKERS` | int | 4 | Gunicorn workers |

### Security

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `JWT_SECRET` | string | — | Token signing key (required) |
| `JWT_ALGORITHM` | string | `HS256` | JWT algorithm |
| `JWT_EXPIRE_MINUTES` | int | 1440 | Token expiry |
| `CORS_ORIGINS` | string | `*` | Allowed origins |

### Database

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `DATABASE_URL` | string | — | PostgreSQL URL (required) |
| `DB_POOL_SIZE` | int | 5 | Connection pool size |
| `DB_MAX_OVERFLOW` | int | 10 | Max overflow |

### Redis

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `REDIS_URL` | string | — | Redis URL (required) |
| `REDIS_PREFIX` | string | `wakelink:` | Key prefix |

### Logging

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `LOG_LEVEL` | string | `INFO` | Log level |
| `LOG_FORMAT` | string | `json` | Format (json/text) |
| `ACCESS_LOG` | bool | true | Enable access logs |

### Rate Limiting

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `RATE_LIMIT_ENABLED` | bool | true | Enable rate limiting |
| `RATE_LIMIT_PER_MINUTE` | int | 60 | Requests per minute |

### Features

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `ENABLE_REGISTRATION` | bool | true | Allow registration |
| `ENABLE_METRICS` | bool | true | Prometheus endpoint |
| `ENABLE_DOCS` | bool | true | OpenAPI docs |

---

## CLI Configuration

### Config File: `~/.wakelink/config.yaml`

```yaml
# Server
server:
  api_url: https://wakelink-project.org/api
  ws_url: wss://relay.wakelink.io
  timeout: 30

# Authentication (managed automatically)
auth:
  token: <jwt-token>
  refresh_token: <refresh-token>

# Output preferences
output:
  format: text  # text, json, table
  color: true
  verbose: false

# Defaults
defaults:
  timeout: 30
  retries: 3
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| `WAKELINK_API_URL` | Override API URL |
| `WAKELINK_TOKEN` | Override auth token |
| `WAKELINK_CONFIG` | Config file path |
| `WAKELINK_NO_COLOR` | Disable colors |
| `WAKELINK_DEBUG` | Debug output |

---

## Android App Settings

### In-App Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Server URL | `https://wakelink-project.org/api` | API server |
| Theme | System | Light/Dark/System |
| Notifications | Enabled | Push notifications |
| Background refresh | 15 min | Status update interval |
| Timeout | 30s | Command timeout |

### Widget Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| Device | — | Selected device(s) |
| Update interval | 15 min | Refresh frequency |
| One-tap wake | Disabled | Wake on tap |

---

## Example Configurations

### Minimal Firmware Config

```json
{
  "device": {"name": "mydevice", "token": "xxx"},
  "wifi": {"ssid": "MyWiFi", "password": "mypass"},
  "server": {"url": "wss://relay.wakelink.io"},
  "target": {"mac": "AA:BB:CC:DD:EE:FF"}
}
```

### Production Server Config

```bash
DEBUG=false
LOG_LEVEL=INFO
JWT_SECRET=<64-char-hex>
JWT_EXPIRE_MINUTES=1440
DATABASE_URL=postgresql://user:pass@db:5432/wakelink
REDIS_URL=redis://redis:6379/0
CORS_ORIGINS=https://app.example.com
RATE_LIMIT_ENABLED=true
ENABLE_DOCS=false
WORKERS=4
```

### Development Server Config

```bash
DEBUG=true
LOG_LEVEL=DEBUG
LOG_FORMAT=text
JWT_SECRET=dev-secret-only
DATABASE_URL=postgresql://wakelink:wakelink@localhost:5432/wakelink_dev
REDIS_URL=redis://localhost:6379/0
CORS_ORIGINS=*
ENABLE_DOCS=true
```
