[🇬🇧 English](EWSP_PROTOCOL_SPEC_v1.0.md) | [🇷🇺 Русский](EWSP_PROTOCOL_SPEC_v1.0_RU.md)

# Спецификация протокола EWSP v1.0

**Зашифрованный протокол безопасного Wake-on-LAN**

| Поле          | Значение               |
|---------------|------------------------|
| Версия        | 1.0                    |
| Статус        | Стабильный             |
| Дата          | Февраль 2026           |
| Автор         | Команда WakeLink       |
| Лицензия      | CC BY-SA 4.0           |

---

## Содержание

1. [Введение](#1-введение)
2. [Модель безопасности](#2-модель-безопасности)
3. [Криптографические примитивы](#3-криптографические-примитивы)
4. [Управление ключами](#4-управление-ключами)
5. [Структура пакетов](#5-структура-пакетов)
6. [Блокчейн-цепочка](#6-блокчейн-цепочка)
7. [Управление сессиями](#7-управление-сессиями)
8. [Протокол команд](#8-протокол-команд)
9. [Режимы транспорта](#9-режимы-транспорта)
10. [Соображения безопасности](#10-соображения-безопасности)
11. [Приложение A: Тестовые векторы](#приложение-a-тестовые-векторы)
12. [Приложение B: Референсные реализации](#приложение-b-референсные-реализации)

---

## 1. Введение

### 1.1 Назначение

EWSP (Encrypted Wake-on-LAN Secure Protocol) — безопасный протокол связи, разработанный для удалённого управления устройствами с основным акцентом на операции Wake-on-LAN. Протокол обеспечивает:

- **Сквозное шифрование** с использованием XChaCha20-Poly1305 AEAD
- **Аутентификацию сообщений** через HMAC-SHA256
- **Защиту от воспроизведения** посредством цепочки пакетов в стиле блокчейна
- **Прямую секретность** за счёт раскрутки сессионного ключа
- **Взаимную аутентификацию** между клиентом и устройством

### 1.2 Цели

| Цель                       | Решение                                              |
|----------------------------|------------------------------------------------------|
| Конфиденциальность         | Шифрование XChaCha20-Poly1305 AEAD                   |
| Целостность                | Подписи HMAC-SHA256                                  |
| Аутентификация             | Общий токен устройства + взаимный запрос-ответ       |
| Защита от воспроизведения  | Блокчейн-цепочка с монотонными порядковыми номерами  |
| Прямая секретность         | Формирование ключей HKDF + раскрутка ключей          |
| Устойчивость к DoS         | Ограничение частоты + пределы прыжка по последовательности |

### 1.3 Обзор архитектуры

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          WakeLink System                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌──────────┐                                      ┌──────────────┐     │
│   │  Client  │◄──────── EWSP v1.0 ────────────────►│   Firmware   │     │
│   │ (Python/ │         TCP Direct                  │  (ESP8266/   │     │
│   │ Android/ │         or                          │   ESP32)     │     │
│   │ Desktop) │         WSS Relay                   │              │     │
│   └──────────┘                                      └──────────────┘     │
│        │                                                  │              │
│        │                 ┌──────────┐                     │              │
│        └────────────────►│  Server  │◄────────────────────┘              │
│                          │  (Relay) │                                    │
│                          └──────────┘                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.4 Уровни протокола

```
┌─────────────────────────────────────────┐
│           Application Layer             │   Commands (wake, ping, status)
├─────────────────────────────────────────┤
│            Session Layer                │   Key ratcheting, counters
├─────────────────────────────────────────┤
│            Packet Layer                 │   Blockchain chaining, JSON envelope
├─────────────────────────────────────────┤
│           Crypto Layer                  │   XChaCha20-Poly1305, HMAC, HKDF
├─────────────────────────────────────────┤
│          Transport Layer                │   TCP / WebSocket Secure (WSS)
└─────────────────────────────────────────┘
```

---

## 2. Модель безопасности

### 2.1 Модель угроз

EWSP v1.0 защищает от:

| Угроза                        | Защита                                               |
|-------------------------------|------------------------------------------------------|
| Прослушивание                 | Шифрование XChaCha20-Poly1305                        |
| Подделка сообщений            | Подписи HMAC-SHA256 + теги AEAD                      |
| Атаки воспроизведения         | Блокчейн-цепочка + монотонная последовательность     |
| Атака «человек посередине»    | Предварительно распределённый токен + взаимная аутентификация |
| Перебор                       | 256-битные ключи + ограничение частоты               |
| Атаки по побочным каналам     | Функции сравнения за константное время               |
| Компрометация ключей          | Прямая секретность через раскрутку ключей            |
| DoS через прыжок по последовательности | Максимальный предел прыжка (100)            |

### 2.2 Допущения о доверии

1. **Токен устройства** безопасно предоставляется при сопряжении
2. **Транспортный уровень** (TLS 1.3 для WSS) обеспечивает безопасность при передаче
3. **Генератор случайных чисел** является криптографически стойким (CSPRNG)
4. **Системное время** достаточно точно (±1 день для истечения сессии)

### 2.3 Уровни безопасности

| Уровень | Размер ключа | Биты безопасности | Область применения       |
|---------|--------------|-------------------|--------------------------|
| EWSP    | 256 бит      | 128 бит           | По умолчанию для всех устройств |

---

## 3. Криптографические примитивы

### 3.1 Хеш-функции

#### SHA-256 (FIPS 180-4)

Используется для:
- Хеширования пакетов (блокчейн)
- Формирования ключей (HKDF)
- Построения HMAC

```
Output: 32 bytes (256 bits)
```

### 3.2 Аутентификация сообщений

#### HMAC-SHA256 (RFC 2104)

Используется для:
- Проверки подписи пакетов
- Подписи прошивки OTA

```
Key:    32 bytes
Input:  Variable length
Output: 32 bytes
```

**Формат подписи:**

```
signature = HMAC-SHA256(auth_key, "v" || id || seq || prev || payload)
```

### 3.3 Формирование ключей

#### HKDF-SHA256 (RFC 5869)

Используется для:
- Получения ключей шифрования и аутентификации из мастер-токена
- Формирования сессионных ключей
- Раскрутки ключей

```
Salt:   Optional (32 bytes or empty)
IKM:    Input key material
Info:   Context/application-specific info
Output: Variable length key material
```

**Разделение ключей:**

```
enc_key  = HKDF(token, "", "wakelink_encryption_v1", 32)
auth_key = HKDF(token, "", "wakelink_authentication_v1", 32)
```

### 3.4 Шифрование

#### XChaCha20-Poly1305 AEAD (RFC 7539 + XSalsa extension)

Используется для:
- Шифрования нагрузки
- Сообщений с шифрованием на уровне сессии

```
Key:            32 bytes
Nonce:          24 bytes (extended nonce)
Plaintext:      Variable length
Associated Data: Variable length (packet header fields)
Ciphertext:     len(plaintext) bytes
Tag:            16 bytes (appended to ciphertext)
```

**Построение AEAD:**

```
subkey = HChaCha20(key, nonce[0:16])
nonce' = 0x00000000 || nonce[16:24]
ciphertext = ChaCha20(subkey, nonce', 1, plaintext)
tag = Poly1305(poly_key, aad || ciphertext || lengths)
```

#### XChaCha20 (потоковый шифр)

Используется для:
- Устаревшего шифрования нагрузки (с отдельным HMAC)
- Контекстов без AEAD

```
Key:    32 bytes
Nonce:  24 bytes
Counter: 32 bits (starting at 0 or 1)
```

### 3.5 Генерация случайных чисел

Все случайные значения ДОЛЖНЫ генерироваться с использованием криптографически стойкого ГПСЧ:

| Платформа     | Источник                          |
|---------------|-----------------------------------|
| Windows       | CryptGenRandom / BCryptGenRandom  |
| Linux         | /dev/urandom / getrandom()        |
| ESP8266/ESP32 | esp_random()                      |
| Android       | SecureRandom                      |
| iOS           | SecRandomCopyBytes                |

---

## 4. Управление ключами

### 4.1 Токен устройства

Токен устройства — это секрет длиной 32+ символа, разделяемый между устройством и авторизованными клиентами.

**Требования:**
- Минимум 32 символа
- Генерируется с использованием CSPRNG
- Хранится безопасно (зашифрован на устройстве, в связке ключей на мобильном)
- Никогда не передаётся в открытом виде

**Предоставление:**
1. Устройство генерирует токен при первоначальной настройке
2. Отображается в виде QR-кода на портале настройки
3. Клиент сканирует QR и сохраняет токен

### 4.2 Дерево формирования ключей

```
                    Device Token (master secret)
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         enc_key         auth_key        session_key
     (encryption)    (authentication)   (session mgmt)
              │
    ┌─────────┴─────────┐
    ▼                   ▼
 chacha_key        ota_key
(XChaCha20)    (OTA signing)
```

**Строки контекста HKDF:**

| Назначение ключа   | Строка info HKDF                    |
|--------------------|-------------------------------------|
| Шифрование         | `wakelink_encryption_v1`            |
| Аутентификация     | `wakelink_authentication_v1`        |
| Сессия             | `wakelink_session_v1`               |
| Подпись OTA        | `wakelink_ota_signing_v1`           |
| Привязка           | `wakelink_binding_v1`               |

### 4.3 Ротация ключей

Ротация ключей поддерживается через команду `key_rotate`:

1. Сервер генерирует новый зашифрованный токен
2. Клиент отправляет команду `update_token`
3. Устройство расшифровывает и сохраняет новый токен
4. Устройство хранит предыдущий токен 24 ч (откат)
5. Все сессии используют новый ключ после ротации

---

## 5. Структура пакетов

### 5.1 Внешний пакет (JSON)

Каждый пакет EWSP v1.0 — это JSON-объект следующей структуры:

```json
{
  "v": "1.0",
  "id": "WL35080814",
  "seq": 42,
  "prev": "a1b2c3d4e5f6...",
  "p": "base64_encrypted_payload...",
  "sig": "hex_hmac_signature..."
}
```

| Поле   | Тип    | Размер       | Описание                                      |
|--------|--------|--------------|-----------------------------------------------|
| `v`    | string | 3 символа    | Версия протокола ("1.0")                      |
| `id`   | string | 10 символов  | Идентификатор устройства (WL + 8 hex)         |
| `seq`  | uint64 | 1-20 цифр    | Монотонный порядковый номер                   |
| `prev` | string | 64 hex       | SHA256-хеш предыдущего пакета                 |
| `p`    | string | переменный   | Зашифрованная нагрузка в base64               |
| `sig`  | string | 64 hex       | Подпись HMAC-SHA256                           |

**Порядок полей:** поля ДОЛЖНЫ появляться в указанном порядке для канонической сериализации.

### 5.2 Внутренний пакет (расшифрованный JSON)

Расшифрованная нагрузка содержит:

```json
{
  "cmd": "wake",
  "d": {
    "mac": "AA:BB:CC:DD:EE:FF",
    "port": 9
  },
  "rid": "X7K2M9P1"
}
```

| Поле  | Тип    | Описание                                    |
|-------|--------|---------------------------------------------|
| `cmd` | string | Имя команды (см. §8)                        |
| `d`   | object | Данные, специфичные для команды             |
| `rid` | string | Идентификатор запроса (8 буквенно-цифровых) |

### 5.3 Схема пакета

```
┌─────────────────────────────────────────────────────────────────┐
│                        OUTER PACKET (JSON)                       │
├─────────────────────────────────────────────────────────────────┤
│  {                                                               │
│    "v": "1.0",              ◄─── Protocol version               │
│    "id": "WL35080814",      ◄─── Device identifier              │
│    "seq": 42,               ◄─── Sequence number                │
│    "prev": "a1b2c3...",     ◄─── Previous packet hash (64 hex)  │
│    "p": "base64...",        ◄─── Encrypted inner packet         │
│    "sig": "hmac64..."       ◄─── HMAC signature (64 hex)        │
│  }                                                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ Decrypt(enc_key, nonce, p)
┌─────────────────────────────────────────────────────────────────┐
│                        INNER PACKET (JSON)                       │
├─────────────────────────────────────────────────────────────────┤
│  {                                                               │
│    "cmd": "wake",           ◄─── Command name                   │
│    "d": {                   ◄─── Command data                   │
│      "mac": "AA:BB:CC:DD:EE:FF"                                  │
│    },                                                            │
│    "rid": "X7K2M9P1"        ◄─── Request ID                     │
│  }                                                               │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Расчёт подписи

Подпись охватывает все маршрутизирующие поля для предотвращения подделки:

```
message = v || "|" || id || "|" || seq || "|" || prev || "|" || p
signature = HMAC-SHA256(auth_key, message)
```

**При проверке НЕОБХОДИМО:**
1. Разобрать внешний JSON
2. Восстановить сообщение подписи из полей
3. Вычислить HMAC с auth_key
4. Использовать сравнение за константное время с предоставленной подписью

### 5.5 Шифрование нагрузки

**Шифрование (клиент → устройство):**

```python
nonce = random_bytes(24)
ciphertext = XChaCha20(enc_key, nonce, 0, inner_json.encode())
payload = base64_encode(nonce + ciphertext)
```

**Дешифрование (устройство):**

```python
raw = base64_decode(payload)
nonce = raw[0:24]
ciphertext = raw[24:]
inner_json = XChaCha20(enc_key, nonce, 0, ciphertext)
```

---

## 6. Блокчейн-цепочка

### 6.1 Концепция

Каждый пакет содержит SHA256-хеш предыдущего пакета того же направления, образуя цепочку наподобие блокчейна. Это обеспечивает:

- **Защиту от воспроизведения**: старые пакеты не могут быть воспроизведены
- **Гарантию порядка**: пакеты должны поступать строго по очереди
- **Обнаружение подделки**: изменение любого пакета нарушает цепочку

### 6.2 Двунаправленные цепочки

Каждое направление имеет независимую цепочку:

```
TX Chain (Client → Device):
  Genesis ──► Pkt1 ──► Pkt2 ──► Pkt3 ──► ...
     │          │         │
    seq=0     seq=1    seq=2

RX Chain (Device → Client):
  Genesis ──► Pkt1 ──► Pkt2 ──► Pkt3 ──► ...
     │          │         │
    seq=0     seq=1    seq=2
```

### 6.3 Начальное состояние (Genesis)

```
GENESIS_HASH = "0000000000000000000000000000000000000000000000000000000000000000"
GENESIS_SEQ  = 0
```

Обе цепочки начинаются с начального состояния. Первый пакет имеет:
- `seq = 1`
- `prev = GENESIS_HASH`

### 6.4 Алгоритм обновления цепочки

**Отправка пакета:**

```python
def send_packet(ctx, command, data):
    seq = ctx.tx.sequence + 1
    prev = ctx.tx.last_hash
    
    packet = create_packet(seq, prev, command, data)
    packet_hash = SHA256(packet.to_json())
    
    ctx.tx.sequence = seq
    ctx.tx.last_hash = packet_hash
    
    return packet
```

**Получение пакета:**

```python
def receive_packet(ctx, packet):
    # Validate chain
    if packet.seq <= ctx.rx.sequence:
        return ERROR_REPLAY_DETECTED
    
    if packet.seq > ctx.rx.sequence + MAX_SEQ_JUMP:
        return ERROR_SEQUENCE_TOO_FAR
    
    if packet.prev != ctx.rx.last_hash:
        return ERROR_CHAIN_BROKEN
    
    # Verify signature
    if not verify_hmac(packet):
        return ERROR_INVALID_SIGNATURE
    
    # Update chain
    packet_hash = SHA256(packet.to_json())
    ctx.rx.sequence = packet.seq
    ctx.rx.last_hash = packet_hash
    ctx.last_received_hash = packet_hash
    
    # Decrypt and process
    return decrypt_and_process(packet)
```

### 6.5 Связывание ответов

Ответы привязываются к запросу, на который отвечают:

```
Request (TX):  seq=5, prev=hash(pkt4)    ───────┐
                                                │
Response (RX): seq=7, prev=hash(rsp6)   ◄──────┘
               Ответ ЗНАЕТ, на какой запрос он отвечает,
               через ctx.last_received_hash
```

### 6.6 Восстановление цепочки

Если цепочки рассинхронизировались:

1. Любая из сторон может отправить команду `chain_reset`
2. Обе цепочки сбрасываются в начальное состояние
3. Повторная синхронизация начинается с seq=1

---

## 7. Управление сессиями

### 7.1 Установление сессии

```
Client                                    Device
   │                                         │
   │── session_init ────────────────────────►│
   │   { client_random[32] }                 │
   │                                         │
   │                     Generate session_id │
   │                     Generate device_random │
   │                     Derive session keys │
   │                     Create device_proof │
   │                                         │
   │◄─────────────────── session_challenge ──│
   │   { session_id[16],                     │
   │     device_random[32],                  │
   │     device_proof[32],                   │
   │     expires_in }                        │
   │                                         │
   │  Verify device_proof                    │
   │  Derive session keys                    │
   │  Create client_proof                    │
   │                                         │
   │── session_confirm ─────────────────────►│
   │   { session_id[16],                     │
   │     client_proof[32] }                  │
   │                                         │
   │                     Verify client_proof │
   │                     Activate session    │
   │                                         │
   │◄─────────────── session_established ────│
   │   { binding_token[32],                  │
   │     expires }                           │
   │                                         │
   ═══════════ SESSION ACTIVE ══════════════
```

### 7.2 Формирование ключей

**Сессионные ключи формируются с использованием HKDF:**

```python
# Combine randoms
combined = client_random + device_random + master_token

# Extract PRK
prk = HKDF_Extract(salt=session_id, ikm=combined)

# Expand to individual keys
session_key = HKDF_Expand(prk, "ewsp_session_key", 32)
enc_key     = HKDF_Expand(prk, "ewsp_session_enc", 32)
auth_key    = HKDF_Expand(prk, "ewsp_session_auth", 32)
binding_key = HKDF_Expand(prk, "ewsp_session_bind", 32)
```

### 7.3 Расчёт доказательства

**Доказательство устройства (подтверждает знание токена устройством):**

```python
device_proof = HMAC-SHA256(
    binding_key,
    "device" + session_id + client_random + device_random
)
```

**Доказательство клиента (подтверждает знание токена клиентом):**

```python
client_proof = HMAC-SHA256(
    binding_key,
    "client" + session_id + client_random + device_random
)
```

### 7.4 Шифрование сессии (AEAD)

После установления сессии сообщения используют AEAD:

```python
def session_encrypt(session, plaintext):
    nonce = session.send_counter.to_bytes(24, 'little')
    session.send_counter += 1
    
    aad = session.session_id + nonce[:8]
    ciphertext, tag = AEAD_Encrypt(session.enc_key, nonce, aad, plaintext)
    
    return nonce + ciphertext + tag
```

### 7.5 Раскрутка ключей

Для обеспечения прямой секретности ключи периодически раскручиваются:

```python
def ratchet_keys(session):
    new_key = HKDF(
        salt=session.ratchet_key,
        ikm=session.enc_key,
        info="ewsp_ratchet",
        length=32
    )
    
    session.ratchet_key = new_key
    session.enc_key = HKDF_Expand(new_key, "ewsp_enc", 32)
    session.auth_key = HKDF_Expand(new_key, "ewsp_auth", 32)
    session.ratchet_count = 0
```

Раскрутка происходит каждые `RATCHET_INTERVAL` (100) сообщений.

### 7.6 Защита от воспроизведения

64-битные счётчики + битовое окно предотвращают воспроизведение:

```python
def validate_counter(session, counter):
    if counter <= session.recv_counter - REPLAY_WINDOW:
        return False  # Too old
    
    if counter <= session.recv_counter:
        # Check bitmap
        bit_pos = session.recv_counter - counter
        if session.replay_bitmap & (1 << bit_pos):
            return False  # Already seen
        session.replay_bitmap |= (1 << bit_pos)
    else:
        # New high counter
        shift = counter - session.recv_counter
        session.replay_bitmap = (session.replay_bitmap << shift) | 1
        session.recv_counter = counter
    
    return True
```

---

## 8. Протокол команд

### 8.1 Список команд

| Команда           | Описание                              | Поля данных                    |
|-------------------|---------------------------------------|--------------------------------|
| `ping`            | Проверка соединения                   | —                              |
| `wake`            | Отправка Wake-on-LAN пакета           | `mac`, `port`                  |
| `info`            | Получение информации об устройстве    | —                              |
| `status`          | Получение статуса устройства          | —                              |
| `restart`         | Перезагрузка устройства               | —                              |
| `ota_start`       | Включение режима OTA-обновления       | —                              |
| `open_setup`      | Переход в режим настройки             | —                              |
| `update_token`    | Обновление токена устройства          | `encrypted_token`              |
| `set_cloud`       | Настройка параметров облака           | `host`, `port`, `enabled`      |
| `set_wifi`        | Настройка WiFi                        | `ssid`, `password`             |
| `factory_reset`   | Сброс к заводским настройкам          | —                              |
| `chain_sync`      | Синхронизация состояния цепочки       | `tx_seq`, `rx_seq`, `hashes`   |
| `chain_reset`     | Сброс цепочек в начальное состояние   | —                              |
| `session_init`    | Инициализация установления сессии     | `client_random`                |
| `session_confirm` | Подтверждение сессии                  | `session_id`, `client_proof`   |

### 8.2 Формат запроса команды

```json
{
  "cmd": "wake",
  "d": {
    "mac": "AA:BB:CC:DD:EE:FF",
    "port": 9
  },
  "rid": "X7K2M9P1"
}
```

### 8.3 Формат ответа

**Успех:**

```json
{
  "status": "ok",
  "rid": "X7K2M9P1",
  "d": {
    "sent": true,
    "target": "AA:BB:CC:DD:EE:FF"
  }
}
```

**Ошибка:**

```json
{
  "status": "error",
  "rid": "X7K2M9P1",
  "error": {
    "code": 1001,
    "message": "Invalid MAC address format"
  }
}
```

### 8.4 Идентификатор запроса (rid)

- 8 символов: A–Z, 0–9
- Генерируется клиентом
- Дублируется в ответе
- Используется для сопоставления запроса/ответа

---

## 9. Режимы транспорта

### 9.1 TCP Direct (локальный)

Для связи в локальной сети:

```
Client ◄──── TCP/JSON ────► Device
         Port: 8080 (default)
         Keepalive: 30s
```

**Процесс подключения:**
1. Клиент подключается к IP:порт устройства
2. Дополнительное рукопожатие не требуется (EWSP обеспечивает аутентификацию)
3. Двунаправленные JSON-сообщения (с разделителем новой строки)

### 9.2 WebSocket Secure (облачный ретранслятор)

Для удалённого доступа через облачный сервер:

```
Client ◄──── WSS ────► Server ◄──── WSS ────► Device
                     (Relay)
```

**Протокол WSS:**
1. Клиент подключается к `wss://server/wss`
2. Аутентифицируется с помощью API-токена
3. Подписывается на канал устройства
4. Сервер ретранслирует пакеты EWSP в обоих направлениях
5. Автоматическое переподключение с экспоненциальной задержкой

**Формат сообщения (WebSocket):**

```json
{
  "type": "command",
  "device_id": "WL35080814",
  "payload": { /* EWSP packet */ }
}
```

### 9.3 Безопасность транспорта

| Режим      | Безопасность транспорта | Безопасность EWSP        |
|------------|-------------------------|--------------------------|
| TCP Local  | Нет (доверенная ЛВС)    | Полное шифрование + auth |
| WSS Cloud  | TLS 1.3                 | Полное шифрование + auth |

**Примечание:** шифрование EWSP применяется независимо от транспорта. TLS — дополнительный уровень защиты.

---

## 10. Соображения безопасности

### 10.1 Требования к реализации

| Требование                              | Причина                                       |
|-----------------------------------------|-----------------------------------------------|
| Использовать сравнение за константное время | Предотвратить атаки по времени на HMAC/токены |
| Обнулять ключи после использования      | Предотвратить раскрытие памяти                |
| Валидировать все длины входных данных   | Предотвратить переполнение буфера             |
| Проверять границы последовательности    | Предотвратить целочисленное переполнение      |
| Ограничивать максимальный размер пакета | Предотвратить исчерпание памяти               |
| Использовать CSPRNG для всех случайных  | Предотвратить предсказание nonce/ключей       |

### 10.2 Рекомендуемые ограничения

| Параметр                   | Значение   | Обоснование                          |
|----------------------------|------------|--------------------------------------|
| Максимальный размер пакета | 64 КБ      | Предотвратить исчерпание памяти      |
| Максимальный прыжок по seq | 100        | Предотвратить DoS через перемотку    |
| Таймаут сессии             | 5 мин простоя | Ограничить окно перехвата сессии  |
| Максимальная жизнь сессии  | 24 часа    | Требовать периодической переаутентификации |
| Блокировка после ошибок auth | 5 попыток | Предотвратить перебор               |
| Ограничение частоты        | 10 запр/с  | Предотвратить DoS                    |

### 10.3 Обработка ошибок

**Никогда не раскрывайте конфиденциальную информацию в ошибках:**

```
✓ "Authentication failed"
✗ "Invalid token: expected 'abc123...'"
```

**Используйте унифицированные коды ошибок:**

| Диапазон  | Категория           |
|-----------|---------------------|
| 1xxx      | Ошибки протокола    |
| 2xxx      | Аутентификация      |
| 3xxx      | Ошибки команд       |
| 4xxx      | Ошибки цепочки      |
| 5xxx      | Ошибки крипто       |

### 10.4 Руководство по логированию

**НУЖНО логировать:**
- Попытки подключения (IP, идентификатор устройства)
- Ошибки аутентификации
- Нарушения цепочки
- Срабатывание ограничений частоты

**НЕ НУЖНО логировать:**
- Токены или ключи
- Расшифрованные нагрузки
- Полное содержимое пакетов

---

## Приложение A: Тестовые векторы

### A.1 SHA-256

```
Input:  "" (empty)
Output: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855

Input:  "hello"
Output: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

### A.2 HMAC-SHA256

```
Key:    "wakelink_test_key_32_characters!"
Input:  "test message"
Output: b1392a3a6768fdd5319792f45e7cc8922cc680db224e40da1898905d31a4d998
```

### A.3 HKDF-SHA256

```
IKM:    "input key material"
Salt:   "" (empty)
Info:   "wakelink_encryption_v1"
Length: 32
Output: [32 bytes of derived key]
```

### A.4 XChaCha20

```
Key:    [32 bytes of 0x00]
Nonce:  [24 bytes of 0x00]
Input:  [64 bytes of 0x00]
Output: [64 bytes of keystream]
```

### A.5 Пример пакета

**Входные данные:**

```
Device Token: "my_super_secret_device_token_32ch"
Device ID: "WL35080814"
Command: "wake"
Data: {"mac": "AA:BB:CC:DD:EE:FF"}
TX Sequence: 1
TX Last Hash: GENESIS_HASH
```

**Выходной пакет:**

```json
{
  "v": "1.0",
  "id": "WL35080814",
  "seq": 1,
  "prev": "0000000000000000000000000000000000000000000000000000000000000000",
  "p": "base64_of_nonce_and_encrypted_inner_json",
  "sig": "hmac_sha256_of_v_id_seq_prev_p"
}
```

---

## Приложение B: Референсные реализации

### B.1 C (ewsp-core)

```
Repository: ewsp-core/
Files:      src/ewsp_*.c, include/ewsp_*.h
Build:      CMake
Platforms:  Linux, Windows, macOS, ESP8266/ESP32
```

### B.2 Python

```
Repository: ewsp-core/bindings/python/
Package:    ewsp-core (pip installable)
API:        ewsp_core.CryptoManager, PacketManager
Fallback:   Pure Python (crypto_pure.py)
```

### B.3 Kotlin/JNI

```
Repository: ewsp-core/bindings/kotlin/
Package:    org.wakelink.ewsp
API:        Crypto.create(), Packet.create()
Fallback:   Pure Kotlin (CryptoPure.kt)
```

### B.4 Прошивка

```
Repository: wakelink-firmware/WakeLink/
Platform:   ESP8266, ESP32
Framework:  Arduino
Files:      packet.cpp, SessionManager.cpp
```

---

## История изменений

| Версия | Дата    | Изменения                                          |
|--------|---------|----------------------------------------------------|
| 1.0    | 2026-02 | Первый выпуск. Блокчейн-цепочка, AEAD              |

---

**Конец спецификации протокола EWSP v1.0**

*Данная спецификация распространяется по лицензии CC BY-SA 4.0. Реализации приветствуются.*
