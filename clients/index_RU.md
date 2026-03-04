[🇬🇧 English](index.md) | [🇷🇺 Русский](index_RU.md)

# Обзор клиентов

Несколько способов взаимодействия с WakeLink.

## Доступные клиенты

| Клиент | Платформа | Сценарий использования |
|--------|-----------|------------------------|
| [Python CLI](cli.md) | Linux, macOS, Windows | Опытные пользователи, автоматизация, скрипты |
| [Android App](android.md) | Android 8+ | Мобильные пользователи, быстрый доступ |
| [REST API](api.md) | Любая | Интеграция, собственные клиенты |

## Сравнение возможностей

| Возможность | CLI | Android | API |
|-------------|-----|---------|-----|
| Пробуждение устройств | ✅ | ✅ | ✅ |
| Управление устройствами | ✅ | ✅ | ✅ |
| Мониторинг состояния | ✅ | ✅ | ✅ |
| Push-уведомления | ❌ | ✅ | ❌ |
| Виджет на главном экране | ❌ | ✅ | ❌ |
| Пакетные операции | ✅ | ❌ | ✅ |
| Скриптинг | ✅ | ❌ | ✅ |
| Автономный режим | ❌ | Частично | ❌ |

## Быстрые ссылки

<div class="grid cards" markdown>

-   :material-console:{ .lg .middle } __Python CLI__

    ---

    Полнофункциональный интерфейс командной строки

    [:octicons-arrow-right-24: Руководство по CLI](cli.md)

-   :material-android:{ .lg .middle } __Android App__

    ---

    Пробуждайте устройства с телефона

    [:octicons-arrow-right-24: Руководство по Android](android.md)

-   :material-api:{ .lg .middle } __REST API__

    ---

    Создавайте собственные интеграции

    [:octicons-arrow-right-24: Справочник API](api.md)

</div>

---

## Аутентификация

Все клиенты используют одинаковый механизм аутентификации:

1. **Вход пользователя**: Email + пароль → JWT-токен
2. **API-запросы**: Передавайте JWT в заголовке `Authorization: Bearer <token>`
3. **Обновление токена**: Токены истекают через 24 часа, обновляются автоматически

### Получение API-токена

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

    Ответ:
    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIs...",
      "token_type": "bearer",
      "expires_in": 86400
    }
    ```

---

## Идентификация устройств

На устройства можно ссылаться по:

| Метод | Пример | Примечания |
|-------|--------|------------|
| Имя | `office-pc` | Понятное человеку, должно быть уникальным для пользователя |
| ID | `d7a8f9b0-...` | UUID, глобально уникальный |
| MAC | `AA:BB:CC:DD:EE:FF` | Аппаратный адрес |

```bash
# All equivalent
wakelink wake office-pc
wakelink wake d7a8f9b0-1234-5678-9abc-def012345678
wakelink wake --mac AA:BB:CC:DD:EE:FF
```

---

## Основные операции

### Пробудить устройство

=== "CLI"
    ```bash
    wakelink wake my-device
    ```

=== "Android"
    Нажмите на устройство в списке → Нажмите «Wake»

=== "API"
    ```bash
    curl -X POST https://wakelink-project.org/api/v1/devices/my-device/wake \
      -H "Authorization: Bearer $TOKEN"
    ```

### Список устройств

=== "CLI"
    ```bash
    wakelink list
    ```

=== "API"
    ```bash
    curl https://wakelink-project.org/api/v1/devices \
      -H "Authorization: Bearer $TOKEN"
    ```

### Проверить состояние

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

## Настройка для собственного сервера

Укажите клиентам адрес вашего сервера:

=== "CLI"
    ```yaml
    # ~/.wakelink/config.yaml
    server:
      api_url: https://api.your-server.com
      ws_url: wss://ws.your-server.com
    ```

    Или через переменную среды:
    ```bash
    export WAKELINK_API_URL=https://api.your-server.com
    ```

=== "Android"
    Настройки → Сервер → Собственный сервер → Введите URL

=== "API"
    Замените `wakelink-project.org/api` на URL вашего сервера

---

## Обработка ошибок

Типичные ответы с ошибками:

| Код | Значение | Действие |
|-----|----------|----------|
| 401 | Не авторизован | Войдите повторно, обновите токен |
| 403 | Доступ запрещён | Проверьте права на устройство |
| 404 | Не найдено | Проверьте имя/ID устройства |
| 408 | Таймаут | Устройство офлайн, повторите попытку |
| 429 | Превышен лимит запросов | Уменьшите частоту запросов |
| 500 | Ошибка сервера | Обратитесь в поддержку |

---

## Ограничения запросов

Стандартные лимиты:

| Эндпоинт | Лимит |
|----------|-------|
| Вход | 5/минута |
| Команды пробуждения | 30/минута |
| Прочие API-запросы | 60/минута |

Заголовки ограничений запросов:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1640000000
```
