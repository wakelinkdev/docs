[🇬🇧 English](error-codes.md) | [🇷🇺 Русский](error-codes_RU.md)

# Error Codes Reference

Complete list of WakeLink error codes and their meanings.

## Protocol Errors (E0xx)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| E001 | `INVALID_PACKET` | Malformed packet structure | Check packet format |
| E002 | `AUTH_FAILED` | Authentication failed | Verify token, re-login |
| E003 | `SEQ_OUT_OF_ORDER` | Invalid sequence number | Reconnect session |
| E004 | `CHAIN_BROKEN` | Chain hash mismatch | Reset chain state |
| E005 | `SIG_INVALID` | Signature verification failed | Check key, reconnect |
| E006 | `TS_EXPIRED` | Timestamp out of range | Sync time on device |
| E007 | `UNKNOWN_CMD` | Unrecognized command | Update firmware |
| E008 | `CMD_FAILED` | Command execution failed | See specific error |
| E009 | `DECRYPT_FAILED` | Decryption error | Wrong key, reconnect |
| E010 | `PACKET_TOO_LARGE` | Packet exceeds size limit | Reduce payload |

## Network Errors (E1xx)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| E101 | `CONNECT_FAILED` | Cannot connect to server | Check network, server URL |
| E102 | `DNS_FAILED` | DNS resolution failed | Check DNS settings |
| E103 | `TLS_HANDSHAKE` | TLS handshake failed | Check certificates |
| E104 | `TIMEOUT` | Connection timed out | Retry, check network |
| E105 | `DISCONNECTED` | Connection lost | Auto-reconnect |
| E106 | `WIFI_DISCONNECTED` | WiFi connection lost | Check WiFi credentials |

## Device Errors (E2xx)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| E201 | `DEVICE_NOT_FOUND` | Device doesn't exist | Check device name/ID |
| E202 | `DEVICE_OFFLINE` | Device is disconnected | Wait for reconnection |
| E203 | `DEVICE_BUSY` | Device processing other command | Retry later |
| E204 | `INVALID_CONFIG` | Configuration invalid | Fix configuration |
| E205 | `FLASH_ERROR` | Flash storage error | Factory reset |
| E206 | `OTA_FAILED` | Firmware update failed | Retry, USB flash |

## Wake Errors (E3xx)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| E301 | `INVALID_MAC` | Invalid MAC address format | Check MAC format |
| E302 | `WOL_SEND_FAILED` | Failed to send WoL packet | Check network |
| E303 | `TARGET_NOT_FOUND` | Target not in config | Add target |
| E304 | `RATE_LIMITED` | Too many wake commands | Wait and retry |

## Server Errors (E4xx)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| E401 | `UNAUTHORIZED` | Not authenticated | Login first |
| E402 | `FORBIDDEN` | Not authorized for resource | Check permissions |
| E403 | `NOT_FOUND` | Resource not found | Check URL/ID |
| E404 | `VALIDATION_ERROR` | Invalid request data | Fix request body |
| E405 | `RATE_LIMITED` | Rate limit exceeded | Slow down |
| E406 | `SERVER_ERROR` | Internal server error | Contact support |
| E407 | `SERVICE_UNAVAILABLE` | Server temporarily down | Retry later |

## CLI Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Authentication error |
| 3 | Device not found |
| 4 | Device offline |
| 5 | Timeout |
| 6 | Network error |
| 7 | Configuration error |

---

## Error Response Format

### API Error Response

```json
{
  "error": {
    "code": "E201",
    "name": "DEVICE_NOT_FOUND",
    "message": "Device with ID 'd7a8f9b0-...' not found",
    "details": {
      "device_id": "d7a8f9b0-..."
    }
  }
}
```

### Protocol Error Packet

```json
{
  "v": 2,
  "id": "...",
  "seq": 43,
  "prev": "...",
  "p": "<encrypted: {cmd: 'ERROR', data: {code: 'E005', message: '...'}}}>",
  "sig": "..."
}
```

---

## Handling Errors

### In Code

```python
try:
    client.wake("my-device")
except WakeLinkError as e:
    if e.code == "E202":  # DEVICE_OFFLINE
        print("Device is offline, queued for later")
    elif e.code == "E304":  # RATE_LIMITED
        time.sleep(60)
        client.wake("my-device")  # Retry
    else:
        raise
```

### In CLI

```bash
if ! wakelink wake my-device; then
    echo "Wake failed"
    exit 1
fi
```

---

## Common Issues

### E004: CHAIN_BROKEN

**Causes:**
- Device or server restarted but other side has old chain state
- Network caused packet loss

**Solution:**
Both sides automatically attempt chain recovery. If persistent, restart device.

### E006: TS_EXPIRED

**Causes:**
- Device clock significantly off
- Network latency exceeds threshold

**Solution:**
1. Check device time sync
2. Verify NTP server access
3. Increase timestamp tolerance (server config)

### E202: DEVICE_OFFLINE

**Causes:**
- Device not powered
- WiFi connection lost
- Server connection dropped

**Solution:**
1. Check device power
2. Check WiFi connectivity
3. View device serial logs
4. Restart device

### E304: RATE_LIMITED

**Causes:**
- Too many wake commands in short period
- Default: 30 commands/minute

**Solution:**
1. Wait for rate limit reset
2. Reduce command frequency
3. Request higher limits (if self-hosted)
