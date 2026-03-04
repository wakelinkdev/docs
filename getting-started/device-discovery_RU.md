[🇬🇧 English](device-discovery.md) | [🇷🇺 Русский](device-discovery_RU.md)

# Обнаружение устройств WakeLink

Устройства WakeLink поддерживают три метода обнаружения и подключения:

## 1. mDNS (Рекомендуется для одного устройства)

После подключения к WiFi ESP32 объявляет себя как **`wakelink.local`**.

### Android
```kotlin
// Простое подключение через mDNS
val result = SetupHandler.connectViaMdns()
if (result.success) {
    // Устройство доступно по адресу wakelink.local
}
```

### CLI
```bash
wakelink-cli --host wakelink.local config show
```

### Вручную
- Откройте браузер: `http://wakelink.local:80`
- Работает в любой сети (локальная WiFi, домашняя сеть и т.д.)

**Преимущества:**
- Нулевая конфигурация
- Работает в разных сетях
- Автоматическое разрешение локального имени

**Ограничения:**
- Работает только с одним устройством в сети
- Некоторые сети могут блокировать mDNS

---

## 2. UDP Broadcast обнаружение (Рекомендуется для нескольких устройств)

Устройства рассылают широковещательные пакеты о своём присутствии каждые 10 секунд на порт **5555**.

### Формат широковещательного пакета

```json
{
  "type": "wakelink-discovery",
  "id": "WL35080814",
  "ip": "192.168.1.100",
  "mac": "AA:BB:CC:DD:EE:FF",
  "port": 80,
  "mdns": "wakelink.local",
  "firmware": "1.0"
}
```

### Android

```kotlin
// Сканирование устройств (таймаут 5 секунд)
scope.launch {
    val devices = SetupHandler.discoverDevices(timeoutMs = 5000)
    devices.forEach { device ->
        println("Найдено: ${device.displayName} на ${device.ip}:${device.port}")
    }
}
```

### CLI

```bash
wakelink-cli discover --timeout 5s
# Вывод:
# WakeLink-Setup    192.168.1.100   AA:BB:CC:DD:EE:FF  port 80
# wakelink          192.168.1.101   AA:BB:CC:DD:EE:01  port 80
```

### Python

```python
import socket
import json
import time

def discover_devices(timeout=5):
    """Scan for WakeLink devices via UDP broadcast."""
    devices = []
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind(('0.0.0.0', 5555))
    sock.settimeout(timeout)
    
    start = time.time()
    try:
        while time.time() - start < timeout:
            try:
                data, addr = sock.recvfrom(512)
                device = json.loads(data.decode())
                
                # Avoid duplicates
                if not any(d['id'] == device['id'] for d in devices):
                    devices.append(device)
            except socket.timeout:
                break
    finally:
        sock.close()
    
    return devices

# Usage
for dev in discover_devices():
    print(f"{dev['id']} at {dev['ip']}")
```

**Преимущества:**
- Автоматически находит все устройства в сети
- Работает с несколькими устройствами
- Кроссплатформенный

**Ограничения:**
- Некоторые сети/роутеры блокируют UDP-широковещание
- Работает только в локальной сети (без интернета)

---

## 3. Cloud Relay (Для удалённого/облачного доступа)

После подключения устройства к WiFi и облачному серверу:

```
Мобильное приложение → Облачный сервер ← Устройство (через WSS или HTTPS)
```

### Android

```kotlin
val cloudHandler = CloudHandler(cloudUrl = "https://relay.wakelink.io")
val devices = cloudHandler.listMyDevices()  // Возвращает все зарегистрированные устройства
val device = devices.first()
val client = cloudHandler.connectToDevice(device.id)
```

### API

```bash
GET /api/v1/devices
# Возвращает список всех устройств пользователя с IP-адресами и портами
```

**Преимущества:**
- Работает из любой точки (за NAT, в разных сетях)
- Безопасный облачный relay
- Управление несколькими устройствами

**Ограничения:**
- Требует интернет-соединения
- Необходима облачная учётная запись
- Возможная задержка

---

## Приоритет подключения

Рекомендуемый порядок подключения:

1. **mDNS** — сначала попробуйте `wakelink.local` (быстро, локально)
2. **UDP Discovery** — сканирование устройств (находит все в сети)
3. **AP Mode** — если не подключено, устройство в режиме точки доступа по `192.168.4.1`
4. **Cloud** — для удалённого доступа

---

## Пример реализации: SetupScreen

```kotlin
val setupScope = rememberCoroutineScope()

// Сначала попробуем mDNS
LaunchedEffect(Unit) {
    setupScope.launch {
        val result = SetupHandler.connectViaMdns()
        if (result.success) {
            // Устройство найдено через mDNS
            isConnected = true
            apIp = "wakelink.local"  // Используем mDNS-имя
        } else {
            // Переходим к UDP-обнаружению
            val devices = SetupHandler.discoverDevices(timeoutMs = 5000)
            if (devices.isNotEmpty()) {
                // Подключаемся к первому найденному устройству
                apIp = devices.first().ip
                isConnected = true
            } else {
                // Устройство не найдено, вероятно, в режиме точки доступа
                apIp = SetupHandler.DEFAULT_AP_IP
            }
        }
    }
}
```

---

## Конфигурация

### Прошивка ESP32

```cpp
// Включить mDNS-обнаружение
MDNS.begin("wakelink");
MDNS.addService("wakelink", "tcp", 80);

// UDP-широковещание происходит автоматически каждые 10 секунд
// Порт: 5555 (настраивается в коде)
```

### Требования к роутеру

- **mDNS**: Большинство роутеров поддерживают (включено по умолчанию)
- **UDP Broadcast**: Некоторые корпоративные сети блокируют
- **Cloud Relay**: Требует доступ в интернет

---

## Устранение неполадок

### mDNS не работает
- Проверьте, что роутер поддерживает mDNS (большинство поддерживают)
- Убедитесь, что устройство подключено к той же WiFi-сети
- Попробуйте указать IP вручную или используйте UDP-обнаружение

### UDP Broadcast не находит устройства
- Проверьте, что брандмауэр/роутер разрешает UDP-порт 5555
- Попробуйте подключиться к `192.168.4.1` (режим точки доступа)
- Убедитесь, что устройство подключено к WiFi

### Cloud Relay медленный
- Проверьте интернет-соединение
- Проверьте статус облачного сервера
- Попробуйте сначала локальное подключение

---

## Смотрите также

- [Конфигурация устройства](./configuration.md)
- [Справочник веб-API](../reference/api.md)
- [Руководство по настройке Android](../clients/android.md)
