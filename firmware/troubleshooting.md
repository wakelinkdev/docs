[🇬🇧 English](troubleshooting.md) | [🇷🇺 Русский](troubleshooting_RU.md)

# Troubleshooting

Common issues and solutions for WakeLink firmware.

## Diagnostic Tools

### Serial Monitor

First step in any troubleshooting:

```bash
# PlatformIO
pio device monitor -b 115200

# Arduino IDE
Tools → Serial Monitor (115200 baud)
```

### Status Command

```bash
wakelink status my-device
```

Output includes:
- Connection state
- Last seen timestamp
- Firmware version
- WiFi signal strength

---

## WiFi Issues

### Device Won't Connect to WiFi

**Symptoms:**
- LED blinking slowly
- Serial: `[WIFI] Connecting...` repeating

**Solutions:**

1. **Check credentials**
   ```
   config set wifi.ssid "ExactNetworkName"
   config set wifi.password "ExactPassword"
   config save
   ```

2. **Verify 2.4 GHz** — ESP8266 doesn't support 5 GHz

3. **Check special characters** — Some routers have issues with spaces or symbols in SSID

4. **Move closer to router** — Weak signal causes connection failures

5. **Check MAC filtering** — Whitelist ESP's MAC address on router

### WiFi Connected but No Server Connection

**Symptoms:**
- LED blinking fast (WiFi OK, server failing)
- Serial: `[WS] Connection failed`

**Solutions:**

1. **Check server URL**
   ```
   config get server.url
   # Should be wss://relay.wakelink.io or your server
   ```

2. **Check firewall** — Outbound port 443 (HTTPS/WSS) must be open

3. **DNS resolution** — Try IP address instead of hostname

4. **Verify TLS** — Self-hosted servers need valid certificates

### WiFi Keeps Disconnecting

**Symptoms:**
- Frequent reconnections in logs
- Intermittent availability

**Solutions:**

1. **Power supply** — Use stable 5V source, avoid PC USB ports
2. **WiFi interference** — Move away from microwaves, Bluetooth devices
3. **Router overload** — Too many connected devices
4. **Firmware update** — Older versions had stability issues

---

## Wake-on-LAN Issues

### Wake Command Sent but PC Doesn't Wake

This is the most common issue. Check in order:

#### 1. Verify MAC Address

```bash
# Check configured MAC
wakelink config my-device | grep mac

# Compare with actual PC MAC (on the PC)
ipconfig /all  # Windows
ip link show   # Linux
```

#### 2. Verify WoL Enabled on PC

**BIOS Settings:**
- Wake on LAN: Enabled
- Wake on PCI-E: Enabled
- ErP/EuP: Disabled (saves power but blocks WoL)

**Windows:**
1. Device Manager → Network Adapters → Your adapter → Properties
2. Power Management: ✅ Allow this device to wake the computer
3. Advanced: Wake on Magic Packet = Enabled

**Disable Fast Startup (Windows):**
```
Control Panel → Power Options → Choose what power buttons do
→ Change settings → Uncheck "Turn on fast startup"
```

#### 3. Test Locally

Send WoL from another device on the same network:

```bash
# Linux
wakeonlan AA:BB:CC:DD:EE:FF

# Windows (with tool)
wolcmd AABBCCDDEEFF 192.168.1.255 255.255.255.0 9

# Python
pip install wakeonlan
wakeonlan AA:BB:CC:DD:EE:FF
```

If local WoL works but WakeLink doesn't, the issue is with the ESP device.

#### 4. Check ESP Network

ESP must be on the **same subnet** as the target PC:

```
ESP IP:     192.168.1.100
Target PC:  192.168.1.50
            └── Same subnet ✅

ESP IP:     192.168.1.100
Target PC:  192.168.2.50
            └── Different subnet ❌
```

#### 5. Check Broadcast Address

Default is `255.255.255.255`. Some networks require subnet broadcast:

```json
{
  "target": {
    "mac": "AA:BB:CC:DD:EE:FF",
    "broadcast": "192.168.1.255"
  }
}
```

---

## Server Connection Issues

### "Authentication Failed"

**Symptoms:**
- Serial: `[AUTH] Token rejected`
- CLI: `Error: Device not authenticated`

**Solutions:**

