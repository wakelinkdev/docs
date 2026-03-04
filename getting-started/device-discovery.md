# WakeLink Device Discovery

WakeLink devices support three methods for discovery and connection:

## 1. mDNS (Recommended for single device)

When connected to WiFi, ESP32 advertises itself as **`wakelink.local`**.

### Android
```kotlin
// Simple connection via mDNS
val result = SetupHandler.connectViaMdns()
if (result.success) {
    // Device is accessible at wakelink.local
}
```

### CLI
```bash
wakelink-cli --host wakelink.local config show
```

### Manual
- Open browser: `http://wakelink.local:80`
- Works on any network (local WiFi, home network, etc.)

**Benefits:**
- Zero configuration
- Works across networks
- Automatic local name resolution

**Limitations:**
- Only works with one device per network
- Some networks may block mDNS

---

## 2. UDP Broadcast Discovery (Recommended for multiple devices)

Devices broadcast their presence every 10 seconds on port **5555**.

### Broadcast Packet Format

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
// Scan for devices (5 second timeout)
scope.launch {
    val devices = SetupHandler.discoverDevices(timeoutMs = 5000)
    devices.forEach { device ->
        println("Found: ${device.displayName} at ${device.ip}:${device.port}")
    }
}
```

### CLI

```bash
wakelink-cli discover --timeout 5s
# Output:
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

**Benefits:**
- Finds all devices on network automatically
- Works with multiple devices
- Cross-platform

**Limitations:**
- Some networks/routers block UDP broadcast
- Only works on local network (no internet)

---

## 3. Cloud Relay (For remote/cloud access)

After device connects to WiFi and cloud server:

```
Mobile App → Cloud Server ← Device (via WSS or HTTPS)
```

### Android

```kotlin
val cloudHandler = CloudHandler(cloudUrl = "https://relay.wakelink.io")
val devices = cloudHandler.listMyDevices()  // Returns all registered devices
val device = devices.first()
val client = cloudHandler.connectToDevice(device.id)
```

### API

```bash
GET /api/v1/devices
# Returns list of all user's devices with IPs/ports
```

**Benefits:**
- Works anywhere (behind NAT, different networks)
- Secure cloud relay
- Multi-device management

**Limitations:**
- Requires internet connection
- Cloud account required
- Potential latency

---

## Connection Priority

Recommended connection order:

1. **mDNS** - Try `wakelink.local` first (fast, local)
2. **UDP Discovery** - Scan for devices (finds all on network)
3. **AP Mode** - If not connected, device is in AP mode at `192.168.4.1`
4. **Cloud** - For remote access

---

## Implementation Example: SetupScreen

```kotlin
val setupScope = rememberCoroutineScope()

// Try mDNS first
LaunchedEffect(Unit) {
    setupScope.launch {
        val result = SetupHandler.connectViaMdns()
        if (result.success) {
            // Device found via mDNS
            isConnected = true
            apIp = "wakelink.local"  // Use mDNS hostname
        } else {
            // Fall back to UDP discovery
            val devices = SetupHandler.discoverDevices(timeoutMs = 5000)
            if (devices.isNotEmpty()) {
                // Connect to first device found
                apIp = devices.first().ip
                isConnected = true
            } else {
                // No device found, must be in AP mode
                apIp = SetupHandler.DEFAULT_AP_IP
            }
        }
    }
}
```

---

## Configuration

### ESP32 Firmware

```cpp
// Enable mDNS discovery
MDNS.begin("wakelink");
MDNS.addService("wakelink", "tcp", 80);

// UDP broadcast happens every 10 seconds automatically
// Port: 5555 (configurable in code)
```

### Router Requirements

- **mDNS**: Most routers support it (enabled by default)
- **UDP Broadcast**: Some enterprise networks block it
- **Cloud Relay**: Requires internet access

---

## Troubleshooting

### mDNS not working
- Check router supports mDNS (most do)
- Verify device is connected to same WiFi
- Try manual IP or UDP discovery

### UDP broadcast not found
- Check firewall/router allows UDP port 5555
- Try connecting to `192.168.4.1` (AP mode)
- Verify device is connected to WiFi

### Cloud relay slow
- Check internet connection
- Check cloud server status
- Try local connection first

---

## See Also

- [Device Configuration](./configuration.md)
- [Web API Reference](../reference/api.md)
- [Android Setup Guide](../clients/android.md)
