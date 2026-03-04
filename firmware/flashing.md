[🇬🇧 English](flashing.md) | [🇷🇺 Русский](flashing_RU.md)

# Flashing Guide

Install WakeLink firmware on your ESP8266 or ESP32 board.

## Method 1: Web Flasher (Recommended)

The easiest way to flash — no software installation required.

### Requirements
- Chrome, Edge, or Opera browser
- USB cable
- ESP board

### Steps

1. **Connect your ESP board** via USB
2. **Open** [flash.wakelink.io](https://flash.wakelink.io)
3. **Click "Connect"** and select your COM port
4. **Select your board** (ESP8266 or ESP32)
5. **Click "Install"**
6. **Wait** for completion (~60 seconds)

!!! success "Done!"
    When you see "Installation complete", disconnect and reconnect the board.

### Troubleshooting Web Flasher

| Issue | Solution |
|-------|----------|
| No COM port shown | Install [CP2102 drivers](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) or [CH340 drivers](https://sparks.gogo.co.nz/ch340.html) |
| "Failed to connect" | Hold BOOT button while clicking Connect |
| Browser not supported | Use Chrome, Edge, or Opera |

---

## Method 2: Arduino IDE

For users who want to customize or develop.

### 1. Install Arduino IDE

Download [Arduino IDE 2.x](https://www.arduino.cc/en/software)

### 2. Add ESP Board Support

=== "ESP8266"
    1. Go to **File → Preferences**
    2. Add to "Additional Board Manager URLs":
       ```
       https://arduino.esp8266.com/stable/package_esp8266com_index.json
       ```
    3. Go to **Tools → Board → Boards Manager**
    4. Search "esp8266" and install **ESP8266 by ESP8266 Community**

=== "ESP32"
    1. Go to **File → Preferences**
    2. Add to "Additional Board Manager URLs":
       ```
       https://espressif.github.io/arduino-esp32/package_esp32_index.json
       ```
    3. Go to **Tools → Board → Boards Manager**
    4. Search "esp32" and install **ESP32 by Espressif Systems**

### 3. Install Required Libraries

Go to **Tools → Manage Libraries** and install:

| Library | Version | Author |
|---------|---------|--------|
| WebSocketsClient | 2.4.x | Markus Sattler |
| ArduinoJson | 6.x | Benoit Blanchon |
| Crypto | 0.4.x | Rhys Weatherley (ESP8266 only) |

### 4. Download Firmware

```bash
git clone https://github.com/wakelinkdev/wakelink-firmware
```

Or download ZIP from [releases](https://github.com/wakelinkdev/wakelink-firmware/releases)

### 5. Open Project

1. Open Arduino IDE
2. **File → Open**
3. Navigate to `wakelink-firmware/WakeLink/WakeLink.ino`

### 6. Select Board and Port

=== "ESP8266"
    - **Tools → Board**: `NodeMCU 1.0 (ESP-12E Module)` or `Generic ESP8266 Module`
    - **Tools → Flash Size**: `4MB (FS:1MB OTA:~1019KB)`
    - **Tools → Port**: Your COM port

=== "ESP32"
    - **Tools → Board**: `ESP32 Dev Module`
    - **Tools → Partition Scheme**: `Default 4MB with spiffs`
    - **Tools → Port**: Your COM port

### 7. Upload

1. Click **Upload** (→ button)
2. Wait for "Done uploading"

!!! tip "ESP8266 Upload Mode"
    If upload fails, hold the **FLASH** button on your board while uploading starts.

---

## Method 3: PlatformIO (Advanced)

For professional development workflow.

### 1. Install PlatformIO

1. Install [VS Code](https://code.visualstudio.com/)
2. Install [PlatformIO extension](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)

### 2. Clone Repository

```bash
git clone https://github.com/wakelinkdev/wakelink-firmware
cd wakelink-firmware
```

### 3. Build and Upload

=== "ESP8266"
    ```bash
    # Build only
    pio run -e esp8266

    # Build and upload
    pio run -e esp8266 -t upload

    # Upload and monitor serial
    pio run -e esp8266 -t upload -t monitor
    ```

=== "ESP32"
    ```bash
    # Build only
    pio run -e esp32

    # Build and upload
    pio run -e esp32 -t upload

    # Upload and monitor serial
    pio run -e esp32 -t upload -t monitor
    ```

### PlatformIO Configuration

The `platformio.ini` file defines build environments:

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

## Method 4: esptool (Command Line)

For pre-built binaries without IDE.

### 1. Install esptool

```bash
pip install esptool
```

### 2. Download Binary

Get pre-built `.bin` from [releases](https://github.com/wakelinkdev/wakelink-firmware/releases)

### 3. Flash

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

### 4. Erase Flash (if needed)

```bash
esptool.py --port COM3 erase_flash
```

---

## Verifying Installation

After flashing:

1. **Disconnect and reconnect** the USB cable
2. **Open Serial Monitor** (115200 baud)
3. **Press Reset** button on the board

You should see:

```
WakeLink v1.0.0 starting...
[CONFIG] No saved config, entering provisioning mode
[WIFI] Starting AP: WakeLink-Setup
[PROV] Captive portal at 192.168.4.1
```

### Serial Monitor

=== "Arduino IDE"
    **Tools → Serial Monitor** (set baud to 115200)

=== "PlatformIO"
    ```bash
    pio device monitor -b 115200
    ```

=== "putty/screen"
    ```bash
    # Linux/macOS
    screen /dev/ttyUSB0 115200

    # Windows (use PuTTY)
    # COM port, Speed: 115200
    ```

---

## Next Steps

[Configure your device →](configuration.md)
