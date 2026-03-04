[🇬🇧 English](backup.md) | [🇷🇺 Русский](backup_RU.md)

# Резервное копирование и восстановление

Защитите данные WakeLink с помощью регулярного резервного копирования.

## Что нужно резервировать

| Компонент | Данные | Приоритет |
|-----------|--------|-----------|
| PostgreSQL | Пользователи, устройства, токены, логи | **Критично** |
| Redis | Сессии, ограничения запросов | Низкий (можно восстановить) |
| Конфигурация | `.env`, конфиг-файлы | **Критично** |
| SSL-сертификаты | Сертификаты и ключи TLS | **Критично** |

---

## Резервное копирование базы данных

### Ручное резервное копирование

```bash
# Full database dump
docker-compose exec db pg_dump -U wakelink wakelink > backup_$(date +%Y%m%d_%H%M%S).sql

# Compressed backup
docker-compose exec db pg_dump -U wakelink wakelink | gzip > backup_$(date +%Y%m%d).sql.gz

# Custom format (faster restore)
docker-compose exec db pg_dump -U wakelink -Fc wakelink > backup.dump
```

### Автоматизированный скрипт резервного копирования

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

### Расписание Cron

```bash
# Daily at 3 AM
0 3 * * * /opt/wakelink/backup.sh >> /var/log/wakelink-backup.log 2>&1
```

---

## Восстановление базы данных

### Из дампа SQL

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

### Из формата custom

```bash
docker-compose exec -T db pg_restore -U wakelink -d wakelink --clean backup.dump
```

### Восстановление на определённый момент времени

Для production-развёртываний включите архивирование WAL в PostgreSQL:

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

## Резервное копирование в облако

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

### Restic (зашифрованные резервные копии)

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

## Резервное копирование конфигурации

### Экспорт всей конфигурации

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

### Управление секретами

**Не загружайте `.env` с секретами в публичное хранилище!**

Используйте зашифрованные секреты:

```bash
# Encrypt .env
gpg --symmetric --cipher-algo AES256 .env
# Creates .env.gpg

# Decrypt
gpg --decrypt .env.gpg > .env
```

Или используйте менеджер секретов:
- HashiCorp Vault
- AWS Secrets Manager
- Docker Secrets

---

## Аварийное восстановление

### План восстановления

1. **Развернуть новый сервер**
   ```bash
   # Install Docker
   curl -fsSL https://get.docker.com | sh
   ```

2. **Восстановить конфигурацию**
   ```bash
   tar -xzf wakelink-config-backup.tar.gz -C /opt/wakelink
   ```

3. **Восстановить SSL-сертификаты**
   ```bash
   tar -xzf ssl-backup.tar.gz -C /etc/nginx/
   ```

4. **Запустить контейнеры** (создаётся пустая база данных)
   ```bash
   docker-compose up -d
   ```

5. **Восстановить базу данных**
   ```bash
   docker-compose stop api ws
   gunzip -c db_backup.sql.gz | docker-compose exec -T db psql -U wakelink wakelink
   docker-compose start api ws
   ```

6. **Проверить**
   ```bash
   curl https://your-domain.com/health
   ```

### Целевые показатели восстановления

| Сценарий | RTO | RPO |
|----------|-----|-----|
| Перезапуск контейнера | < 1 мин | 0 |
| Восстановление БД (100 МБ) | 5 мин | Последняя резервная копия |
| Полное восстановление сервера | 30 мин | Последняя резервная копия |

---

## Проверка резервных копий

### Регулярное тестирование восстановления

```bash
# Restore to test database
docker run --rm -v $(pwd):/backup postgres:15 \
  psql -h $TEST_DB_HOST -U wakelink -d wakelink_test < /backup/latest.sql

# Verify data
docker-compose exec db psql -U wakelink -c "SELECT count(*) FROM users;"
docker-compose exec db psql -U wakelink -c "SELECT count(*) FROM devices;"
```

### Мониторинг резервного копирования

Добавьте метрику успеха резервного копирования:

```bash
# In backup.sh
if [ $? -eq 0 ]; then
  echo "wakelink_backup_success 1" | curl --data-binary @- http://pushgateway:9091/metrics/job/backup
else
  echo "wakelink_backup_success 0" | curl --data-binary @- http://pushgateway:9091/metrics/job/backup
fi
```

Оповещение при сбое резервного копирования:

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

## Данные Redis

Данные Redis, как правило, временные (сессии, кэш), но при необходимости их тоже можно резервировать:

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

## Лучшие практики

1. **Тестируйте восстановление ежемесячно** — Резервные копии бесполезны, если восстановление невозможно
2. **Правило 3-2-1** — 3 копии, 2 разных носителя, 1 вне площадки
3. **Шифруйте всё** — Особенно облачные резервные копии
4. **Мониторьте задания резервного копирования** — Настройте оповещения о сбоях
5. **Документируйте восстановление** — Письменный план действий для экстренных случаев
6. **Автоматизируйте** — Ручное резервное копирование легко забыть

---

Назад к [Обзору сервера →](index.md)
