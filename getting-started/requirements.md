# Requirements

Before setting up WakeLink, ensure you have the following hardware and software.

## Hardware Requirements

### ESP Board (Required)

You'll need an ESP8266 or ESP32 microcontroller board. These are low-cost WiFi-enabled boards that will send Wake-on-LAN packets.

| Board | Price | Notes |
|-------|-------|-------|
| **ESP8266 (NodeMCU)** | ~$3-5 | Recommended. Widely available, proven reliable |
| **ESP32** | ~$5-8 | More powerful, Bluetooth support (not needed for WakeLink) |
| **Wemos D1 Mini** | ~$3-4 | Compact form factor, ESP8266-based |

!!! tip "Where to Buy"
    - Amazon, AliExpress, eBay
    - Local electronics stores
    - Search for "NodeMCU ESP8266" or "ESP32 DevKit"

### USB Cable

- Micro-USB cable (for most ESP8266 boards)
- USB-C cable (for newer ESP32 boards)

This is for initial flashing only. After setup, the ESP runs independently.

### Target Computer

Your PC must support Wake-on-LAN:

- [x] Ethernet port (WiFi WoL is unreliable)
- [x] WoL-capable network card (most modern NICs)
- [x] Powered PSU (PC must be plugged in)

!!! warning "WoL Limitations"
    - Works from **Shutdown** or **Sleep** states
    - Does **NOT** work if PC is forcefully powered off
    - Must be enabled in BIOS and OS

### Network Requirements

- WiFi network (2.4 GHz for ESP8266)
- ESP and target PC on the **same local network**
- Internet connection (for relay communication)

## Software Requirements

### For Flashing Firmware

=== "Arduino IDE"
    - [Arduino IDE 2.x](https://www.arduino.cc/en/software)
    - ESP8266/ESP32 board support
    - Required libraries (installed automatically)

=== "PlatformIO"
    - [VS Code](https://code.visualstudio.com/)
    - [PlatformIO extension](https://platformio.org/install/ide?install=vscode)

=== "Web Flasher"
    - Modern browser (Chrome, Edge, Firefox)
    - No installation required

### For Server (Self-Hosted)

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- 512MB RAM minimum
- Linux recommended (Windows/macOS work too)

### For Clients

=== "CLI"
    - Python 3.9+
    - pip (Python package manager)

=== "Android"
    - Android 8.0+ (API 26)
    - Google Play Store or APK sideload

## PC Wake-on-LAN Setup

### 1. Enable in BIOS/UEFI

1. Enter BIOS (usually Del, F2, or F12 during boot)
2. Find Power Management or Network settings
3. Enable **Wake on LAN** / **Wake on PCI** / **Power On By PCI-E**
4. Save and exit

!!! note "BIOS Names Vary"
    Look for: "Wake on LAN", "WoL", "Power On by Network", "Wake on PCI/PCI-E"

### 2. Enable in Operating System

=== "Windows 11/10"
    1. Open **Device Manager**
    2. Expand **Network adapters**
    3. Right-click your Ethernet adapter → **Properties**
    4. Go to **Power Management** tab
    5. Check:
        - ✅ Allow this device to wake the computer
        - ✅ Only allow a magic packet to wake the computer
    6. Go to **Advanced** tab
    7. Set **Wake on Magic Packet** = Enabled

=== "Windows (Disable Fast Startup)"
    Fast Startup can prevent WoL from working:

    1. Open **Control Panel** → **Power Options**
    2. Click **Choose what the power buttons do**
    3. Click **Change settings that are currently unavailable**
    4. Uncheck **Turn on fast startup**
    5. Save changes

=== "Linux"
    ```bash
    # Install ethtool
    sudo apt install ethtool

    # Check current WoL status
    sudo ethtool eth0 | grep Wake

    # Enable WoL (g = magic packet)
    sudo ethtool -s eth0 wol g
    ```

    Make it persistent (systemd):
    ```bash
    # /etc/systemd/system/wol.service
    [Unit]
    Description=Enable Wake-on-LAN
    After=network.target

    [Service]
    Type=oneshot
    ExecStart=/sbin/ethtool -s eth0 wol g

    [Install]
    WantedBy=multi-user.target
    ```

=== "macOS"
    1. **System Preferences** → **Energy Saver** (or Battery)
    2. Check **Wake for network access**

### 3. Find Your MAC Address

You'll need the target PC's MAC address for configuration.

=== "Windows"
    ```cmd
    ipconfig /all
    ```
    Look for "Physical Address" under your Ethernet adapter.

=== "Linux"
    ```bash
    ip link show
    # or
    cat /sys/class/net/eth0/address
    ```

=== "macOS"
    ```bash
    ifconfig en0 | grep ether
    ```

!!! example "MAC Address Format"
    ```
    AA:BB:CC:DD:EE:FF
    ```
    WakeLink accepts formats: `AA:BB:CC:DD:EE:FF`, `AA-BB-CC-DD-EE-FF`, `AABBCCDDEEFF`

## Verification Checklist

Before proceeding, verify:

- [ ] ESP board in hand (ESP8266 or ESP32)
- [ ] USB cable for flashing
- [ ] WiFi network name and password ready
- [ ] Target PC MAC address noted
- [ ] WoL enabled in BIOS
- [ ] WoL enabled in OS
- [ ] Fast Startup disabled (Windows)

---

Ready? [Continue to Quick Start →](quickstart.md)
