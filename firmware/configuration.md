# Firmware Configuration

Configure your WakeLink device settings.

## Initial Setup (Provisioning Mode)

When the device has no saved configuration, it starts in **Provisioning Mode**:

1. Connect to WiFi network: `WakeLink-Setup`
2. Open [http://192.168.4.1](http://192.168.4.1)
3. Fill in the configuration form
4. Click **Save**

!!! info "Captive Portal"
    Most devices will automatically open the configuration page when connecting to the setup network.

## Configuration Options

### Basic Settings

| Setting | Description | Required |
|---------|-------------|----------|
| **Device Name** | Friendly name (e.g., `office-pc`) | Yes |
| **WiFi SSID** | Your WiFi network name | Yes |
| **WiFi Password** | Your WiFi password | Yes |
| **Server URL** | WebSocket server address | Yes |
| **Device Token** | Authentication token from server | Yes |
| **Target MAC** | MAC address of PC to wake | Yes |

### Advanced Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Server Port** | WebSocket port | 443 |
| **Use TLS** | Enable encrypted connection | true |
| **Local Mode** | Enable local HTTP server | false |
| **Local Port** | Port for local HTTP server | 80 |
| **Deep Sleep** | Enable power saving | false |
| **Sleep Interval** | Minutes between wake checks | 5 |
| **LED Enabled** | Status LED indicators | true |
| **OTA Enabled** | Allow remote firmware updates | true |
| **Debug Mode** | Verbose serial logging | false |

---

## Configuration Methods

### Method 1: Web Interface

The web provisioning interface is the easiest method.

**Accessing the interface:**

1. **First time**: Device auto-starts in provisioning mode
2. **Reconfiguration**: Hold FLASH button for 5 seconds during boot
3. **Factory reset**: Hold FLASH button for 10 seconds

### Method 2: Serial Console

Configure via serial commands (115200 baud):

```bash
# View current configuration
config show

# Set individual values
config set wifi.ssid "MyNetwork"
config set wifi.password "MyPassword"
config set server.url "wss://relay.wakelink.io"
config set device.token "abc123..."
config set target.mac "AA:BB:CC:DD:EE:FF"
config set device.name "office-pc"

# Save and apply
config save
config apply

# Reset to factory defaults
config reset
```

### Method 3: JSON File

Upload a `config.json` file via the web interface:

```json
{
  "device": {
    "name": "office-pc",
    "token": "your-device-token-here"
  },
  "wifi": {
    "ssid": "MyNetwork",
    "password": "MyPassword"
  },
  "server": {
    "url": "wss://relay.wakelink.io",
    "port": 443,
    "tls": true
  },
  "target": {
    "mac": "AA:BB:CC:DD:EE:FF",
    "broadcast": "255.255.255.255",
    "port": 9
  },
  "local": {
    "enabled": false,
    "port": 80,
    "auth_required": true
  },
  "power": {
    "deep_sleep": false,
    "sleep_interval_minutes": 5
  },
  "features": {
    "led_enabled": true,
    "ota_enabled": true,
    "debug_mode": false
  }
}
```

---

## Server URL Formats

| Format | Example |
|--------|---------|
| Public relay | `wss://relay.wakelink.io` |
| Self-hosted (TLS) | `wss://your-server.com:443` |
| Self-hosted (no TLS) | `ws://your-server.local:8080` |
| IP address | `wss://203.0.113.50:443` |

!!! warning "Security"
    Always use `wss://` (WebSocket Secure) for production deployments.

---

## Target MAC Address

The MAC address of the computer you want to wake.

### Formats Accepted

All these formats are valid:

```
AA:BB:CC:DD:EE:FF
AA-BB-CC-DD-EE-FF
AABBCCDDEEFF
aa:bb:cc:dd:ee:ff
```

### Finding MAC Address

See [Requirements Guide](../getting-started/requirements.md#3-find-your-mac-address)

---

## Multiple Wake Targets

You can configure multiple MAC addresses for batch wake:

```json
{
  "targets": [
    {
      "name": "main-pc",
      "mac": "AA:BB:CC:DD:EE:FF"
    },
    {
      "name": "backup-server",
      "mac": "11:22:33:44:55:66"
    }
  ]
}
```

Then wake specific target:

```bash
wakelink wake office-pc --target main-pc
wakelink wake office-pc --target backup-server
wakelink wake office-pc --all  # Wake all targets
```

---

## Local Mode

Enable local HTTP server for LAN-only operation:

```json
{
  "local": {
    "enabled": true,
    "port": 80,
    "auth_required": true,
    "auth_token": "local-secret-token"
  }
}
```

### Local API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /` | GET | Status page |
| `GET /status` | GET | JSON status |
| `POST /wake` | POST | Send wake command |
| `GET /config` | GET | Current config |
| `POST /config` | POST | Update config |

### Local Wake Request

```bash
curl -X POST http://192.168.1.100/wake \
  -H "Authorization: Bearer local-secret-token" \
  -H "Content-Type: application/json" \
  -d '{"target": "main-pc"}'
```

---

## Power Management

### Deep Sleep Mode

For battery-powered deployments:

```json
{
  "power": {
    "deep_sleep": true,
    "sleep_interval_minutes": 5,
    "wake_on_schedule": true,
    "schedule": ["08:00", "12:00", "18:00"]
  }
}
```

**How it works:**

1. Device wakes from deep sleep
2. Connects to WiFi and server
3. Checks for pending commands (30 second window)
4. Executes any pending commands
5. Returns to deep sleep

!!! warning "Deep Sleep Limitation"
    In deep sleep mode, wake commands are not instant — the device only checks at the configured interval.

---

## GPIO Configuration

Customize LED and button pins:

```json
{
  "gpio": {
    "led_pin": 2,
    "led_inverted": true,
    "button_pin": 0,
    "button_inverted": true
  }
}
```

| Board | Default LED | Default Button |
|-------|-------------|----------------|
| NodeMCU | GPIO2 (inverted) | GPIO0 |
| Wemos D1 Mini | GPIO2 (inverted) | GPIO0 |
| ESP32 DevKit | GPIO2 | GPIO0 |

---

## Validation

The device validates configuration before applying:

| Check | Error |
|-------|-------|
| WiFi SSID length | 1-32 characters |
| WiFi password | 0 or 8-63 characters |
| MAC address format | Valid hex, 6 bytes |
| Server URL | Valid ws/wss URL |
| Token | Non-empty string |

Invalid configuration shows error on web interface and serial console.

---

## Factory Reset

Three ways to reset:

1. **Button**: Hold FLASH for 10+ seconds during boot
2. **Serial**: `config reset` command
3. **Web**: Provisioning page reset button

Reset clears all settings and restarts in provisioning mode.

---

## Backup & Restore

### Export Configuration

```bash
# Via serial
config export

# Via web interface
GET http://192.168.4.1/config/export
```

### Import Configuration

```bash
# Via serial (paste JSON)
config import {"device":{"name":"test"},...}

# Via web interface
POST http://192.168.4.1/config/import
Content-Type: application/json
{...configuration JSON...}
```

---

Continue to [OTA Updates →](ota.md)
