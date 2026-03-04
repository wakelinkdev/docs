[🇬🇧 English](configuration.md) | [🇷🇺 Русский](configuration_RU.md)

# Справочник по конфигурации

Полный список параметров конфигурации для всех компонентов WakeLink.

## Конфигурация прошивки

### Настройки устройства

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `device.name` | string | — | Отображаемое имя устройства (обязательно) |
| `device.token` | string | — | Токен аутентификации (обязательно) |

### Настройки WiFi

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `wifi.ssid` | string | — | Имя WiFi-сети (обязательно) |
| `wifi.password` | string | — | Пароль WiFi (обязательно) |
| `wifi.static_ip` | string | — | Статический IP-адрес |
| `wifi.gateway` | string | — | Шлюз по умолчанию |
| `wifi.subnet` | string | — | Маска подсети |
| `wifi.dns` | string | — | DNS-сервер |

### Настройки сервера

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `server.url` | string | `wss://relay.wakelink.io` | URL WebSocket-сервера |
| `server.port` | int | 443 | Порт сервера |
| `server.tls` | bool | true | Использовать TLS |
| `server.ping_interval` | int | 30000 | Интервал пинга (мс) |

### Настройки цели

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `target.mac` | string | — | MAC-адрес цели (обязательно) |
| `target.broadcast` | string | `255.255.255.255` | Широковещательный адрес |
| `target.port` | int | 9 | WoL-порт |

### Локальный режим

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `local.enabled` | bool | false | Включить локальный HTTP-сервер |
| `local.port` | int | 80 | Порт локального сервера |
| `local.auth_required` | bool | true | Требовать аутентификацию |
| `local.auth_token` | string | — | Токен локальной аутентификации |

### Управление питанием

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `power.deep_sleep` | bool | false | Включить глубокий сон |
| `power.sleep_interval` | int | 300 | Продолжительность сна (секунды) |

### Функции

| Параметр | Тип | По умолчанию | Описание |
|---------|------|---------|-------------|
| `features.led_enabled` | bool | true | Статусный LED |
| `features.ota_enabled` | bool | true | OTA-обновления |
| `features.debug_mode` | bool | false | Отладочное логирование |

---

## Конфигурация сервера

### Основные настройки

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `SERVER_HOST` | string | `0.0.0.0` | Адрес привязки |
| `SERVER_PORT` | int | 8000 | Порт API |
| `WS_PORT` | int | 8001 | Порт WebSocket |
| `DEBUG` | bool | false | Режим отладки |
| `WORKERS` | int | 4 | Количество воркеров Gunicorn |

### Безопасность

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `JWT_SECRET` | string | — | Ключ подписи токена (обязательно) |
| `JWT_ALGORITHM` | string | `HS256` | Алгоритм JWT |
| `JWT_EXPIRE_MINUTES` | int | 1440 | Срок действия токена |
| `CORS_ORIGINS` | string | `*` | Разрешённые источники |

### База данных

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `DATABASE_URL` | string | — | URL PostgreSQL (обязательно) |
| `DB_POOL_SIZE` | int | 5 | Размер пула соединений |
| `DB_MAX_OVERFLOW` | int | 10 | Максимальное переполнение |

### Redis

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `REDIS_URL` | string | — | URL Redis (обязательно) |
| `REDIS_PREFIX` | string | `wakelink:` | Префикс ключей |

### Логирование

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `LOG_LEVEL` | string | `INFO` | Уровень логирования |
| `LOG_FORMAT` | string | `json` | Формат (json/text) |
| `ACCESS_LOG` | bool | true | Включить логи доступа |

### Ограничение запросов

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `RATE_LIMIT_ENABLED` | bool | true | Включить ограничение запросов |
| `RATE_LIMIT_PER_MINUTE` | int | 60 | Запросов в минуту |

### Функции

| Переменная | Тип | По умолчанию | Описание |
|----------|------|---------|-------------|
| `ENABLE_REGISTRATION` | bool | true | Разрешить регистрацию |
| `ENABLE_METRICS` | bool | true | Эндпоинт Prometheus |
| `ENABLE_DOCS` | bool | true | Документация OpenAPI |

---

## Конфигурация CLI

### Файл конфигурации: `~/.wakelink/config.yaml`

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

### Переменные окружения

| Переменная | Описание |
|----------|-------------|
| `WAKELINK_API_URL` | Переопределить URL API |
| `WAKELINK_TOKEN` | Переопределить токен аутентификации |
| `WAKELINK_CONFIG` | Путь к файлу конфигурации |
| `WAKELINK_NO_COLOR` | Отключить цвета |
| `WAKELINK_DEBUG` | Отладочный вывод |

---

## Настройки Android-приложения

### Настройки в приложении

| Параметр | По умолчанию | Описание |
|---------|---------|-------------|
| URL сервера | `https://wakelink-project.org/api` | API-сервер |
| Тема | Системная | Светлая/Тёмная/Системная |
| Уведомления | Включены | Push-уведомления |
| Фоновое обновление | 15 мин | Интервал обновления статуса |
| Тайм-аут | 30 с | Тайм-аут команды |

### Конфигурация виджета

| Параметр | По умолчанию | Описание |
|---------|---------|-------------|
| Устройство | — | Выбранное устройство(а) |
| Интервал обновления | 15 мин | Частота обновления |
| Пробуждение одним касанием | Отключено | Пробуждать при нажатии |

---

## Примеры конфигураций

### Минимальная конфигурация прошивки

```json
{
  "device": {"name": "mydevice", "token": "xxx"},
  "wifi": {"ssid": "MyWiFi", "password": "mypass"},
  "server": {"url": "wss://relay.wakelink.io"},
  "target": {"mac": "AA:BB:CC:DD:EE:FF"}
}
```

### Конфигурация сервера для продакшена

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

### Конфигурация сервера для разработки

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
