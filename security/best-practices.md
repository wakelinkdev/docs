# Security Best Practices

Recommendations for secure WakeLink deployment.

## Device Security

### Physical Security

1. **Secure location**: Place ESP device in a secure area
2. **No exposed pins**: Cover UART/debug headers
3. **Tamper detection**: Consider enclosure with detection

### Firmware Security

1. **Flash encryption** (ESP32):
   ```bash
   # Enable in menuconfig
   idf.py menuconfig
   # Security features → Enable flash encryption
   ```

2. **Secure boot** (ESP32):
   ```bash
   # Security features → Enable secure boot
   ```

3. **Disable debug output** in production:
   ```cpp
   // config.h
   #define DEBUG_MODE false
   ```

### Token Security

1. **Never share device tokens** publicly
2. **Rotate tokens** if compromise suspected:
   ```bash
   wakelink token regenerate my-device
   ```
3. **Use unique tokens** per device

---

## Server Security

### Configuration

```bash
# .env - Security settings
DEBUG=false
LOG_LEVEL=WARNING

# Strong, unique secret
JWT_SECRET=$(openssl rand -hex 32)

# Restrict CORS
CORS_ORIGINS=https://your-domain.com

# Enable rate limiting
RATE_LIMIT_ENABLED=true
RATE_LIMIT_PER_MINUTE=60
```

### Database

1. **Strong password**:
   ```bash
   DB_PASSWORD=$(openssl rand -hex 32)
   ```

2. **Network isolation**:
   ```yaml
   # docker-compose.yml
   services:
     db:
       networks:
         - internal  # Not exposed to host
   ```

3. **Regular backups** with encryption

### TLS Configuration

1. **TLS 1.3 only** (if clients support):
   ```nginx
   ssl_protocols TLSv1.3;
   ```

2. **Strong ciphers**:
   ```nginx
   ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
   ```

3. **HSTS header**:
   ```nginx
   add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
   ```

### Firewall

```bash
# Only allow necessary ports
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp    # SSH (restrict to your IP)
ufw allow 80/tcp    # HTTP (for ACME)
ufw allow 443/tcp   # HTTPS
ufw enable
```

---

## Network Security

### Home Network

1. **Separate IoT network**: Put ESP on isolated VLAN
2. **No UPnP**: Disable UPnP on router
3. **No port forwarding**: WakeLink doesn't need it

```
┌─────────────────────────────────────────┐
│              Home Network                │
│  ┌──────────────┐  ┌──────────────────┐ │
│  │  Main VLAN   │  │   IoT VLAN       │ │
│  │  (Trusted)   │  │   (Isolated)     │ │
│  │              │  │                  │ │
│  │  PC, Laptop  │  │  ESP, Cameras    │ │
│  └──────────────┘  └──────────────────┘ │
│          │                  │           │
│          └────────┬─────────┘           │
│                   │                     │
│            ┌──────▼──────┐              │
│            │   Firewall   │              │
│            │  (Block IoT→Main)          │
│            └─────────────┘              │
└─────────────────────────────────────────┘
```

### DNS Security

1. Use **DNS-over-HTTPS** or **DNS-over-TLS**
2. Consider **Pi-hole** for local DNS with filtering

---

## Account Security

### Strong Passwords

- Minimum 12 characters
- Mix of letters, numbers, symbols
- Use password manager

### Two-Factor Authentication

Enable 2FA when available (planned feature).

### Session Management

1. Log out from unused sessions
2. Review active sessions periodically
3. Revoke suspicious sessions

---

## Monitoring & Alerts

### Set Up Alerts

```yaml
# prometheus/alerts.yml
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"4.."}[5m]) > 0.1
  annotations:
    summary: "Possible attack - high 4xx error rate"

- alert: UnusualActivity
  expr: rate(wakelink_wake_commands_total[1h]) > 100
  annotations:
    summary: "Unusual wake command volume"
```

### Log Review

1. **Regular log review**: Check for anomalies
2. **Failed login alerts**: Monitor authentication failures
3. **Geolocation alerts**: Notify on access from new locations

---

## Incident Response

### If Token Compromised

1. **Immediately** regenerate token:
   ```bash
   wakelink token regenerate <device>
   ```
2. Update ESP device with new token
3. Review logs for unauthorized access
4. Consider factory reset of device

### If Server Compromised

1. Take server offline
2. Rotate all secrets (JWT, DB password)
3. Force logout all users
4. Regenerate all device tokens
5. Review access logs
6. Restore from clean backup

### If Device Compromised

1. Remove device from account
2. Factory reset device
3. Re-register with new token
4. Review network for lateral movement

---

## Checklist

### Before Deployment

- [ ] Change all default passwords
- [ ] Generate strong JWT secret
- [ ] Enable HTTPS/TLS
- [ ] Configure firewall
- [ ] Set up monitoring
- [ ] Enable rate limiting
- [ ] Disable debug mode
- [ ] Test backup/restore

### Regular Maintenance

- [ ] Update firmware monthly
- [ ] Review access logs weekly
- [ ] Rotate secrets quarterly
- [ ] Test recovery annually
- [ ] Security audit annually

---

## Compliance Considerations

### GDPR

- User data can be exported (`wakelink export`)
- User data can be deleted (`wakelink delete-account`)
- No unnecessary data collection

### Data Retention

- Logs retained for 30 days (configurable)
- Session data expires after 24 hours
- Wake history optional

---

## Security Contact

Found a vulnerability? Please report responsibly:

- **Email**: security@wakelink.io
- **PGP Key**: [keyserver link]
- **Bug Bounty**: See [SECURITY.md](https://github.com/wakelinkdev/wakelink/security/policy)

We aim to respond within 24 hours and fix critical issues within 7 days.
