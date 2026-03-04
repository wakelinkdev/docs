# Python CLI

Full-featured command-line interface for WakeLink.

## Installation

```bash
pip install wakelink-cli
```

Or from source:
```bash
git clone https://github.com/wakelinkdev/wakelink-client
cd wakelink-client
pip install -e .
```

### Requirements

- Python 3.9+
- pip

## Quick Start

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

## Commands Reference

### Authentication

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

### Device Management

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

### Wake Operations

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

### Monitoring

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

### Configuration

```bash
# Show config
wakelink config show

# Set config value
wakelink config set server.api_url https://api.custom.com

# Reset to defaults
wakelink config reset
```

### Utilities

```bash
# Version
wakelink version

# Help
wakelink --help
wakelink wake --help
```

---

## Configuration

### Config File

Located at `~/.wakelink/config.yaml`:

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

### Environment Variables

| Variable | Description |
|----------|-------------|
| `WAKELINK_API_URL` | API server URL |
| `WAKELINK_TOKEN` | Authentication token |
| `WAKELINK_CONFIG` | Config file path |
| `WAKELINK_NO_COLOR` | Disable colors |
| `WAKELINK_DEBUG` | Enable debug output |

---

## Output Formats

### Text (Default)

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

### Table

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

## Scripting

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Authentication error |
| 3 | Device not found |
| 4 | Device offline |
| 5 | Timeout |

### Examples

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

### Python Integration

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

## Advanced Usage

### Batch Operations

```bash
# Wake devices from file
cat devices.txt | xargs -I {} wakelink wake {}

# Parallel wake
cat devices.txt | xargs -P 5 -I {} wakelink wake {}
```

### Cron Jobs

```bash
# Wake office PC at 8 AM on weekdays
0 8 * * 1-5 /usr/local/bin/wakelink wake office-pc >> /var/log/wakelink.log 2>&1
```

### Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias wl='wakelink'
alias wlw='wakelink wake'
alias wls='wakelink status'
alias wll='wakelink list'
```

---

## Troubleshooting

### "Authentication failed"

```bash
# Re-login
wakelink logout
wakelink login
```

### "Connection timeout"

```bash
# Check server status
curl https://wakelink-project.org/api/health

# Increase timeout
wakelink wake my-device --timeout 60
```

### "Device not found"

```bash
# List available devices
wakelink list

# Check exact name
wakelink show my-device
```

### Debug Mode

```bash
WAKELINK_DEBUG=1 wakelink wake my-device
```

---

## Uninstall

```bash
pip uninstall wakelink-cli

# Remove config
rm -rf ~/.wakelink
```
