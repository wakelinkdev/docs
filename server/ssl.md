# SSL/TLS Setup

Secure your WakeLink server with TLS certificates.

## Why TLS?

- 🔒 **Encryption** — All traffic encrypted in transit
- ✅ **Authentication** — Verify server identity
- 🌐 **Required for WSS** — WebSocket Secure needs TLS
- 📱 **Mobile apps require HTTPS** — App stores mandate secure connections

## Option 1: Let's Encrypt (Recommended)

Free, automated certificates from Let's Encrypt.

### With Certbot

```bash
# Install certbot
sudo apt install certbot

# Get certificate (standalone)
sudo certbot certonly --standalone -d wakelink.example.com

# Or with webroot (if nginx running)
sudo certbot certonly --webroot -w /var/www/html -d wakelink.example.com
```

Certificates saved to:
- Certificate: `/etc/letsencrypt/live/wakelink.example.com/fullchain.pem`
- Private key: `/etc/letsencrypt/live/wakelink.example.com/privkey.pem`

### Docker with Certbot

```yaml
# docker-compose.yml
services:
  certbot:
    image: certbot/certbot
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    command: certonly --webroot -w /var/www/certbot --email your@email.com -d wakelink.example.com --agree-tos

  nginx:
    volumes:
      - ./certbot/conf:/etc/letsencrypt:ro
      - ./certbot/www:/var/www/certbot:ro
```

### Auto-Renewal

```bash
# Test renewal
sudo certbot renew --dry-run

# Cron job (runs twice daily)
# Already set up by certbot installation
cat /etc/cron.d/certbot
```

With Docker, add to crontab:
```bash
0 12 * * * docker-compose run --rm certbot renew && docker-compose restart nginx
```

---

## Option 2: Traefik (Auto-TLS)

Traefik handles TLS automatically.

```yaml
# docker-compose.yml
services:
  traefik:
    image: traefik:v2.10
    command:
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=your@email.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./letsencrypt:/letsencrypt

  api:
    image: ghcr.io/wakelink/wakelink-server:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`wakelink.example.com`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
```

---

## Option 3: Cloudflare (Proxy)

Use Cloudflare as a reverse proxy with automatic TLS.

### Setup

1. Add domain to Cloudflare
2. Update nameservers
3. Enable "Full (strict)" SSL mode
4. Create origin certificate

### Origin Certificate

1. Cloudflare Dashboard → SSL/TLS → Origin Server
2. Create Certificate
3. Download certificate and key
4. Save to server:
   ```bash
   # /etc/nginx/ssl/cloudflare.pem (certificate)
   # /etc/nginx/ssl/cloudflare.key (private key)
   ```

### Nginx Config

```nginx
server {
    listen 443 ssl;
    server_name wakelink.example.com;

    ssl_certificate /etc/nginx/ssl/cloudflare.pem;
    ssl_certificate_key /etc/nginx/ssl/cloudflare.key;

    # Only allow Cloudflare IPs
    # https://www.cloudflare.com/ips/
    allow 173.245.48.0/20;
    allow 103.21.244.0/22;
    # ... add all Cloudflare IPs
    deny all;
}
```

---

## Option 4: Self-Signed (Development Only)

⚠️ **Never use in production!**

```bash
# Generate self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt \
  -subj "/CN=localhost"
```

ESP devices need certificate skip flag:
```cpp
// Not recommended!
client.setInsecure();
```

---

## Nginx TLS Configuration

### Modern Configuration (TLS 1.3)

```nginx
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers off;
```

### Intermediate Configuration (TLS 1.2+)

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
```

### Full Example

```nginx
server {
    listen 443 ssl http2;
    server_name wakelink.example.com;

    # Certificates
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # Protocol and ciphers
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # Session settings
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # ... locations
}
```

---

## WebSocket TLS

WebSocket connections need TLS too (`wss://`).

### Nginx WebSocket Proxy

```nginx
location /ws {
    proxy_pass http://ws:8001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # Timeouts for long-lived connections
    proxy_connect_timeout 7d;
    proxy_send_timeout 7d;
    proxy_read_timeout 7d;
}
```

---

## Certificate Testing

### SSL Labs

Test your configuration:
```
https://www.ssllabs.com/ssltest/analyze.html?d=wakelink.example.com
```

Aim for **A+** rating.

### OpenSSL

```bash
# Check certificate
openssl s_client -connect wakelink.example.com:443 -servername wakelink.example.com

# Check expiry
echo | openssl s_client -connect wakelink.example.com:443 2>/dev/null | openssl x509 -noout -dates
```

### Curl

```bash
curl -vI https://wakelink.example.com/health
```

---

## Troubleshooting

### Certificate Not Trusted

- Check certificate chain (fullchain.pem includes intermediate)
- Verify domain matches certificate CN/SAN

### Connection Refused

- Check firewall allows port 443
- Verify nginx is running: `docker-compose ps`

### Mixed Content

- Ensure all resources use HTTPS
- Check API calls use `https://`

### WebSocket Fails

- Check nginx WebSocket configuration
- Verify timeout settings
- Check `wss://` not `ws://`

---

Next: [Monitoring →](monitoring.md)
