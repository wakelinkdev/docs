[🇬🇧 English](quickstart.md) | [🇷🇺 Русский](quickstart_RU.md)

# Быстрый старт

Подключите первое устройство WakeLink за 5 минут.

## Предварительные требования

- Плата ESP8266 или ESP32
- USB-кабель для прошивки
- Учётные данные WiFi-сети
- MAC-адрес целевого компьютера ([как узнать](requirements.md#3-find-your-mac-address))

---

## Шаг 1: Прошивка устройства

Выберите удобный способ:

=== "Веб-прошивальщик (проще всего)"

    1. Подключите ESP-плату по USB
    2. Откройте [flash.wakelink.io](https://flash.wakelink.io) в Chrome/Edge
    3. Нажмите **Connect** и выберите COM-порт
    4. Нажмите **Install WakeLink**
    5. Дождитесь завершения (~60 секунд)

    !!! success "Готово"
        Прошивка установлена!

=== "Arduino IDE"

    1. Скачайте `wakelink-firmware` из [releases](https://github.com/wakelinkdev/wakelink-firmware/releases)
    2. Откройте `WakeLink.ino` в Arduino IDE
    3. Установите необходимые библиотеки:
        - `WebSocketsClient`
        - `ArduinoJson`
        - `Crypto` (для ESP8266) или встроенная для ESP32
    4. Выберите плату:
        - ESP8266: `NodeMCU 1.0` или `Generic ESP8266 Module`
        - ESP32: `ESP32 Dev Module`
    5. Нажмите **Upload**

=== "PlatformIO"

    ```bash
    # Клонируйте репозиторий
    git clone https://github.com/wakelinkdev/wakelink-firmware
    cd wakelink-firmware

    # Сборка и загрузка (ESP8266)
    pio run -e esp8266 -t upload

    # Или для ESP32
    pio run -e esp32 -t upload
    ```

---

## Шаг 2: Настройка устройства

После прошивки устройство запускается в **режиме первоначальной настройки (Provisioning Mode)**.

### Подключение к сети настройки

1. На телефоне или ноутбуке найдите WiFi-сеть: `WakeLink-Setup`
2. Подключитесь к ней (без пароля)
3. Откройте [http://192.168.4.1](http://192.168.4.1) в браузере

### Заполните конфигурацию

Заполните форму:

| Поле | Значение |
|------|---------|
| **Device Name** | Любое имя (например, `living-room`) |
| **WiFi SSID** | Название вашей домашней WiFi-сети |
| **WiFi Password** | Пароль от WiFi |
| **Server URL** | `wss://relay.wakelink.io` (или собственный сервер) |
| **Device Token** | Токен устройства (из шага 3) |
| **Target MAC** | MAC-адрес компьютера (например, `AA:BB:CC:DD:EE:FF`) |

Нажмите **Save**. Устройство перезагрузится и подключится к вашей WiFi-сети.

!!! tip "Световые индикаторы"
    - **Мигает** = Подключение к WiFi
    - **Горит постоянно** = Подключено к серверу
    - **Выключен** = Глубокий сон (штатная работа)

---

## Шаг 3: Регистрация устройства

Для аутентификации на сервере необходим токен устройства.

=== "CLI (Рекомендуется)"

    ```bash
    # Установите CLI
    pip install wakelink-cli

    # Зарегистрируйте устройство
    wakelink register --name living-room
    ```

    Скопируйте отображённый токен и вставьте его в шаге 2.

=== "Веб-панель"

    1. Перейдите на [app.wakelink.io](https://app.wakelink.io)
    2. Зарегистрируйтесь или войдите
    3. Нажмите **Add Device**
    4. Скопируйте сгенерированный токен

=== "Собственный сервер"

    ```bash
    # Внутри контейнера вашего сервера
    docker exec -it wakelink-api python -m wakelink.cli register --name living-room
    ```

---

## Шаг 4: Проверка команды пробуждения

### Через CLI

```bash
# Разбудить устройство
wakelink wake living-room

# С подробным выводом
wakelink wake living-room -v
```

### Через Android-приложение

1. Установите из [Google Play](https://play.google.com/store/apps/details?id=io.wakelink)
2. Войдите в аккаунт
3. Нажмите на устройство
4. Нажмите **Wake**

### Ожидаемый вывод

```
🚀 Sending wake command to 'living-room'...
✅ Wake command sent successfully
   Response time: 234ms
   Device acknowledged
```

---

## Шаг 5: Проверьте, что компьютер просыпается

1. Выключите целевой компьютер или переведите его в спящий режим
2. Подождите 10–15 секунд
3. Отправьте команду пробуждения
4. Компьютер должен включиться в течение 5 секунд

!!! warning "Устранение неполадок"
    Если компьютер не просыпается:

    1. **Проверьте настройки WoL**: [Руководство по требованиям](requirements.md#pc-wake-on-lan-setup)
    2. **Проверьте MAC-адрес**: Опечатки — частая причина
    3. **Проверьте подключение ESP**: Посмотрите на индикатор
    4. **Просмотрите логи**: `wakelink logs living-room`

---

## Краткий справочник

### Команды CLI

```bash
wakelink register --name <name>  # Зарегистрировать новое устройство
wakelink wake <name>             # Отправить команду пробуждения
wakelink status <name>           # Проверить статус устройства
wakelink logs <name>             # Просмотреть логи устройства
wakelink list                    # Список всех устройств
wakelink ping <name>             # Пингануть устройство
```

### Файлы конфигурации

| Расположение | Назначение |
|-------------|-----------|
| `~/.wakelink/config.yaml` | Конфигурация CLI |
| `~/.wakelink/devices.yaml` | Локальный кэш устройств |

### Эндпоинты сервера

| Эндпоинт | Описание |
|---------|---------|
| `wss://relay.wakelink.io` | Публичный relay-сервер |
| `https://wakelink-project.org/api` | REST API |
| `https://app.wakelink.io` | Веб-панель управления |

---

## Следующие шаги

<div class="grid cards" markdown>

-   :material-tune:{ .lg .middle } __Расширенная настройка__

    ---

    Локальный режим, OTA-обновления, несколько целей

    [:octicons-arrow-right-24: Конфигурация прошивки](../firmware/configuration.md)

-   :material-server:{ .lg .middle } __Собственный сервер__

    ---

    Запустите собственный relay-сервер

    [:octicons-arrow-right-24: Настройка сервера](../server/docker.md)

-   :material-shield:{ .lg .middle } __Лучшие практики безопасности__

    ---

    Ротация ключей, сетевая изоляция

    [:octicons-arrow-right-24: Руководство по безопасности](../security/best-practices.md)

</div>

---

## Нужна помощь?

- 📖 [FAQ](../reference/faq.md)
- 🔧 [Устранение неполадок](../firmware/troubleshooting.md)
- 💬 [Discord-сообщество](https://discord.gg/wakelink)
- 🐛 [Сообщить об ошибке](https://github.com/wakelinkdev/wakelink/issues/new)
