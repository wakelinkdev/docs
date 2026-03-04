[🇬🇧 English](cli.md) | [🇷🇺 Русский](cli_RU.md)

# Python CLI

Полнофункциональный интерфейс командной строки для WakeLink.

## Установка

```bash
pip install wakelink-cli
```

Или из исходного кода:
```bash
git clone https://github.com/wakelinkdev/wakelink-client
cd wakelink-client
pip install -e .
```

### Требования

- Python 3.9+
- pip

## Быстрый старт

```bash
# Login
wakelink login

# List devices
wakelink list

# Wake a device
wakelink wake office-pc

# Check status
wakelink status office-pc
```

---

## Справочник команд

### Аутентификация

```bash
# Interactive login
wakelink login
# Enter email and password when prompted

# Login with arguments
wakelink login --email user@example.com --password mypassword

# Logout
wakelink logout

# Show current user
wakelink whoami
```

### Управление устройствами

```bash
# List all devices
wakelink list
wakelink list --format json
wakelink list --verbose

# Register new device
wakelink register --name my-device
wakelink register --name my-device --mac AA:BB:CC:DD:EE:FF

# Show device details
wakelink show my-device

# Update device
wakelink update my-device --name new-name
wakelink update my-device --mac BB:CC:DD:EE:FF:00

# Delete device
wakelink delete my-device
wakelink delete my-device --force
```

### Операции пробуждения

```bash
# Wake device
wakelink wake my-device

# Wake with timeout
wakelink wake my-device --timeout 30

# Wake multiple devices
wakelink wake device1 device2 device3

# Wake all devices
wakelink wake --all

# Verbose output
wakelink wake my-device -v
```

### Мониторинг

```bash
# Device status
wakelink status my-device

# Real-time logs
wakelink logs my-device
wakelink logs my-device --follow

# Ping device (check connectivity)
wakelink ping my-device
wakelink ping my-device --count 5
```

### Конфигурация

```bash
# Show config
wakelink config show

# Set config value
wakelink config set server.api_url https://api.custom.com

# Reset to defaults
wakelink config reset
```

### Утилиты

```bash
# Version
wakelink version

# Help
wakelink --help
wakelink wake --help
```

---

## Конфигурация

### Файл конфигурации

Расположен по пути `~/.wakelink/config.yaml`:

```yaml
# Server settings
server:
  api_url: https://wakelink-project.org/api
  ws_url: wss://relay.wakelink.io
  timeout: 30

# Authentication
auth:
  token: <your-jwt-token>
  refresh_token: <refresh-token>

# Output preferences
output:
  format: text  # text, json, table
  color: true
  verbose: false

# Default behavior
defaults:
  timeout: 30
  retries: 3
```

### Переменные среды

| Переменная | Описание |
|------------|----------|
| `WAKELINK_API_URL` | URL API-сервера |
| `WAKELINK_TOKEN` | Токен аутентификации |
| `WAKELINK_CONFIG` | Путь к файлу конфигурации |
| `WAKELINK_NO_COLOR` | Отключить цвета |
| `WAKELINK_DEBUG` | Включить отладочный вывод |

---

## Форматы вывода

### Текстовый (по умолчанию)

```bash
$ wakelink list
NAME         STATUS    LAST SEEN
office-pc    online    2 minutes ago
home-server  offline   3 hours ago
```

### JSON

```bash
$ wakelink list --format json
[
  {
    "id": "d7a8f9b0-...",
    "name": "office-pc",
    "status": "online",
    "last_seen": "2024-01-15T10:30:00Z"
  }
]
```

### Таблица

```bash
$ wakelink list --format table
┌─────────────┬─────────┬─────────────────────┐
│ Name        │ Status  │ Last Seen           │
├─────────────┼─────────┼─────────────────────┤
│ office-pc   │ online  │ 2024-01-15 10:30:00 │
│ home-server │ offline │ 2024-01-15 07:30:00 │
└─────────────┴─────────┴─────────────────────┘
```

---

## Скриптинг

### Коды завершения

| Код | Значение |
|-----|----------|
| 0 | Успех |
| 1 | Общая ошибка |
| 2 | Ошибка аутентификации |
| 3 | Устройство не найдено |
| 4 | Устройство офлайн |
| 5 | Таймаут |

### Примеры

```bash
#!/bin/bash
# Wake office PC if it's offline

status=$(wakelink status office-pc --format json | jq -r '.status')

if [ "$status" = "offline" ]; then
    wakelink wake office-pc
    echo "Wake command sent"
else
    echo "Device already online"
fi
```

```bash
#!/bin/bash
# Wake all devices and check results

for device in $(wakelink list --format json | jq -r '.[].name'); do
    if wakelink wake "$device" --timeout 10; then
        echo "✅ $device woke up"
    else
        echo "❌ $device failed"
    fi
done
```

### Интеграция с Python

```python
import subprocess
import json

def wake_device(name: str) -> bool:
    result = subprocess.run(
        ["wakelink", "wake", name, "--format", "json"],
        capture_output=True,
        text=True
    )
    return result.returncode == 0

def list_devices() -> list:
    result = subprocess.run(
        ["wakelink", "list", "--format", "json"],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)
```

---

## Продвинутое использование

### Пакетные операции

```bash
# Wake devices from file
cat devices.txt | xargs -I {} wakelink wake {}

# Parallel wake
cat devices.txt | xargs -P 5 -I {} wakelink wake {}
```

### Задания Cron

```bash
# Wake office PC at 8 AM on weekdays
0 8 * * 1-5 /usr/local/bin/wakelink wake office-pc >> /var/log/wakelink.log 2>&1
```

### Псевдонимы

Добавьте в `~/.bashrc` или `~/.zshrc`:

```bash
alias wl='wakelink'
alias wlw='wakelink wake'
alias wls='wakelink status'
alias wll='wakelink list'
```

---

## Устранение неполадок

### «Authentication failed» (Ошибка аутентификации)

```bash
# Re-login
wakelink logout
wakelink login
```

### «Connection timeout» (Таймаут соединения)

```bash
# Check server status
curl https://wakelink-project.org/api/health

# Increase timeout
wakelink wake my-device --timeout 60
```

### «Device not found» (Устройство не найдено)

```bash
# List available devices
wakelink list

# Check exact name
wakelink show my-device
```

### Режим отладки

```bash
WAKELINK_DEBUG=1 wakelink wake my-device
```

---

## Удаление

```bash
pip uninstall wakelink-cli

# Remove config
rm -rf ~/.wakelink
```
