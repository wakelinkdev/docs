# Quick Start

Get your first WakeLink device online in 5 minutes.

## Prerequisites

- ESP8266 or ESP32 board
- USB cable for flashing
- WiFi network credentials
- Target PC's MAC address ([how to find](requirements.md#3-find-your-mac-address))

---

## Step 1: Flash the Firmware

Choose your preferred method:

=== "Web Flasher (Easiest)"

    1. Connect your ESP board via USB
    2. Open [flash.wakelink.io](https://flash.wakelink.io) in Chrome/Edge
    3. Click **Connect** and select your COM port
    4. Click **Install WakeLink**
    5. Wait for completion (~60 seconds)

    !!! success "Done"
        The firmware is now installed!

=== "Arduino IDE"

    1. Download `wakelink-firmware` from [releases](https://github.com/wakelinkdev/wakelink-firmware/releases)
    2. Open `WakeLink.ino` in Arduino IDE
    3. Install required libraries:
        - `WebSocketsClient`
        - `ArduinoJson`
        - `Crypto` (for ESP8266) or built-in for ESP32
    4. Select your board:
        - ESP8266: `NodeMCU 1.0` or `Generic ESP8266 Module`
        - ESP32: `ESP32 Dev Module`
    5. Click **Upload**

=== "PlatformIO"

    ```bash
    # Clone the repository
    git clone https://github.com/wakelinkdev/wakelink-firmware
    cd wakelink-firmware

    # Build and upload (ESP8266)
    pio run -e esp8266 -t upload

    # Or for ESP32
    pio run -e esp32 -t upload
    ```

---

## Step 2: Configure the Device

After flashing, the device starts in **Provisioning Mode**.

### Connect to Provisioning Network

1. On your phone/laptop, look for WiFi network: `WakeLink-Setup`
2. Connect to it (no password)
3. Open [http://192.168.4.1](http://192.168.4.1) in your browser

### Enter Configuration

Fill in the form:

| Field | Value |
|-------|-------|
| **Device Name** | Any name (e.g., `living-room`) |
| **WiFi SSID** | Your home WiFi name |
| **WiFi Password** | Your WiFi password |
| **Server URL** | `wss://relay.wakelink.io` (or your own server) |
| **Device Token** | Your device token (from Step 3) |
| **Target MAC** | PC's MAC address (e.g., `AA:BB:CC:DD:EE:FF`) |

Click **Save**. The device will restart and connect to your WiFi.

!!! tip "LED Indicators"
    - **Blinking** = Connecting to WiFi
    - **Solid** = Connected to server
    - **Off** = Deep sleep (normal operation)

---

## Step 3: Register Your Device

You need a device token to authenticate with the server.

=== "CLI (Recommended)"

    ```bash
    # Install CLI
    pip install wakelink-cli

    # Register
    wakelink register --name living-room
    ```

    Copy the displayed token and use it in Step 2.

=== "Web Dashboard"

    1. Go to [app.wakelink.io](https://app.wakelink.io)
    2. Sign up or log in
    3. Click **Add Device**
    4. Copy the generated token

=== "Self-Hosted"

    ```bash
    # Inside your server container
    docker exec -it wakelink-api python -m wakelink.cli register --name living-room
    ```

---

## Step 4: Test Wake Command

### From CLI

```bash
# Wake the device
wakelink wake living-room

# With verbose output
wakelink wake living-room -v
```

### From Android App

1. Install from [Google Play](https://play.google.com/store/apps/details?id=io.wakelink)
2. Log in with your account
3. Tap your device
4. Tap **Wake**

### Expected Output

```
🚀 Sending wake command to 'living-room'...
✅ Wake command sent successfully
   Response time: 234ms
   Device acknowledged
```

---

## Step 5: Verify Your PC Wakes Up

1. Shut down or put your target PC to sleep
2. Wait 10-15 seconds
3. Send the wake command
4. PC should power on within 5 seconds

!!! warning "Troubleshooting"
    If your PC doesn't wake:

    1. **Check WoL settings**: [Requirements Guide](requirements.md#pc-wake-on-lan-setup)
    2. **Verify MAC address**: Typos are common
    3. **Check ESP connection**: Look at LED status
    4. **Check logs**: `wakelink logs living-room`

---

## Quick Reference

### CLI Commands

```bash
wakelink register --name <name>  # Register new device
wakelink wake <name>             # Send wake command
wakelink status <name>           # Check device status
wakelink logs <name>             # View device logs
wakelink list                    # List all devices
wakelink ping <name>             # Ping device
```

### Configuration Files

| Location | Purpose |
|----------|---------|
| `~/.wakelink/config.yaml` | CLI configuration |
| `~/.wakelink/devices.yaml` | Local device cache |

### Server Endpoints

| Endpoint | Description |
|----------|-------------|
| `wss://relay.wakelink.io` | Public relay server |
| `https://wakelink-project.org/api` | REST API |
| `https://app.wakelink.io` | Web dashboard |

---

## Next Steps

<div class="grid cards" markdown>

-   :material-tune:{ .lg .middle } __Advanced Configuration__

    ---

    Local mode, OTA updates, multiple targets

    [:octicons-arrow-right-24: Firmware Configuration](../firmware/configuration.md)

-   :material-server:{ .lg .middle } __Self-Host Server__

    ---

    Run your own relay server

    [:octicons-arrow-right-24: Server Setup](../server/docker.md)

-   :material-shield:{ .lg .middle } __Security Best Practices__

    ---

    Key rotation, network isolation

    [:octicons-arrow-right-24: Security Guide](../security/best-practices.md)

</div>

---

## Need Help?

- 📖 [FAQ](../reference/faq.md)
- 🔧 [Troubleshooting](../firmware/troubleshooting.md)
- 💬 [Discord Community](https://discord.gg/wakelink)
- 🐛 [Report a Bug](https://github.com/wakelinkdev/wakelink/issues/new)
