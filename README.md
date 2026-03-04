# 🔗 WakeLink

<p align="center">
  <strong>Система удалённого управления компьютерами с end-to-end шифрованием</strong>
</p>

<p align="center">
  <a href="https://github.com/wakelinkdev"><img src="https://img.shields.io/badge/org-wakelinkdev-purple.svg" alt="Organization"></a>
  <a href="https://github.com/wakelinkdev/ewsp-core"><img src="https://img.shields.io/badge/protocol-EWSP%20v1.0-blue.svg" alt="Protocol"></a>
  <a href="#"><img src="https://img.shields.io/badge/crypto-XChaCha20%20%2B%20HMAC--SHA256-green.svg" alt="Crypto"></a>
  <a href="https://wakelink-project.org/docs"><img src="https://img.shields.io/badge/docs-wakelink--project.org%2Fdocs-orange.svg" alt="Docs"></a>
  <a href="https://github.com/wakelinkdev/docs/actions"><img src="https://img.shields.io/badge/build-passing-brightgreen.svg" alt="Build"></a>
</p>

<p align="center">
  <a href="https://github.com/wakelinkdev/ewsp-core">ewsp-core</a> •
  <a href="https://github.com/wakelinkdev/firmware">firmware</a> •
  <a href="https://github.com/wakelinkdev/cli">cli</a> •
  <a href="https://github.com/wakelinkdev/android">android</a> •
  <a href="https://github.com/wakelinkdev/multiplatform">multiplatform</a>
</p>