1. **Regenerate token**
   ```bash
   wakelink token regenerate my-device
   ```
   
2. **Update device config** with new token

3. **Check token expiry** — Tokens may expire after long inactivity

### "Device Offline"

**Symptoms:**
- CLI: `Error: Device offline`
- Dashboard shows offline status

**Solutions:**

1. **Check physical device** — Is it powered?
2. **Check WiFi** — Can you ping the device?
3. **Check serial logs** — Any error messages?
4. **Restart device** — Sometimes WebSocket needs reconnection

### "Connection Timeout"

**Symptoms:**
- Commands hang then timeout
- Serial: `[WS] Ping timeout`

**Solutions:**

1. **Check internet** — Device needs stable connection
2. **Check server status** — Visit status.wakelink.io
3. **Reduce ping interval** — Network may be unstable
   ```json
   {
     "server": {
       "ping_interval_ms": 15000
     }
   }
   ```

---

## Provisioning Issues

### Can't Find "WakeLink-Setup" Network

**Solutions:**

1. **Wait 30 seconds** — AP takes time to start
2. **Factory reset** — Hold FLASH button 10+ seconds
3. **Check if already configured** — Device may be trying to connect to saved WiFi
4. **ESP may be damaged** — Try different board

### Captive Portal Doesn't Open

**Solutions:**

1. **Navigate manually** — Open http://192.168.4.1
2. **Disable mobile data** — Phone may prefer 4G
3. **Try different device** — Some captive portal implementations have issues
4. **Clear browser cache**

### Configuration Saves but Doesn't Apply

**Solutions:**

1. **Wait for reboot** — Device should reboot after save
2. **Check serial** — Look for config validation errors
3. **Factory reset and retry**

---

## LED Behavior Reference

| Pattern | Meaning | Action |
|---------|---------|--------|
| Off | No power or deep sleep | Check power supply |
| Solid ON | Connected and ready | Normal operation |
| Slow blink (1/sec) | Connecting to WiFi | Wait or check credentials |
| Fast blink (4/sec) | WiFi OK, server connecting | Check server URL/token |
| Double blink | Command received | Normal |
| Triple blink | Error occurred | Check serial logs |
| Continuous rapid | Firmware update in progress | Don't power off! |

---

## Memory Issues

### "Out of Memory" Errors

**Symptoms:**
- Device crashes/reboots
- Serial: `[HEAP] Low memory warning`

**Solutions:**

1. **Reduce JSON buffer** — Config file too large
2. **Disable debug mode** — Debug logging uses RAM
3. **Update firmware** — Memory leaks may be fixed
4. **Use ESP32** — More RAM available

### Configuration Lost After Reboot

**Solutions:**

1. **Check flash health** — Flash may be worn out
2. **Verify save operation** — Serial should show `[CONFIG] Saved`
3. **Check power stability** — Brown-out can corrupt flash

---

## Recovery Procedures

### Factory Reset

**Method 1: Button**
1. Power off device
2. Hold FLASH button
3. Power on while holding
4. Keep holding for 10 seconds
5. Release — device enters provisioning mode

**Method 2: Serial**
```
config reset
```

**Method 3: Erase Flash**
```bash
esptool.py --port COM3 erase_flash
```
Then re-flash firmware.

### Recover from Bad OTA Update

If device doesn't boot after OTA:

1. **Wait** — Watchdog should trigger rollback (30 seconds)
2. **If no rollback** — Factory reset via button
3. **If button doesn't work** — Re-flash via USB

### Bricked Device

If device seems completely unresponsive:

1. **Check power** — Is LED on at all?
2. **Try different USB cable**
3. **Try different USB port**
4. **Flash in download mode**:
   - Hold FLASH + RST
   - Release RST
   - Release FLASH
   - Device now in download mode

---

## Getting Help

If you can't resolve the issue:

1. **Collect information**:
   - Serial log output
   - Firmware version
   - Board type
   - Network configuration
   - Steps to reproduce

2. **Search existing issues**:
   - [GitHub Issues](https://github.com/wakelinkdev/wakelink-firmware/issues)

3. **Ask community**:
   - [Discord #help](https://discord.gg/wakelink)

4. **Open new issue**:
   - Use bug report template
   - Include all collected information
