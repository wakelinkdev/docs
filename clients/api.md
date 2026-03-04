# REST API Reference

Complete API documentation for WakeLink server.

## Base URL

| Environment | URL |
|-------------|-----|
| Production | `https://wakelink-project.org/api` |
| Self-hosted | Your server URL |

## Authentication

All endpoints (except login/register) require authentication.

### Get Token

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "your-password"
}
```

Response:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 86400
}
```

### Using Token

Include in all requests:
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Refresh Token

```http
POST /api/v1/auth/refresh
Authorization: Bearer <current-token>
```

---

## Endpoints

### Authentication

#### Register User

```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123",
  "name": "John Doe"
}
```

Response `201 Created`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "name": "John Doe",
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### Login

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

Response `200 OK`:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 86400,
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

#### Get Current User

```http
GET /api/v1/auth/me
Authorization: Bearer <token>
```

Response `200 OK`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "name": "John Doe",
  "created_at": "2024-01-15T10:30:00Z",
  "devices_count": 3
}
```

---

### Devices

#### List Devices

```http
GET /api/v1/devices
Authorization: Bearer <token>
```

Query parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by status: `online`, `offline` |
| `limit` | int | Max results (default 50) |
| `offset` | int | Pagination offset |

Response `200 OK`:
```json
{
  "devices": [
    {
      "id": "d7a8f9b0-1234-5678-9abc-def012345678",
      "name": "office-pc",
      "mac_address": "AA:BB:CC:DD:EE:FF",
      "status": "online",
      "last_seen": "2024-01-15T10:30:00Z",
      "firmware_version": "1.0.0",
      "created_at": "2024-01-10T08:00:00Z"
    }
  ],
  "total": 1,
  "limit": 50,
  "offset": 0
}
```

#### Get Device

```http
GET /api/v1/devices/{device_id}
Authorization: Bearer <token>
```

Response `200 OK`:
```json
{
  "id": "d7a8f9b0-1234-5678-9abc-def012345678",
  "name": "office-pc",
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "status": "online",
  "last_seen": "2024-01-15T10:30:00Z",
  "firmware_version": "1.0.0",
  "ip_address": "192.168.1.100",
  "wifi_signal": -45,
  "uptime_seconds": 86400,
  "created_at": "2024-01-10T08:00:00Z",
  "targets": [
    {
      "name": "main-pc",
      "mac": "AA:BB:CC:DD:EE:FF"
    }
  ]
}
```

#### Create Device

```http
POST /api/v1/devices
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "office-pc",
  "mac_address": "AA:BB:CC:DD:EE:FF"
}
```

Response `201 Created`:
```json
{
  "id": "d7a8f9b0-1234-5678-9abc-def012345678",
  "name": "office-pc",
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "token": "device-token-for-esp-configuration",
  "created_at": "2024-01-15T10:30:00Z"
}
```

!!! warning "Token"
    The device token is only returned once during creation. Store it securely for ESP configuration.

#### Update Device

```http
PATCH /api/v1/devices/{device_id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "new-name",
  "mac_address": "BB:CC:DD:EE:FF:00"
}
```

Response `200 OK`:
```json
{
  "id": "d7a8f9b0-1234-5678-9abc-def012345678",
  "name": "new-name",
  "mac_address": "BB:CC:DD:EE:FF:00",
  "status": "online",
  "last_seen": "2024-01-15T10:30:00Z"
}
```

#### Delete Device

```http
DELETE /api/v1/devices/{device_id}
Authorization: Bearer <token>
```

Response `204 No Content`

#### Regenerate Device Token

```http
POST /api/v1/devices/{device_id}/token
Authorization: Bearer <token>
```

Response `200 OK`:
```json
{
  "token": "new-device-token"
}
```

---

### Wake Commands

#### Send Wake Command

```http
POST /api/v1/devices/{device_id}/wake
Authorization: Bearer <token>
Content-Type: application/json

{
  "target": "main-pc"
}
```

The `target` field is optional if the device has only one target.

Response `200 OK`:
```json
{
  "request_id": "req-550e8400-e29b-41d4-a716-446655440000",
  "status": "sent",
  "device": "office-pc",
  "target": "main-pc",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

Response `202 Accepted` (queued):
```json
{
  "request_id": "req-550e8400-e29b-41d4-a716-446655440000",
  "status": "queued",
  "device": "office-pc",
  "message": "Device offline, command queued"
}
```

#### Wake Command Status

```http
GET /api/v1/commands/{request_id}
Authorization: Bearer <token>
```

Response `200 OK`:
```json
{
  "request_id": "req-550e8400-e29b-41d4-a716-446655440000",
  "status": "acknowledged",
  "device": "office-pc",
  "target": "main-pc",
  "sent_at": "2024-01-15T10:30:00Z",
  "acknowledged_at": "2024-01-15T10:30:01Z",
  "latency_ms": 234
}
```

Status values:
- `queued` — Waiting for device to come online
- `sent` — Sent to device, awaiting acknowledgment
- `acknowledged` — Device confirmed execution
- `failed` — Execution failed
- `timeout` — No response from device

---

### Health & Monitoring

#### Health Check

```http
GET /health
```

Response `200 OK`:
```json
{
  "status": "ok"
}
```

#### Detailed Health

```http
GET /health/live
```

Response `200 OK`:
```json
{
  "status": "ok",
  "version": "1.0.0",
  "uptime_seconds": 86400,
  "checks": {
    "database": {"status": "ok", "latency_ms": 2},
    "redis": {"status": "ok", "latency_ms": 1}
  }
}
```

#### Metrics (Prometheus)

```http
GET /metrics
```

Response:
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",endpoint="/api/v1/devices",status="200"} 1234
...
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": {
    "code": "DEVICE_NOT_FOUND",
    "message": "Device with ID 'd7a8...' not found",
    "details": {}
  }
}
```

### Error Codes

| HTTP | Code | Description |
|------|------|-------------|
| 400 | `VALIDATION_ERROR` | Invalid request body |
| 401 | `UNAUTHORIZED` | Missing or invalid token |
| 403 | `FORBIDDEN` | Not authorized for this resource |
| 404 | `DEVICE_NOT_FOUND` | Device doesn't exist |
| 404 | `USER_NOT_FOUND` | User doesn't exist |
| 408 | `TIMEOUT` | Request timed out |
| 409 | `CONFLICT` | Resource already exists |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |
| 503 | `SERVICE_UNAVAILABLE` | Service temporarily unavailable |

---

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| `POST /auth/login` | 5/minute |
| `POST /devices/*/wake` | 30/minute |
| Other | 60/minute |

Rate limit headers:
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1640000000
```

---

## Pagination

List endpoints support pagination:

```http
GET /api/v1/devices?limit=10&offset=20
```

Response includes pagination metadata:
```json
{
  "devices": [...],
  "total": 100,
  "limit": 10,
  "offset": 20
}
```

---

## OpenAPI Specification

Full OpenAPI 3.0 spec available at:
- **Interactive**: `https://wakelink-project.org/api/docs`
- **JSON**: `https://wakelink-project.org/api/openapi.json`
- **YAML**: `https://wakelink-project.org/api/openapi.yaml`

---

## SDKs

Official SDKs:

| Language | Package |
|----------|---------|
| Python | `pip install wakelink` |
| JavaScript | `npm install @wakelink/sdk` |
| Go | `go get github.com/wakelinkdev/wakelink-go` |

Community SDKs: See [GitHub](https://github.com/wakelink)
