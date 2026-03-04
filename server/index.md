# Server Overview

Self-host your own WakeLink relay server for complete control over your data.

## Why Self-Host?

| Benefit | Description |
|---------|-------------|
| **Privacy** | Your data stays on your infrastructure |
| **Control** | Customize everything |
| **Reliability** | No dependency on third-party services |
| **No limits** | Unlimited devices and requests |
| **Custom domain** | Use your own domain and branding |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Compose                        │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Nginx     │  │   API       │  │   WS        │     │
│  │   Proxy     │──│   Server    │──│   Gateway   │     │
│  └─────────────┘  └──────┬──────┘  └──────┬──────┘     │
│                          │                │            │
│                   ┌──────▼──────┐  ┌──────▼──────┐     │
│                   │ PostgreSQL  │  │   Redis     │     │
│                   │             │  │             │     │
│                   └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

## Components

| Service | Purpose | Port |
|---------|---------|------|
| **API Server** | REST API, device management | 8000 |
| **WebSocket Gateway** | Device connections | 8001 |
| **PostgreSQL** | User/device data | 5432 |
| **Redis** | Sessions, pub/sub | 6379 |
| **Nginx** | Reverse proxy, TLS | 80/443 |

## Requirements

### Hardware (Minimum)

| Resource | Requirement |
|----------|-------------|
| CPU | 1 core |
| RAM | 1 GB |
| Storage | 10 GB |
| Network | 100 Mbps |

### Hardware (Recommended for 1000+ devices)

| Resource | Requirement |
|----------|-------------|
| CPU | 2+ cores |
| RAM | 4 GB |
| Storage | 50 GB SSD |
| Network | 1 Gbps |

### Software

- Docker 20.10+
- Docker Compose 2.0+
- Linux (Ubuntu 22.04 recommended)

---

## Quick Links

<div class="grid cards" markdown>

-   :material-docker:{ .lg .middle } __Docker Deployment__

    ---

    Get running in 5 minutes with Docker Compose

    [:octicons-arrow-right-24: Docker Guide](docker.md)

-   :material-cog:{ .lg .middle } __Configuration__

    ---

    Environment variables and settings

    [:octicons-arrow-right-24: Configuration](configuration.md)

-   :material-lock:{ .lg .middle } __SSL/TLS Setup__

    ---

    Secure your server with certificates

    [:octicons-arrow-right-24: SSL Guide](ssl.md)

-   :material-chart-line:{ .lg .middle } __Monitoring__

    ---

    Prometheus, Grafana, and alerting

    [:octicons-arrow-right-24: Monitoring](monitoring.md)

-   :material-backup-restore:{ .lg .middle } __Backup & Recovery__

    ---

    Protect your data

    [:octicons-arrow-right-24: Backup Guide](backup.md)

</div>

---

## Quick Start

```bash
# Clone repository
git clone https://github.com/wakelinkdev/wakelink-server
cd wakelink-server

# Configure
cp .env.example .env
# Edit .env with your settings

# Start
docker-compose up -d

# Check status
docker-compose ps
```

Server available at `http://localhost:8000`. See [Docker Guide](docker.md) for details.

---

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `POST /api/v1/auth/login` | User authentication |
| `GET /api/v1/devices` | List user's devices |
| `POST /api/v1/devices` | Register new device |
| `POST /api/v1/devices/{id}/wake` | Send wake command |
| `GET /api/v1/devices/{id}/status` | Device status |
| `GET /health` | Health check |
| `GET /metrics` | Prometheus metrics |

Full API documentation: [API Reference](../clients/api.md)

---

## Security Considerations

1. **Always use TLS** — Never expose server without HTTPS
2. **Strong secrets** — Generate random JWT secret and DB password
3. **Firewall** — Only expose ports 80/443
4. **Updates** — Keep Docker images updated
5. **Backups** — Regular database backups

See [Security Best Practices](../security/best-practices.md) for more.
