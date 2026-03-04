# Backup & Recovery

Protect your WakeLink data with regular backups.

## What to Backup

| Component | Data | Priority |
|-----------|------|----------|
| PostgreSQL | Users, devices, tokens, logs | **Critical** |
| Redis | Sessions, rate limits | Low (can regenerate) |
| Configuration | `.env`, config files | **Critical** |
| SSL Certificates | TLS certs and keys | **Critical** |

---

## Database Backup

### Manual Backup

```bash
# Full database dump
docker-compose exec db pg_dump -U wakelink wakelink > backup_$(date +%Y%m%d_%H%M%S).sql

# Compressed backup
docker-compose exec db pg_dump -U wakelink wakelink | gzip > backup_$(date +%Y%m%d).sql.gz

# Custom format (faster restore)
docker-compose exec db pg_dump -U wakelink -Fc wakelink > backup.dump
```

### Automated Backup Script

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/opt/wakelink/backups"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup
docker-compose exec -T db pg_dump -U wakelink wakelink | gzip > $BACKUP_DIR/db_$DATE.sql.gz

# Configuration backup
tar -czf $BACKUP_DIR/config_$DATE.tar.gz .env nginx/ prometheus/ grafana/

# SSL certificates backup
tar -czf $BACKUP_DIR/ssl_$DATE.tar.gz /etc/nginx/ssl/

# Cleanup old backups
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete

# Optional: Upload to S3
# aws s3 cp $BACKUP_DIR/db_$DATE.sql.gz s3://wakelink-backups/

echo "Backup completed: $DATE"
```

### Cron Schedule

```bash
# Daily at 3 AM
0 3 * * * /opt/wakelink/backup.sh >> /var/log/wakelink-backup.log 2>&1
```

---

## Database Restore

### From SQL Dump

```bash
# Stop services
docker-compose stop api ws

# Restore
cat backup.sql | docker-compose exec -T db psql -U wakelink wakelink

# Or from compressed
gunzip -c backup.sql.gz | docker-compose exec -T db psql -U wakelink wakelink

# Start services
docker-compose start api ws
```

### From Custom Format

```bash
docker-compose exec -T db pg_restore -U wakelink -d wakelink --clean backup.dump
```

### Point-in-Time Recovery

For production deployments, enable PostgreSQL WAL archiving:

```yaml
# docker-compose.yml
services:
  db:
    environment:
      - POSTGRES_INITDB_ARGS=--wal-segsize=16
    command: >
      postgres
      -c wal_level=replica
      -c archive_mode=on
      -c archive_command='cp %p /backups/wal/%f'
    volumes:
      - ./backups/wal:/backups/wal
```

---

## Cloud Backup

### AWS S3

```bash
# Install AWS CLI
pip install awscli

# Configure credentials
aws configure

# Backup script addition
aws s3 sync $BACKUP_DIR s3://your-bucket/wakelink-backups/

# Restore from S3
aws s3 cp s3://your-bucket/wakelink-backups/db_20260101.sql.gz ./
```

### Backblaze B2

```bash
# Install B2 CLI
pip install b2

# Authorize
b2 authorize-account $APP_KEY_ID $APP_KEY

# Upload
b2 upload-file wakelink-backups backup.sql.gz backups/db_$(date +%Y%m%d).sql.gz
```

### Restic (Encrypted Backups)

```bash
# Initialize repository
restic -r s3:s3.amazonaws.com/wakelink-backups init

# Backup
restic -r s3:s3.amazonaws.com/wakelink-backups backup /opt/wakelink/backups

# List snapshots
restic -r s3:s3.amazonaws.com/wakelink-backups snapshots

# Restore
restic -r s3:s3.amazonaws.com/wakelink-backups restore latest --target /restore
```

---

## Configuration Backup

### Export All Configuration

```bash
# Create config package
tar -czf wakelink-config-$(date +%Y%m%d).tar.gz \
  .env \
  docker-compose.yml \
  docker-compose.*.yml \
  nginx/ \
  prometheus/ \
  grafana/ \
  alertmanager/
```

### Secrets Management

**Don't backup .env with secrets to public storage!**

Use encrypted secrets:

```bash
# Encrypt .env
gpg --symmetric --cipher-algo AES256 .env
# Creates .env.gpg

# Decrypt
gpg --decrypt .env.gpg > .env
```

Or use a secrets manager:
- HashiCorp Vault
- AWS Secrets Manager
- Docker Secrets

---

## Disaster Recovery

### Recovery Plan

1. **Provision new server**
   ```bash
   # Install Docker
   curl -fsSL https://get.docker.com | sh
   ```

2. **Restore configuration**
   ```bash
   tar -xzf wakelink-config-backup.tar.gz -C /opt/wakelink
   ```

3. **Restore SSL certificates**
   ```bash
   tar -xzf ssl-backup.tar.gz -C /etc/nginx/
   ```

4. **Start containers** (creates empty database)
   ```bash
   docker-compose up -d
   ```

5. **Restore database**
   ```bash
   docker-compose stop api ws
   gunzip -c db_backup.sql.gz | docker-compose exec -T db psql -U wakelink wakelink
   docker-compose start api ws
   ```

6. **Verify**
   ```bash
   curl https://your-domain.com/health
   ```

### Recovery Time Objectives

| Scenario | RTO | RPO |
|----------|-----|-----|
| Container restart | < 1 min | 0 |
| Database restore (100MB) | 5 min | Last backup |
| Full server rebuild | 30 min | Last backup |

---

## Backup Verification

### Test Restore Regularly

```bash
# Restore to test database
docker run --rm -v $(pwd):/backup postgres:15 \
  psql -h $TEST_DB_HOST -U wakelink -d wakelink_test < /backup/latest.sql

# Verify data
docker-compose exec db psql -U wakelink -c "SELECT count(*) FROM users;"
docker-compose exec db psql -U wakelink -c "SELECT count(*) FROM devices;"
```

### Backup Monitoring

Add backup success metric:

```bash
# In backup.sh
if [ $? -eq 0 ]; then
  echo "wakelink_backup_success 1" | curl --data-binary @- http://pushgateway:9091/metrics/job/backup
else
  echo "wakelink_backup_success 0" | curl --data-binary @- http://pushgateway:9091/metrics/job/backup
fi
```

Alert on backup failure:

```yaml
# prometheus/alerts.yml
- alert: BackupFailed
  expr: wakelink_backup_success == 0
  for: 1h
  labels:
    severity: warning
  annotations:
    summary: "Backup failed"
```

---

## Redis Data

Redis data is generally ephemeral (sessions, cache), but you can backup if needed:

```bash
# Trigger RDB snapshot
docker-compose exec redis redis-cli BGSAVE

# Copy dump file
docker cp wakelink-redis:/data/dump.rdb ./redis-backup.rdb

# Restore
docker cp ./redis-backup.rdb wakelink-redis:/data/dump.rdb
docker-compose restart redis
```

---

## Best Practices

1. **Test restores monthly** — Backups aren't useful if you can't restore
2. **3-2-1 rule** — 3 copies, 2 different media, 1 offsite
3. **Encrypt everything** — Especially cloud backups
4. **Monitor backup jobs** — Alert on failures
5. **Document recovery** — Written runbook for emergencies
6. **Automate** — Manual backups are forgotten

---

Back to [Server Overview →](index.md)
