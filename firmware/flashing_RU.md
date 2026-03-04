[🇬🇧 English](flashing.md) | [🇷🇺 Русский](flashing_RU.md)

# Руководство по прошивке

Установка прошивки WakeLink на плату ESP8266 или ESP32.

## Метод 1: Веб-прошивальщик (рекомендуется)

Самый простой способ прошивки — не требует установки программного обеспечения.

### Требования
- Браузер Chrome, Edge или Opera
- USB-кабель
- Плата ESP

### Шаги

1. **Подключите плату ESP** через USB
2. **Откройте** [flash.wakelink.io](https://flash.wakelink.io)
3. **Нажмите «Connect»** и выберите COM-порт
4. **Выберите плату** (ESP8266 или ESP32)
5. **Нажмите «Install»**
6. **Дождитесь** завершения (~60 секунд)

!!! success "Готово!"
    Когда появится сообщение «Installation complete», отключите и снова подключите плату.

### Устранение неполадок веб-прошивальщика

| Проблема | Решение |
|----------|---------|
| COM-порт не отображается | Установите [драйверы CP2102](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) или [драйверы CH340](https://sparks.gogo.co.nz/ch340.html) |
| «Failed to connect» | Удерживайте кнопку BOOT при нажатии «Connect» |
| Браузер не поддерживается | Используйте Chrome, Edge или Opera |

---

## Метод 2: Arduino IDE

Для пользователей, желающих настраивать прошивку или вести разработку.

### 1. Установите Arduino IDE

Скачайте [Arduino IDE 2.x](https://www.arduino.cc/en/software)

### 2. Добавьте поддержку плат ESP

=== "ESP8266"
    1. Перейдите в **File → Preferences**
    2. Добавьте в «Additional Board Manager URLs»:
       ```
       https://arduino.esp8266.com/stable/package_esp8266com_index.json
       ```
    3. Перейдите в **Tools → Board → Boards Manager**
    4. Найдите «esp8266» и установите **ESP8266 by ESP8266 Community**

=== "ESP32"
    1. Перейдите в **File → Preferences**
    2. Добавьте в «Additional Board Manager URLs»:
       ```
       https://espressif.github.io/arduino-esp32/package_esp32_index.json
       ```
    3. Перейдите в **Tools → Board → Boards Manager**
    4. Найдите «esp32» и установите **ESP32 by Espressif Systems**

### 3. Установите необходимые библиотеки

Перейдите в **Tools → Manage Libraries** и установите:

| Библиотека | Версия | Автор |
|------------|--------|-------|
| WebSocketsClient | 2.4.x | Markus Sattler |
| ArduinoJson | 6.x | Benoit Blanchon |
| Crypto | 0.4.x | Rhys Weatherley (только ESP8266) |

### 4. Скачайте прошивку

```bash
git clone https://github.com/wakelinkdev/wakelink-firmware
```

Или скачайте ZIP из [releases](https://github.com/wakelinkdev/wakelink-firmware/releases)

### 5. Откройте проект

1. Откройте Arduino IDE
2. **File → Open**
3. Перейдите к `wakelink-firmware/WakeLink/WakeLink.ino`

### 6. Выберите плату и порт

=== "ESP8266"
    - **Tools → Board**: `NodeMCU 1.0 (ESP-12E Module)` или `Generic ESP8266 Module`
    - **Tools → Flash Size**: `4MB (FS:1MB OTA:~1019KB)`
    - **Tools → Port**: Ваш COM-порт

=== "ESP32"
    - **Tools → Board**: `ESP32 Dev Module`
    - **Tools → Partition Scheme**: `Default 4MB with spiffs`
    - **Tools → Port**: Ваш COM-порт

### 7. Загрузите прошивку

1. Нажмите **Upload** (кнопка →)
2. Дождитесь «Done uploading»

!!! tip "Режим загрузки ESP8266"
    Если загрузка завершается ошибкой, удерживайте кнопку **FLASH** на плате во время начала загрузки.

---

## Метод 3: PlatformIO (расширенный)

Для профессионального рабочего процесса разработки.

### 1. Установите PlatformIO

1. Установите [VS Code](https://code.visualstudio.com/)
2. Установите [расширение PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)

### 2. Клонируйте репозиторий

```bash
git clone https://github.com/wakelinkdev/wakelink-firmware
cd wakelink-firmware
```

### 3. Соберите и загрузите прошивку

=== "ESP8266"
    ```bash
    # Только сборка
    pio run -e esp8266

    # Сборка и загрузка
    pio run -e esp8266 -t upload

    # Загрузка и мониторинг serial
    pio run -e esp8266 -t upload -t monitor
    ```

=== "ESP32"
    ```bash
    # Только сборка
    pio run -e esp32

    # Сборка и загрузка
    pio run -e esp32 -t upload

    # Загрузка и мониторинг serial
    pio run -e esp32 -t upload -t monitor
    ```

### Конфигурация PlatformIO

Файл `platformio.ini` определяет среды сборки:

```ini
[env:esp8266]
platform = espressif8266
board = nodemcuv2
framework = arduino
monitor_speed = 115200
lib_deps =
    links2004/WebSockets @ ^2.4.0
    bblanchon/ArduinoJson @ ^6.21.0
    rweather/Crypto @ ^0.4.0

[env:esp32]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
lib_deps =
    links2004/WebSockets @ ^2.4.0
    bblanchon/ArduinoJson @ ^6.21.0
```

---

## Метод 4: esptool (командная строка)

Для предварительно собранных бинарных файлов без IDE.

### 1. Установите esptool

```bash
pip install esptool
```

### 2. Скачайте бинарный файл

Получите готовый `.bin` из [releases](https://github.com/wakelinkdev/wakelink-firmware/releases)

### 3. Прошейте устройство

=== "ESP8266"
    ```bash
    esptool.py --port COM3 --baud 460800 write_flash \
      0x00000 wakelink-esp8266.bin
    ```

=== "ESP32"
    ```bash
    esptool.py --port COM3 --baud 460800 write_flash \
      -z 0x1000 wakelink-esp32.bin
    ```

### 4. Очистка Flash (при необходимости)

```bash
esptool.py --port COM3 erase_flash
```

---

## Проверка установки

После прошивки:

1. **Отключите и снова подключите** USB-кабель
2. **Откройте Serial Monitor** (скорость 115200)
3. **Нажмите кнопку Reset** на плате

Вы должны увидеть:

```
WakeLink v1.0.0 starting...
[CONFIG] No saved config, entering provisioning mode
[WIFI] Starting AP: WakeLink-Setup
[PROV] Captive portal at 192.168.4.1
```

### Serial Monitor

=== "Arduino IDE"
    **Tools → Serial Monitor** (установите скорость 115200)

=== "PlatformIO"
    ```bash
    pio device monitor -b 115200
    ```

=== "putty/screen"
    ```bash
    # Linux/macOS
    screen /dev/ttyUSB0 115200

    # Windows (используйте PuTTY)
    # COM-порт, скорость: 115200
    ```

---

## Следующие шаги

[Настройка устройства →](configuration_RU.md)
