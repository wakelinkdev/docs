[🇬🇧 English](api.md) | [🇷🇺 Русский](api_RU.md)

# Справочник REST API

Полная документация API для сервера WakeLink.

## Базовый URL

| Среда | URL |
|-------|-----|
| Продакшн | `https://wakelink-project.org/api` |
| Self-hosted | URL вашего сервера |

## Аутентификация

Все эндпоинты (кроме входа/регистрации) требуют аутентификации.

### Получение токена

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "your-password"
}
```

Ответ:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 86400
}
```

### Использование токена

Включайте во все запросы:
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Обновление токена

```http
POST /api/v1/auth/refresh
Authorization: Bearer <current-token>
```

---

## Эндпоинты

### Аутентификация

#### Регистрация пользователя

```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123",
  "name": "John Doe"
}
```

Ответ `201 Created`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "name": "John Doe",
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### Вход

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

Ответ `200 OK`:
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

#### Получить текущего пользователя

```http
GET /api/v1/auth/me
Authorization: Bearer <token>
```

Ответ `200 OK`:
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

### Устройства

#### Список устройств

```http
GET /api/v1/devices
Authorization: Bearer <token>
```

Параметры запроса:

| Параметр | Тип | Описание |
|----------|-----|----------|
| `status` | string | Фильтр по статусу: `online`, `offline` |
| `limit` | int | Максимум результатов (по умолчанию 50) |
| `offset` | int | Смещение для пагинации |

Ответ `200 OK`:
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

#### Получить устройство

```http
GET /api/v1/devices/{device_id}
Authorization: Bearer <token>
```

Ответ `200 OK`:
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

#### Создать устройство

```http
POST /api/v1/devices
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "office-pc",
  "mac_address": "AA:BB:CC:DD:EE:FF"
}
```

Ответ `201 Created`:
```json
{
  "id": "d7a8f9b0-1234-5678-9abc-def012345678",
  "name": "office-pc",
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "token": "device-token-for-esp-configuration",
  "created_at": "2024-01-15T10:30:00Z"
}
```

!!! warning "Токен"
    Токен устройства возвращается только один раз при создании. Сохраните его в надёжном месте для настройки ESP.

#### Обновить устройство

```http
PATCH /api/v1/devices/{device_id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "new-name",
  "mac_address": "BB:CC:DD:EE:FF:00"
}
```

Ответ `200 OK`:
```json
{
  "id": "d7a8f9b0-1234-5678-9abc-def012345678",
  "name": "new-name",
  "mac_address": "BB:CC:DD:EE:FF:00",
  "status": "online",
  "last_seen": "2024-01-15T10:30:00Z"
}
```

#### Удалить устройство

```http
DELETE /api/v1/devices/{device_id}
Authorization: Bearer <token>
```

Ответ `204 No Content`

#### Перегенерировать токен устройства

```http
POST /api/v1/devices/{device_id}/token
Authorization: Bearer <token>
```

Ответ `200 OK`:
```json
{
  "token": "new-device-token"
}
```

---

### Команды пробуждения

#### Отправить команду пробуждения

```http
POST /api/v1/devices/{device_id}/wake
Authorization: Bearer <token>
Content-Type: application/json

{
  "target": "main-pc"
}
```

Поле `target` необязательно, если у устройства только одна цель.

Ответ `200 OK`:
```json
{
  "request_id": "req-550e8400-e29b-41d4-a716-446655440000",
  "status": "sent",
  "device": "office-pc",
  "target": "main-pc",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

Ответ `202 Accepted` (в очереди):
```json
{
  "request_id": "req-550e8400-e29b-41d4-a716-446655440000",
  "status": "queued",
  "device": "office-pc",
  "message": "Device offline, command queued"
}
```

#### Статус команды пробуждения

```http
GET /api/v1/commands/{request_id}
Authorization: Bearer <token>
```

Ответ `200 OK`:
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

Значения статуса:
- `queued` — Ожидает, пока устройство выйдет в сеть
- `sent` — Отправлено устройству, ожидается подтверждение
- `acknowledged` — Устройство подтвердило выполнение
- `failed` — Выполнение не удалось
- `timeout` — Нет ответа от устройства

---

### Здоровье и мониторинг

#### Проверка работоспособности

```http
GET /health
```

Ответ `200 OK`:
```json
{
  "status": "ok"
}
```

#### Расширенная проверка

```http
GET /health/live
```

Ответ `200 OK`:
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

#### Метрики (Prometheus)

```http
GET /metrics
```

Ответ:
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",endpoint="/api/v1/devices",status="200"} 1234
...
```

---

## Ответы об ошибках

Все ошибки имеют следующий формат:

```json
{
  "error": {
    "code": "DEVICE_NOT_FOUND",
    "message": "Device with ID 'd7a8...' not found",
    "details": {}
  }
}
```

### Коды ошибок

| HTTP | Код | Описание |
|------|-----|----------|
| 400 | `VALIDATION_ERROR` | Некорректное тело запроса |
| 401 | `UNAUTHORIZED` | Отсутствует или недействительный токен |
| 403 | `FORBIDDEN` | Нет прав доступа к ресурсу |
| 404 | `DEVICE_NOT_FOUND` | Устройство не существует |
| 404 | `USER_NOT_FOUND` | Пользователь не существует |
| 408 | `TIMEOUT` | Таймаут запроса |
| 409 | `CONFLICT` | Ресурс уже существует |
| 429 | `RATE_LIMITED` | Слишком много запросов |
| 500 | `INTERNAL_ERROR` | Ошибка сервера |
| 503 | `SERVICE_UNAVAILABLE` | Сервис временно недоступен |

---

## Ограничения запросов

| Эндпоинт | Лимит |
|----------|-------|
| `POST /auth/login` | 5/минута |
| `POST /devices/*/wake` | 30/минута |
| Остальные | 60/минута |

Заголовки ограничений:
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1640000000
```

---

## Пагинация

Списочные эндпоинты поддерживают пагинацию:

```http
GET /api/v1/devices?limit=10&offset=20
```

Ответ содержит метаданные пагинации:
```json
{
  "devices": [...],
  "total": 100,
  "limit": 10,
  "offset": 20
}
```

---

## Спецификация OpenAPI

Полная спецификация OpenAPI 3.0 доступна по адресам:
- **Интерактивная**: `https://wakelink-project.org/api/docs`
- **JSON**: `https://wakelink-project.org/api/openapi.json`
- **YAML**: `https://wakelink-project.org/api/openapi.yaml`

---

## SDK

Официальные SDK:

| Язык | Пакет |
|------|-------|
| Python | `pip install wakelink` |
| JavaScript | `npm install @wakelink/sdk` |
| Go | `go get github.com/wakelinkdev/wakelink-go` |

Сторонние SDK: см. [GitHub](https://github.com/wakelink)
