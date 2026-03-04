# Server Configuration

Complete reference for WakeLink server configuration options.

## Configuration Methods

1. **Environment variables** (recommended for containers)
2. **`.env` file** (development)
3. **`config.yaml`** (advanced)

Environment variables take precedence over file-based configuration.

## Core Settings

### Server

| Variable | Description | Default |
|----------|-------------|---------|
| `SERVER_HOST` | Bind address | `0.0.0.0` |
| `SERVER_PORT` | API port | `8000` |
| `WS_HOST` | WebSocket bind address | `0.0.0.0` |
| `WS_PORT` | WebSocket port | `8001` |
| `WORKERS` | Gunicorn workers | `4` |
| `DEBUG` | Debug mode | `false` |

### Security

| Variable | Description | Default |
|----------|-------------|---------|
| `JWT_SECRET` | Token signing key (REQUIRED) | - |
| `JWT_ALGORITHM` | JWT algorithm | `HS256` |
| `JWT_EXPIRE_MINUTES` | Token expiration | `1440` (24h) |
| `CORS_ORIGINS` | Allowed origins (comma-separated) | `*` |
| `TRUSTED_HOSTS` | Allowed Host headers | `*` |

!!! danger "JWT_SECRET"
    **NEVER** use the default or a weak secret in production.
    
    Generate a secure secret:
    ```bash
    openssl rand -hex 32
    ```

### Database

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | - |
| `DB_POOL_SIZE` | Connection pool size | `5` |
| `DB_MAX_OVERFLOW` | Max overflow connections | `10` |
| `DB_ECHO` | Log SQL queries | `false` |

Database URL format:
```
postgresql://user:password@host:port/database
postgresql://wakelink:secret@db:5432/wakelink
```

### Redis

| Variable | Description | Default |
|----------|-------------|---------|
| `REDIS_URL` | Redis connection string | - |
| `REDIS_PREFIX` | Key prefix | `wakelink:` |
| `REDIS_TTL` | Default TTL (seconds) | `3600` |

Redis URL format:
```
redis://host:port/db
redis://:password@host:port/db
redis://redis:6379/0
```

### Logging

| Variable | Description | Default |
|----------|-------------|---------|
| `LOG_LEVEL` | Log level | `INFO` |
| `LOG_FORMAT` | Format (json/text) | `json` |
| `LOG_FILE` | Log file path | - (stdout) |
| `ACCESS_LOG` | Enable access logs | `true` |

Log levels: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`

---

## Feature Flags

| Variable | Description | Default |
|----------|-------------|---------|
| `ENABLE_REGISTRATION` | Allow new user registration | `true` |
| `ENABLE_METRICS` | Enable /metrics endpoint | `true` |
| `ENABLE_DOCS` | Enable /docs (OpenAPI) | `true` |
| `ENABLE_ADMIN_API` | Enable admin endpoints | `false` |
| `BETA_FEATURES` | Enable beta features | `false` |

---

## Rate Limiting

| Variable | Description | Default |
|----------|-------------|---------|
| `RATE_LIMIT_ENABLED` | Enable rate limiting | `true` |
| `RATE_LIMIT_PER_MINUTE` | Requests per minute | `60` |
| `RATE_LIMIT_BURST` | Burst allowance | `10` |
| `RATE_LIMIT_BY` | Key: `ip` or `user` | `user` |

### Per-Endpoint Limits

```yaml
# config.yaml
rate_limits:
  default: 60/minute
  endpoints:
    /api/v1/auth/login: 5/minute
    /api/v1/devices/*/wake: 30/minute
```

---

## WebSocket Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `WS_PING_INTERVAL` | Ping interval (seconds) | `30` |
| `WS_PING_TIMEOUT` | Ping timeout (seconds) | `10` |
| `WS_MAX_CONNECTIONS` | Max concurrent connections | `10000` |
| `WS_MAX_MESSAGE_SIZE` | Max message size (bytes) | `65536` |
| `WS_COMPRESSION` | Enable compression | `true` |

---

## Email (Optional)

For notifications and password reset:

| Variable | Description | Default |
|----------|-------------|---------|
| `SMTP_HOST` | SMTP server | - |
| `SMTP_PORT` | SMTP port | `587` |
| `SMTP_USER` | SMTP username | - |
| `SMTP_PASSWORD` | SMTP password | - |
| `SMTP_TLS` | Use TLS | `true` |
| `EMAIL_FROM` | Sender address | - |

---

## Metrics & Monitoring

| Variable | Description | Default |
|----------|-------------|---------|
| `METRICS_ENABLED` | Enable Prometheus metrics | `true` |
| `METRICS_PATH` | Metrics endpoint | `/metrics` |
| `HEALTH_PATH` | Health endpoint | `/health` |

---

## Example Configurations

### Development

```bash
# .env.development
DEBUG=true
LOG_LEVEL=DEBUG
LOG_FORMAT=text
DATABASE_URL=postgresql://wakelink:wakelink@localhost:5432/wakelink_dev
REDIS_URL=redis://localhost:6379/0
JWT_SECRET=dev-secret-not-for-production
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
ENABLE_DOCS=true
```

### Production

```bash
# .env.production
DEBUG=false
LOG_LEVEL=INFO
LOG_FORMAT=json
DATABASE_URL=postgresql://wakelink:${DB_PASSWORD}@db:5432/wakelink
REDIS_URL=redis://redis:6379/0
JWT_SECRET=${JWT_SECRET}
CORS_ORIGINS=https://wakelink-project.org
ENABLE_DOCS=false
ENABLE_ADMIN_API=false
RATE_LIMIT_ENABLED=true
WORKERS=4
```

### High-Security

```bash
# .env.secure
DEBUG=false
LOG_LEVEL=WARNING
JWT_EXPIRE_MINUTES=60
ENABLE_REGISTRATION=false
ENABLE_DOCS=false
ENABLE_ADMIN_API=false
RATE_LIMIT_PER_MINUTE=30
TRUSTED_HOSTS=wakelink.example.com
CORS_ORIGINS=https://app.example.com
```

---

## Config File (Advanced)

For complex configurations, use `config.yaml`:

```yaml
server:
  host: 0.0.0.0
  port: 8000
  workers: 4

security:
  jwt:
    secret: ${JWT_SECRET}
    algorithm: HS256
    expire_minutes: 1440
  cors:
    origins:
      - https://wakelink-project.org
    allow_credentials: true
    allow_methods:
      - GET
      - POST
      - PUT
      - DELETE
    allow_headers:
      - Authorization
      - Content-Type

database:
  url: ${DATABASE_URL}
  pool_size: 10
  max_overflow: 20
  echo: false

redis:
  url: ${REDIS_URL}
  prefix: wakelink:

logging:
  level: INFO
  format: json
  handlers:
    - type: console
    - type: file
      path: /var/log/wakelink/api.log
      rotation: daily
      retention: 30

rate_limits:
  enabled: true
  default: 60/minute
  endpoints:
    /api/v1/auth/login: 5/minute
    /api/v1/devices/*/wake: 30/minute

websocket:
  ping_interval: 30
  ping_timeout: 10
  max_connections: 10000
```

Load with:
```bash
export WAKELINK_CONFIG=/etc/wakelink/config.yaml
```

---

## Validation

Check configuration:

```bash
docker-compose exec api python -m wakelink.cli config validate
```

Output:
```
✅ Database connection: OK
✅ Redis connection: OK
✅ JWT secret: Strong (64 chars)
✅ CORS origins: Configured
⚠️  Debug mode: Enabled (disable in production)
```

---

Next: [SSL/TLS Setup →](ssl.md)
