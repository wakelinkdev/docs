# FAQ

Frequently asked questions about WakeLink.

## General

### What is WakeLink?

WakeLink is an open-source system for remotely waking up computers over the Internet using Wake-on-LAN (WoL) magic packets, with end-to-end encryption for security.

### How is WakeLink different from regular Wake-on-LAN?

Regular WoL only works on local networks. WakeLink adds:
- Internet connectivity via relay server
- End-to-end encryption
- No port forwarding required
- Mobile app and CLI

### Is WakeLink free?

Yes, WakeLink is completely free and open source (MIT license). You can also self-host for complete control.

### Do I need technical knowledge?

Basic setup requires minimal technical knowledge. Advanced features (self-hosting, custom configuration) require more expertise.

---

## Hardware

### Which ESP boards are supported?

- ESP8266 (NodeMCU, Wemos D1 Mini, etc.)
- ESP32 (all variants including S2, C3)

ESP8266 is recommended for simplicity and cost.

### Can I use a Raspberry Pi instead?

Not currently, but community contributions are welcome. The protocol is documented for porting.

### How much does the hardware cost?

ESP8266 boards typically cost $3-5 USD. That's the only required purchase.

### Does the ESP need to be always powered?

Yes, the ESP must remain powered to receive wake commands. Power consumption is approximately:
- Active: ~70mA
- Deep sleep: ~20μA (with periodic wakeup)

---

## Setup

### My ESP won't connect to WiFi

1. Verify correct SSID and password
2. Ensure 2.4 GHz network (ESP8266 doesn't support 5 GHz)
3. Check for special characters in credentials
4. Move closer to router

See [Troubleshooting](../firmware/troubleshooting.md) for more.

### My PC doesn't wake up

1. **Enable WoL in BIOS**: Look for "Wake on LAN" or "Wake on PCI-E"
2. **Enable in OS**: Device Manager → Network adapter → Power Management
3. **Disable Fast Startup** (Windows)
4. **Verify MAC address**: Check for typos
5. **Use Ethernet**: WiFi WoL is unreliable

See [Requirements](../getting-started/requirements.md#pc-wake-on-lan-setup).

### How do I find my PC's MAC address?

**Windows**: `ipconfig /all` → Look for "Physical Address"
**Linux**: `ip link show` or `cat /sys/class/net/eth0/address`
**macOS**: `ifconfig en0 | grep ether`

### Can I wake multiple PCs?

Yes, configure multiple targets on one ESP device, or use multiple ESP devices.

---

## Security

### Is WakeLink secure?

Yes. WakeLink uses:
- XChaCha20-Poly1305 encryption (same class as TLS 1.3)
- End-to-end encryption (server cannot read commands)
- HMAC authentication
- Replay protection

See [Security Overview](../security/index.md).

### Can the server see my wake commands?

No. Commands are encrypted end-to-end. The server only sees encrypted blobs.

### What if someone gets my device token?

They could send wake commands to your device. If compromised:
```bash
wakelink token regenerate my-device
```
Then update the ESP with the new token.

### Should I self-host?

Self-hosting provides maximum control but requires maintenance. The public server is secure due to E2E encryption — we can't read your commands anyway.

---

## Connectivity

### Do I need to open ports on my router?

No. WakeLink uses outbound WebSocket connections only.

### Does WakeLink work behind CGNAT?

Yes, because it uses outbound connections.

### What if my ESP loses WiFi?

It will automatically reconnect. Wake commands sent while offline are queued for delivery.

### What's the latency?

Typical end-to-end latency is 100-500ms depending on network conditions.

---

## Server

### Can I self-host?

Yes! See [Server Deployment](../server/docker.md).

### What are the server requirements?

Minimum: 1 CPU, 1 GB RAM, 10 GB storage. Runs on any VPS or home server.

### Is there a hosted option?

Yes, the public relay at `relay.wakelink.io` is free to use.

### How many devices can one server handle?

A single server can handle 10,000+ devices. Horizontal scaling available for more.

---

## Mobile & CLI

### Is there an iOS app?

Not yet, but it's planned. The API is available for third-party integrations.

### Can I use WakeLink from a script?

Yes, the CLI is designed for scripting:
```bash
wakelink wake my-pc
```

### How do I automate wake commands?

Use cron (Linux/macOS) or Task Scheduler (Windows):
```bash
# Wake PC at 8 AM weekdays
0 8 * * 1-5 wakelink wake office-pc
```

---

## Troubleshooting

### "Device offline" error

1. Check ESP power
2. Check ESP WiFi (LED status)
3. Check server connection (serial logs)
4. Restart ESP device

### "Authentication failed"

1. Verify token is correct
2. Re-login on CLI: `wakelink login`
3. Regenerate device token if needed

### "Timeout" error

1. Device may be offline
2. Network issues
3. Increase timeout: `wakelink wake my-device --timeout 60`

### LED blinks but PC doesn't wake

1. WoL not enabled on PC
2. Wrong MAC address
3. PC not on same subnet as ESP
4. Fast Startup enabled (Windows)

---

## Updates

### How do I update the firmware?

Over-the-air (OTA):
```bash
wakelink ota update my-device
```

Or reflash via USB using the Web Flasher.

### How do I update the server?

```bash
docker-compose pull
docker-compose up -d
```

### How do I update the CLI?

```bash
pip install --upgrade wakelink-cli
```

---

## Contributing

### How can I contribute?

- Report bugs on GitHub Issues
- Submit pull requests
- Write documentation
- Help others on Discord

### Where's the source code?

- Firmware: github.com/wakelinkdev/wakelink-firmware
- Server: github.com/wakelinkdev/wakelink-server
- CLI: github.com/wakelinkdev/wakelink-client
- Android: github.com/wakelinkdev/wakelink-android

### What license is WakeLink under?

MIT License — free to use, modify, and distribute.

---

## Still Have Questions?

- **Discord**: [discord.gg/wakelink](https://discord.gg/wakelink)
- **GitHub Discussions**: [github.com/wakelinkdev/wakelink/discussions](https://github.com/wakelinkdev/wakelink/discussions)
- **Email**: support@wakelink.io
