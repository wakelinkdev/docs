[🇬🇧 English](encryption.md) | [🇷🇺 Русский](encryption_RU.md)

# Детали шифрования

Технические подробности о криптографических примитивах, используемых в WakeLink.

## Сводка алгоритмов

| Назначение | Алгоритм | Размер ключа | Примечания |
|------------|----------|--------------|------------|
| Симметричное шифрование | XChaCha20-Poly1305 | 256 бит | Шифр AEAD |
| Аутентификация сообщений | HMAC-SHA256 | 256 бит | Целостность пакетов |
| Формирование ключей | HKDF-SHA256 | Переменный | Сессионные ключи |
| Хеширование | SHA-256 | 256 бит | Хеширование цепочки |
| Хеширование паролей | Argon2id | Переменный | Пароли пользователей |

---

## XChaCha20-Poly1305

### Почему XChaCha20-Poly1305?

| Особенность | Преимущество |
|-------------|--------------|
| **AEAD** | Шифрование и аутентификация в одном |
| **256-битный ключ** | Высокий запас прочности |
| **192-битный nonce** | Безопасная генерация случайного nonce |
| **Без дополнения** | Длина шифротекста = длине открытого текста |
| **Быстрый** | Оптимизирован для программной реализации |
| **Безопасный** | Устойчив к атакам по времени |

### Сравнение с AES-GCM

| Особенность | XChaCha20-Poly1305 | AES-256-GCM |
|-------------|---------------------|-------------|
| Размер nonce | 192 бит | 96 бит |
| Безопасность nonce | Случайный OK | Требует счётчик |
| Аппаратное обеспечение | Оптимизирован программно | Требует AES-NI |
| Скорость на ESP8266 | Быстрый | Медленный (нет AES-NI) |

### Параметры

```
Key:         256 bits (32 bytes)
Nonce:       192 bits (24 bytes)
Tag:         128 bits (16 bytes)
Max message: 2^64 bytes
```

### Использование

```c
// Encryption
uint8_t nonce[24];
getrandom(nonce, 24);

uint8_t ciphertext[plaintext_len + 16];
xchacha20_poly1305_encrypt(
    ciphertext,      // output
    plaintext,       // input
    plaintext_len,
    key,             // 32 bytes
    nonce,           // 24 bytes
    NULL,            // additional data
    0
);
```

---

## HMAC-SHA256

Используется для подписей пакетов (в дополнение к Poly1305):

```
signature = HMAC-SHA256(mac_key, canonical_message)
```

### Почему двойная аутентификация?

1. **Poly1305** аутентифицирует зашифрованную нагрузку
2. **HMAC** аутентифицирует всю структуру пакета

Это обеспечивает:
- Эшелонированную защиту
- Независимые уровни проверки
- Совместимость с серверами-ретрансляторами

### Реализация

```c
uint8_t signature[32];
hmac_sha256(
    mac_key,         // 32 bytes
    message,
    message_len,
    signature        // output: 32 bytes
);
```

---

## Формирование ключей (HKDF)

### Предварительно распределённый ключ

Генерируется при регистрации устройства:
```
PSK = random(32)  // 256 bits of entropy
```

### Формирование сессионного ключа

```
Session Key = HKDF-SHA256(
    IKM  = PSK,
    salt = SHA256(device_id || timestamp || server_nonce),
    info = "ewsp-v1-session",
    L    = 64
)

encryption_key = Session_Key[0:32]   // First 32 bytes
mac_key        = Session_Key[32:64]  // Last 32 bytes
```

### Почему HKDF?

- **Извлечение**: концентрирует энтропию из PSK
- **Расширение**: безопасно генерирует несколько ключей
- **Стандарт**: RFC 5869, широко проверен

---

## Хеш-цепочка (SHA-256)

Каждый пакет связан с предыдущим через хеш:

```
packet_hash = SHA256(canonical_packet_bytes)
```

### Структура цепочки

```
[Genesis] → [Packet 1] → [Packet 2] → [Packet 3]
  hash0   ←   prev    ←    prev    ←    prev
```

### Проверка

