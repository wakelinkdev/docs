# Docker Deployment

Deploy WakeLink server with Docker Compose.

## Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- Linux server (Ubuntu 22.04 recommended)
- Domain name (for TLS)

## Quick Start

```bash
# Clone repository
git clone https://github.com/wakelinkdev/wakelink-server
cd wakelink-server

# Copy environment file
cp .env.example .env

# Generate secrets
echo "JWT_SECRET=$(openssl rand -hex 32)" >> .env
echo "DB_PASSWORD=$(openssl rand -hex 16)" >> .env

# Start services
docker-compose up -d

# Check status
docker-compose ps
docker-compose logs -f
```

## docker-compose.yml

```yaml
version: "3.8"

services:
  api:
    image: ghcr.io/wakelink/wakelink-server:latest
    container_name: wakelink-api
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://wakelink:${DB_PASSWORD}@db:5432/wakelink
      - REDIS_URL=redis://redis:6379/0
      - JWT_SECRET=${JWT_SECRET}
      - CORS_ORIGINS=https://your-domain.com
      - LOG_LEVEL=INFO
    depends_on:
      - db
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  ws:
    image: ghcr.io/wakelink/wakelink-server:latest
    container_name: wakelink-ws
    restart: unless-stopped
    command: ["python", "-m", "wakelink.ws_gateway"]
    ports:
      - "8001:8001"
    environment:
      - REDIS_URL=redis://redis:6379/0
      - LOG_LEVEL=INFO
    depends_on:
      - redis

  db:
    image: postgres:15-alpine
    container_name: wakelink-db
    restart: unless-stopped
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=wakelink
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=wakelink
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U wakelink"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: wakelink-redis
    restart: unless-stopped
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  nginx:
    image: nginx:alpine
    container_name: wakelink-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - ./nginx/www:/var/www/html:ro
    depends_on:
      - api
      - ws

volumes:
  postgres_data:
  redis_data:
```

## Environment Variables

Create `.env` file:

```bash
# Security (REQUIRED - generate unique values!)
JWT_SECRET=your-random-64-char-hex-string
DB_PASSWORD=your-random-32-char-hex-string

# Server
SERVER_HOST=0.0.0.0
SERVER_PORT=8000
WS_PORT=8001

# Database
DATABASE_URL=postgresql://wakelink:${DB_PASSWORD}@db:5432/wakelink

# Redis
REDIS_URL=redis://redis:6379/0

# Logging
LOG_LEVEL=INFO

# CORS (your frontend domain)
CORS_ORIGINS=https://your-domain.com

# Rate limiting
RATE_LIMIT_PER_MINUTE=60

# Optional: Email for notifications
SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=
EMAIL_FROM=noreply@your-domain.com
```

## Nginx Configuration

Create `nginx/nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream api {
        server api:8000;
    }

    upstream ws {
        server ws:8001;
    }

    server {
        listen 80;
        server_name your-domain.com;
        
        # Redirect to HTTPS
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name your-domain.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;

        # API routes
        location /api/ {
            proxy_pass http://api;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # WebSocket
        location /ws {
            proxy_pass http://ws;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_read_timeout 86400;
        }

        # Health check
        location /health {
            proxy_pass http://api;
        }

        # Metrics (restrict access!)
        location /metrics {
            allow 10.0.0.0/8;
            allow 172.16.0.0/12;
            allow 192.168.0.0/16;
            deny all;
            proxy_pass http://api;
        }
    }
}
```

## Database Initialization

First run automatically creates tables. For manual migration:

```bash
# Run migrations
docker-compose exec api alembic upgrade head

# Create admin user
docker-compose exec api python -m wakelink.cli create-admin \
  --email admin@example.com \
  --password your-password
```

## Useful Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f api
docker-compose logs -f ws

# Restart service
docker-compose restart api

# Stop all
docker-compose down

# Stop and remove volumes (DELETES DATA!)
docker-compose down -v

# Update images
docker-compose pull
docker-compose up -d

# Shell access
docker-compose exec api bash
docker-compose exec db psql -U wakelink

# Database backup
docker-compose exec db pg_dump -U wakelink wakelink > backup.sql

# Database restore
cat backup.sql | docker-compose exec -T db psql -U wakelink wakelink
```

## Production Checklist

Before going live:

- [ ] Generate unique JWT_SECRET and DB_PASSWORD
- [ ] Configure domain and SSL certificates
- [ ] Set appropriate CORS_ORIGINS
- [ ] Configure firewall (only 80/443 exposed)
- [ ] Set up log rotation
- [ ] Configure backups
- [ ] Set up monitoring
- [ ] Test health endpoints
- [ ] Load test with expected traffic

## Scaling

### Horizontal Scaling

```yaml
services:
  api:
    deploy:
      replicas: 3
```

With nginx load balancing:

```nginx
upstream api {
    least_conn;
    server api_1:8000;
    server api_2:8000;
    server api_3:8000;
}
```

### Resource Limits

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

---

Next: [Configuration Reference →](configuration.md)