> ⚠️ **Примечание:** Это локальная development-структура. При публикации проект будет разделён на отдельные репозитории в организации [wakelinkdev](https://github.com/wakelinkdev).

---

## 📋 Содержание

- [Обзор](#-обзор)
- [Архитектура](#-архитектура)
- [Репозитории](#-репозитории)
- [Протокол EWSP v1.0](#-протокол-ewsp-v10)
- [Quick Start](#-quick-start)
- [Build Instructions](#-build-instructions)
- [Security Features](#-security-features)
- [Документация](#-документация)
- [Contributing](#-contributing)
- [License](#-license)

---

## 📖 Обзор

WakeLink — система удалённого управления компьютерами с функциями:

- 🖥️ **Wake-on-LAN** — пробуждение устройств по сети
- 🔐 **End-to-end шифрование** — XChaCha20 + HMAC-SHA256
- 🌐 **Локальный и облачный режимы** — TCP:99 или WSS (TLS)
- ⛓️ **Blockchain packet chaining** — защита от replay-атак
- 🔑 **Forward secrecy** — сессионное шифрование с ротацией ключей

Документация публикуется на [wakelink-project.org/docs](https://wakelink-project.org/docs).

---

## 🏗️ Архитектура

```
┌─────────────────────────────────────────────────────────────────┐
│                         КЛИЕНТЫ                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │  Python  │  │ Android  │  │   iOS    │  │     Desktop      │ │
│  │   CLI    │  │   App    │  │   App    │  │  (Win/Mac/Linux) │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬─────────┘ │
│       └─────────────┴──────┬──────┴─────────────────┘           │
│                            │                                     │
│                    ┌───────┴───────┐                            │
│                    │   ПРОТОКОЛ    │                            │
│                    │   EWSP v2.0   │                            │
│                    │ (XChaCha20 +  │                            │
│                    │  HMAC-SHA256) │                            │
│                    └───────┬───────┘                            │
└────────────────────────────┼────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
       ┌──────┴──────┐              ┌───────┴───────┐
       │   ЛОКАЛЬНО   │              │    ОБЛАКО     │
       │   TCP:99     │              │  WSS (TLS)    │
       └──────┬──────┘              └───────┬───────┘
              │                             │
              │                    ┌────────┴────────┐
              │                    │  WakeLink       │
              │                    │  Server         │
              │                    │  (FastAPI)      │
              │                    │  + PostgreSQL   │
              │                    │  + Redis        │
              │                    └────────┬────────┘
              │                             │
              └─────────────┬───────────────┘
                            │
                    ┌───────┴───────┐
                    │   ПРОШИВКА    │
                    │  ESP8266/32   │
                    │  WakeLink     │
                    │  Firmware     │
                    └───────────────┘
```

---

## 📦 Репозитории

| Репозиторий | Описание | Статус |
|-------------|----------|--------|
| [**.github**](.github/) | Профиль организации, шаблоны | ✅ |
| [**ewsp-core**](ewsp-core/) | C библиотека криптографии и протокола (EWSP) | ✅ |
| [**firmware**](firmware/) | Прошивка ESP8266/ESP32 | ✅ |
| [**cli**](cli/) | Python CLI клиент | ✅ |
| [**android**](android/) | Android приложение (Kotlin) | ✅ |
| [**multiplatform**](multiplatform/) | KMP (iOS/Android/Desktop) | 🔄 |
| [**docs**](docs/) | Документация (MkDocs) | ✅ |
| **server** | Backend сервер (FastAPI) | 🔒 Private |

---

## 🔒 Протокол EWSP v1.0

> 📖 **Полная спецификация:** [EWSP Protocol Specification v1.0](EWSP_PROTOCOL_SPEC_v1.0.md)

### Криптография

| Алгоритм | Назначение | Стандарт |
|----------|------------|----------|
| **XChaCha20** | Шифрование данных | RFC 7539 + extended nonce |
| **HMAC-SHA256** | Аутентификация и MAC | RFC 2104 |
| **HKDF-SHA256** | Деривация ключей | RFC 5869 |
| **SHA-256** | Хэширование | FIPS 180-4 |

### Key Separation

Из device_token создаются раздельные ключи:
- `enc_key` = HKDF(token, "wakelink_encryption_v2") — только шифрование
- `auth_key` = HKDF(token, "wakelink_authentication_v2") — только MAC

### Формат пакета

```
┌─────────────────────────────────────────────────────────────┐
│ VERSION (4) │ DEVICE_ID (8) │ SEQ (4) │ PREV_HASH (32) │ ...│
├─────────────────────────────────────────────────────────────┤
│ ... │ REQUEST_ID (8) │ SIGNATURE (64) │ PAYLOAD │ HASH (64) │
└─────────────────────────────────────────────────────────────┘

Payload = LENGTH (2 bytes) + CIPHERTEXT + NONCE (24 bytes)
```

### Blockchain Chaining

Каждый пакет содержит hash предыдущего пакета, образуя цепочку:
- Защита от replay-атак
- Гарантия порядка пакетов
- Детектирование пропущенных сообщений

---

## 🚀 Quick Start

### 1. Запуск сервера

```bash
cd server

# Сгенерируйте SECRET_KEY
export SECRET_KEY=$(python -c "import secrets; print(secrets.token_urlsafe(48))")

# Запуск через Docker
docker-compose up -d
```

### 2. Подключение клиента

```bash
cd cli
pip install -r requirements.txt

# Подключение к локальному устройству
python wakelink.py connect 192.168.1.100 --token YOUR_DEVICE_TOKEN

# Пробуждение ПК
python wakelink.py wake --mac AA:BB:CC:DD:EE:FF
```

### 3. Прошивка устройства

1. Откройте `firmware/WakeLink/WakeLink.ino` в Arduino IDE
2. Установите board: ESP8266 или ESP32
3. Загрузите прошивку
4. Подключитесь к WiFi `WakeLink-Setup` для первичной настройки

---

## 🛠️ Build Instructions

### ewsp-core (C библиотека)

```bash
cd ewsp-core
mkdir build && cd build
cmake ..
make
make test
```

### server (Python)

```bash
cd server
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python run.py
```

### firmware (Arduino)

Требуется:
- Arduino IDE 2.0+
- ESP8266 или ESP32 board package
- Библиотеки: ArduinoJson, WebSockets

```
1. File → Open → firmware/WakeLink/WakeLink.ino
2. Tools → Board → ESP8266/ESP32
3. Sketch → Upload
```

### cli (Python)

```bash
cd cli
pip install -r requirements.txt
python wakelink.py --help
```

### android (Kotlin)

```bash
cd android
./gradlew assembleDebug
```

---

## 🛡️ Security Features

### ✅ Реализовано

| Feature | Описание |
|---------|----------|
| **CSPRNG** | Криптографически безопасный RNG на всех платформах |
| **Key Separation** | Раздельные ключи для шифрования и MAC |
| **Constant-Time Compare** | Защита от timing-атак при сравнении hash/MAC |
| **Forward Secrecy** | Сессионные ключи с ротацией |
| **Replay Protection** | Blockchain packet chaining + sequence numbers |
| **OTA Signature** | HMAC-SHA256 подпись firmware updates |
| **Legacy Protection** | Устаревшие endpoints отключены по умолчанию |

### 🔒 Рекомендации для Production

1. **Сгенерируйте уникальный SECRET_KEY:**
   ```bash
   python -c "import secrets; print(secrets.token_urlsafe(48))"
   ```

2. **Используйте TLS** для облачных подключений

3. **Включите OTA signature verification** (`WAKELINK_OTA_SECURE` в platform.h)

4. **Не коммитьте .env файлы** с секретами

---

## 📚 Документация

Документация построена на [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) и публикуется на [wakelink-project.org/docs](https://wakelink-project.org/docs).

### Структура

```
docs/
├── index.md                      # Главная страница
├── getting-started/
│   ├── index.md                  # Введение
│   ├── requirements.md           # Требования
│   ├── quickstart.md             # Быстрый старт
│   └── architecture.md           # Архитектура
├── firmware/
│   ├── index.md
│   ├── configuration.md
│   ├── flashing.md
│   ├── ota.md
│   └── troubleshooting.md
├── clients/
│   ├── index.md
│   ├── cli.md
│   ├── android.md
│   └── api.md
├── server/
│   ├── index.md
│   ├── docker.md
│   ├── configuration.md
│   ├── ssl.md
│   ├── monitoring.md
│   └── backup.md
├── security/
│   ├── index.md
│   ├── encryption.md
│   ├── ewsp.md
│   └── best-practices.md
├── reference/
│   ├── configuration.md
│   ├── error-codes.md
│   ├── changelog.md
│   └── faq.md
├── EWSP_PROTOCOL_SPEC_v1.0.md    # Полная спецификация EWSP
└── flash/
    └── index.html                # Web-флешер
```

### Локальная разработка

```bash
# Установка
pip install mkdocs-material mkdocs-minify-plugin

# Запуск dev-сервера
mkdocs serve
# Открыть http://localhost:8000

# Сборка
mkdocs build
# Результат в site/
```

### Деплой

Документация автоматически деплоится на GitHub Pages при пуше в `main`:

```yaml
# .github/workflows/docs.yml
name: Deploy Docs
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - run: pip install mkdocs-material mkdocs-minify-plugin
      - run: mkdocs gh-deploy --force
```

---

## 🤝 Contributing

1. Fork репозитория
2. Создайте feature branch: `git checkout -b feature/amazing-feature`
3. Commit изменений: `git commit -m 'Add amazing feature'`
4. Push в branch: `git push origin feature/amazing-feature`
5. Создайте Pull Request

### Стиль документации

- Используйте Markdown (CommonMark)
- Один H1 (`#`) на страницу
- Код в блоках с указанием языка
- Изображения в `docs/assets/`

---

## 📄 License

Проект распространяется под кастомной лицензией. См. [LICENSE](LICENSE) для деталей.

Для коммерческого использования обратитесь к автору.

---

## 🔗 Связанные репозитории

| Репозиторий | Описание |
|-------------|----------|
| [ewsp-core](https://github.com/wakelinkdev/ewsp-core) | Криптографическое ядро |
| [firmware](https://github.com/wakelinkdev/firmware) | ESP прошивка |
| [cli](https://github.com/wakelinkdev/cli) | CLI клиент |
| [android](https://github.com/wakelinkdev/android) | Android приложение |
| [multiplatform](https://github.com/wakelinkdev/multiplatform) | KMP приложение |

---

<p align="center">
  <sub>Часть проекта <a href="https://github.com/wakelinkdev">WakeLink</a> by <a href="https://github.com/deadboizxc">deadboizxc</a></sub>
</p>
<p align="center">
  <strong>Made with ❤️ by deadboizxc</strong>
</p>