```c
uint8_t expected_prev[32];
sha256(previous_packet, previous_len, expected_prev);

if (memcmp(current_packet.prev, expected_prev, 32) != 0) {
    return ERROR_CHAIN_BROKEN;
}
```

---

## Хеширование паролей (Argon2id)

На стороне сервера, для паролей пользователей:

### Параметры

```
Argon2id:
  Memory:     64 MB
  Time:       3 iterations
  Parallelism: 4
  Salt:       16 bytes (random)
  Output:     32 bytes
```

### Почему Argon2id?

- **Устойчив к памяти**: сопротивляется атакам GPU/ASIC
- **Устойчив ко времени**: сопротивляется перебору
- **Защита от побочных каналов**: вариант Argon2id
- **Победитель**: конкурса Password Hashing Competition

---

## Генерация случайных чисел

### Требования

Все случайные числа должны быть криптографически стойкими:

| Платформа | Источник |
|-----------|----------|
| ESP8266 | `esp_random()` (аппаратный ГСЧ) |
| ESP32 | `esp_random()` (аппаратный ГСЧ) |
| Linux | `/dev/urandom` / `getrandom()` |
| Python | `secrets.token_bytes()` |

### Проверка

```c
// ESP8266/ESP32
uint32_t random_word = esp_random();

// Never use rand() or similar!
// ❌ rand()
// ❌ random()
// ❌ Math.random()
```

---

## Операции за константное время

Для предотвращения атак по времени:

### Сравнение

```c
// ✅ Constant time
int secure_compare(const uint8_t *a, const uint8_t *b, size_t len) {
    uint8_t diff = 0;
    for (size_t i = 0; i < len; i++) {
        diff |= a[i] ^ b[i];
    }
    return diff == 0;
}

// ❌ Variable time (vulnerable)
int insecure_compare(const uint8_t *a, const uint8_t *b, size_t len) {
    return memcmp(a, b, len) == 0;  // May return early
}
```

### Выбор

```c
// ✅ Constant time selection
uint8_t select(uint8_t a, uint8_t b, int choose_a) {
    uint8_t mask = -(choose_a != 0);  // All 1s or all 0s
    return (a & mask) | (b & ~mask);
}
```

---

## Хранение ключей

### На ESP-устройстве

```
Flash Memory (encrypted partition):
┌─────────────────────────────┐
│  Device Token (32 bytes)    │
│  PSK (32 bytes)             │ ← AES-256 encrypted
│  Session State (variable)   │
└─────────────────────────────┘
```

ESP32 поддерживает аппаратное шифрование флэш-памяти.

### На сервере

```
PostgreSQL (encrypted column):
┌─────────────────────────────┐
│  device_token_hash          │ ← Argon2 for verification
│  device_psk_encrypted       │ ← AES-256 with master key
└─────────────────────────────┘
```

### Иерархия ключей

```
Master Key (HSM/KMS)
    │
    ├── User Password Keys (Argon2id per user)
    │
    └── Device Encryption Keys (HKDF per device)
            │
            └── Session Keys (HKDF per session)
```

---

## Криптографические библиотеки

### ESP8266

- **BearSSL** (встроенная)
- Библиотека **Crypto** от Rhys Weatherley

### ESP32

- **mbedTLS** (встроенная)
- Аппаратное ускорение SHA, AES

### Python (сервер/CLI)

- Библиотека **cryptography** (PyCA)
- Использует бэкенд OpenSSL

### Справочник по реализации

```c
// ESP8266/ESP32
#include "CryptoManager.h"

ewsp_crypto_init();
ewsp_encrypt(key, nonce, plaintext, len, ciphertext);
ewsp_decrypt(key, nonce, ciphertext, len, plaintext);
ewsp_hmac(key, message, len, output);
ewsp_hkdf(ikm, salt, info, output, output_len);
```

---

## Аудит безопасности

Криптографическая реализация проверена на:

- [ ] Корректное использование алгоритмов
- [ ] Правильная работа с nonce
- [ ] Сравнения за константное время
- [ ] Безопасная генерация случайных чисел
- [ ] Очистка ключей после использования
- [ ] Предотвращение переполнения буфера

По вопросам безопасности см. [Политику безопасности](https://github.com/wakelinkdev/wakelink/security/policy).
