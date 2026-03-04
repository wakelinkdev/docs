# Monitoring

Set up monitoring and alerting for your WakeLink server.

## Overview

WakeLink exposes Prometheus metrics for monitoring:

- Request rates and latencies
- Error rates by type
- Active connections
- Database pool status
- Custom business metrics

## Prometheus Setup

### docker-compose.monitoring.yml

```yaml
version: "3.8"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: wakelink-prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
    ports:
      - "9090:9090"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: wakelink-grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - "3000:3000"
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:latest
    container_name: wakelink-alertmanager
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

### prometheus/prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - /etc/prometheus/alerts.yml

scrape_configs:
  - job_name: 'wakelink-api'
    static_configs:
      - targets: ['api:8000']
    metrics_path: /metrics

  - job_name: 'wakelink-ws'
    static_configs:
      - targets: ['ws:8001']

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

---

## Available Metrics

### HTTP Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `http_requests_total` | Counter | Total requests by method, endpoint, status |
| `http_request_duration_seconds` | Histogram | Request latency |
| `http_requests_in_progress` | Gauge | Currently processing requests |

### WebSocket Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `wakelink_ws_connections_active` | Gauge | Active WebSocket connections |
| `wakelink_ws_messages_total` | Counter | Messages sent/received |
| `wakelink_ws_connection_duration_seconds` | Histogram | Connection lifetime |

### Business Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `wakelink_wake_commands_total` | Counter | Wake commands by status |
| `wakelink_devices_registered_total` | Counter | Device registrations |
| `wakelink_devices_online` | Gauge | Currently online devices |
| `wakelink_command_latency_seconds` | Histogram | End-to-end command latency |

### Database Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `db_pool_size` | Gauge | Connection pool size |
| `db_pool_checked_out` | Gauge | Connections in use |
| `db_pool_overflow` | Gauge | Overflow connections |

---

## Alerting Rules

### prometheus/alerts.yml

```yaml
groups:
  - name: wakelink
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) 
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate ({{ $value | humanizePercentage }})"
          description: "Error rate exceeds 5% for 5 minutes"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency (p99 > 2s)"

      # API down
      - alert: APIDown
        expr: up{job="wakelink-api"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "WakeLink API is down"

      # WebSocket gateway down
      - alert: WSGatewayDown
        expr: up{job="wakelink-ws"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "WebSocket gateway is down"

      # Low device connectivity
      - alert: LowDeviceConnectivity
        expr: |
          wakelink_devices_online / wakelink_devices_registered_total < 0.5
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Less than 50% of devices online"

      # Database connection pool exhausted
      - alert: DBPoolExhausted
        expr: db_pool_checked_out / db_pool_size > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Database connection pool near exhaustion"
```

---

## Alertmanager Configuration

### alertmanager/alertmanager.yml

```yaml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@wakelink.io'
  smtp_auth_username: 'alerts@wakelink.io'
  smtp_auth_password: 'app-password'

route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'critical'

receivers:
  - name: 'default'
    email_configs:
      - to: 'ops@wakelink.io'

  - name: 'critical'
    email_configs:
      - to: 'ops@wakelink.io'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'
        text: '{{ .CommonAnnotations.summary }}'
```

### Slack Integration

```yaml
receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#wakelink-alerts'
        send_resolved: true
        title: '{{ .Status | toUpper }}: {{ .CommonLabels.alertname }}'
        text: '{{ .CommonAnnotations.description }}'
```

### Telegram Integration

```yaml
receivers:
  - name: 'telegram'
    telegram_configs:
      - bot_token: 'YOUR_BOT_TOKEN'
        chat_id: 123456789
        message: '{{ .CommonAnnotations.summary }}'
```

---

## Grafana Dashboards

### WakeLink Overview Dashboard

```json
{
  "title": "WakeLink Overview",
  "panels": [
    {
      "title": "Request Rate",
      "type": "graph",
      "targets": [{
        "expr": "sum(rate(http_requests_total[5m]))"
      }]
    },
    {
      "title": "Error Rate",
      "type": "gauge",
      "targets": [{
        "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
      }]
    },
    {
      "title": "P99 Latency",
      "type": "stat",
      "targets": [{
        "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))"
      }]
    },
    {
      "title": "Online Devices",
      "type": "stat",
      "targets": [{
        "expr": "wakelink_devices_online"
      }]
    }
  ]
}
```

Import dashboards automatically:

```yaml
# grafana/provisioning/dashboards/default.yaml
apiVersion: 1
providers:
  - name: 'WakeLink'
    folder: ''
    type: file
    options:
      path: /etc/grafana/provisioning/dashboards
```

---

## Health Endpoints

### GET /health

Liveness probe — always returns 200 if service is running.

```json
{"status": "ok"}
```

### GET /health/ready

Readiness probe — checks dependencies.

```json
{
  "status": "ok",
  "checks": {
    "database": "ok",
    "redis": "ok"
  }
}
```

### GET /health/live

Detailed status with metrics.

```json
{
  "status": "ok",
  "version": "1.0.0",
  "uptime_seconds": 86400,
  "checks": {
    "database": {"status": "ok", "latency_ms": 2},
    "redis": {"status": "ok", "latency_ms": 1}
  },
  "metrics": {
    "devices_online": 42,
    "requests_per_minute": 120
  }
}
```

---

## Log Aggregation

### With Loki

```yaml
# docker-compose.monitoring.yml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - loki_data:/loki

  promtail:
    image: grafana/promtail:latest
    volumes:
      - ./promtail/config.yml:/etc/promtail/config.yml
      - /var/log:/var/log
```

### promtail/config.yml

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: wakelink
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        target_label: 'container'
```

---

Next: [Backup & Recovery →](backup.md)
