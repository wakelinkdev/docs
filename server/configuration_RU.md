[🇬🇧 English](configuration.md) | [🇷🇺 Русский](configuration_RU.md)

# Конфигурация сервера

Полный справочник по параметрам конфигурации сервера WakeLink.

## Методы конфигурации

1. **Переменные окружения** (рекомендуется для контейнеров)
2. **Файл `.env`** (для разработки)
3. **`config.yaml`** (расширенная конфигурация)

Переменные окружения имеют приоритет над конфигурацией из файлов.

## Основные настройки

### Сервер

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `SERVER_HOST` | Адрес привязки | `0.0.0.0` |
| `SERVER_PORT` | Порт API | `8000` |
| `WS_HOST` | Адрес привязки WebSocket | `0.0.0.0` |
| `WS_PORT` | Порт WebSocket | `8001` |
| `WORKERS` | Рабочие процессы Gunicorn | `4` |
| `DEBUG` | Режим отладки | `false` |

### Безопасность

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `JWT_SECRET` | Ключ подписи токена (ОБЯЗАТЕЛЬНО) | - |
| `JWT_ALGORITHM` | Алгоритм JWT | `HS256` |
| `JWT_EXPIRE_MINUTES` | Срок действия токена | `1440` (24ч) |
| `CORS_ORIGINS` | Разрешённые источники (через запятую) | `*` |
| `TRUSTED_HOSTS` | Разрешённые заголовки Host | `*` |

!!! danger "JWT_SECRET"
    **НИКОГДА** не используйте значение по умолчанию или слабый секрет в production.
    
    Сгенерируйте надёжный секрет:
    ```bash
    openssl rand -hex 32
    ```

### База данных

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `DATABASE_URL` | Строка подключения PostgreSQL | - |
| `DB_POOL_SIZE` | Размер пула соединений | `5` |
| `DB_MAX_OVERFLOW` | Максимальное количество дополнительных соединений | `10` |
| `DB_ECHO` | Логировать SQL-запросы | `false` |

Формат URL базы данных:
```
postgresql://user:password@host:port/database
postgresql://wakelink:secret@db:5432/wakelink
```

### Redis

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `REDIS_URL` | Строка подключения Redis | - |
| `REDIS_PREFIX` | Префикс ключей | `wakelink:` |
| `REDIS_TTL` | TTL по умолчанию (секунды) | `3600` |

Формат URL Redis:
```
redis://host:port/db
redis://:password@host:port/db
redis://redis:6379/0
```

### Логирование

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LOG_LEVEL` | Уровень логирования | `INFO` |
| `LOG_FORMAT` | Формат (json/text) | `json` |
| `LOG_FILE` | Путь к файлу лога | - (stdout) |
| `ACCESS_LOG` | Включить логи доступа | `true` |

Уровни логирования: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`

---

## Флаги функций

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ENABLE_REGISTRATION` | Разрешить регистрацию новых пользователей | `true` |
| `ENABLE_METRICS` | Включить конечную точку /metrics | `true` |
| `ENABLE_DOCS` | Включить /docs (OpenAPI) | `true` |
| `ENABLE_ADMIN_API` | Включить административные конечные точки | `false` |
| `BETA_FEATURES` | Включить бета-функции | `false` |

---

## Ограничение запросов

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RATE_LIMIT_ENABLED` | Включить ограничение запросов | `true` |
| `RATE_LIMIT_PER_MINUTE` | Запросов в минуту | `60` |
| `RATE_LIMIT_BURST` | Допустимые всплески | `10` |
| `RATE_LIMIT_BY` | Ключ: `ip` или `user` | `user` |

### Лимиты для конкретных конечных точек

```yaml
# config.yaml
rate_limits:
  default: 60/minute
  endpoints:
    /api/v1/auth/login: 5/minute
    /api/v1/devices/*/wake: 30/minute
```

---

## Конфигурация WebSocket

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `WS_PING_INTERVAL` | Интервал пинга (секунды) | `30` |
| `WS_PING_TIMEOUT` | Таймаут пинга (секунды) | `10` |
| `WS_MAX_CONNECTIONS` | Максимум одновременных соединений | `10000` |
| `WS_MAX_MESSAGE_SIZE` | Максимальный размер сообщения (байты) | `65536` |
| `WS_COMPRESSION` | Включить сжатие | `true` |

---

## Электронная почта (необязательно)

Для уведомлений и сброса пароля:

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `SMTP_HOST` | SMTP-сервер | - |
| `SMTP_PORT` | Порт SMTP | `587` |
| `SMTP_USER` | Имя пользователя SMTP | - |
| `SMTP_PASSWORD` | Пароль SMTP | - |
| `SMTP_TLS` | Использовать TLS | `true` |
| `EMAIL_FROM` | Адрес отправителя | - |

---

## Метрики и мониторинг

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `METRICS_ENABLED` | Включить метрики Prometheus | `true` |
| `METRICS_PATH` | Конечная точка метрик | `/metrics` |
| `HEALTH_PATH` | Конечная точка проверки работоспособности | `/health` |

---

## Примеры конфигурации

### Для разработки

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

### Повышенная безопасность

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

## Конфигурационный файл (расширенная настройка)

Для сложных конфигураций используйте `config.yaml`:

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

Загрузка конфигурации:
```bash
export WAKELINK_CONFIG=/etc/wakelink/config.yaml
```

---

## Валидация

Проверка конфигурации:

```bash
docker-compose exec api python -m wakelink.cli config validate
```

Вывод:
```
✅ Database connection: OK
✅ Redis connection: OK
✅ JWT secret: Strong (64 chars)
✅ CORS origins: Configured
⚠️  Debug mode: Enabled (disable in production)
```

---

Далее: [Настройка SSL/TLS →](ssl.md)
