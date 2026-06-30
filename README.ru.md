# Midea MFM-60ARN1-R → ESPHome → Home Assistant

[![ESPHome](https://img.shields.io/badge/ESPHome-Midea%20UART-03A9F4)](https://esphome.io/components/climate/midea/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Automations-41BDF5)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Локальная интеграция напольного/колонного кондиционера **Midea MFM-60ARN1-R** в **Home Assistant** через штатный разъём **CN700 Wi‑Fi module interface** на плате дисплея.

Проект использует:

- **ESP32 DevKit / ESP‑WROOM‑32**
- **BSS138 4-channel bidirectional level shifter 5V ↔ 3.3V**
- **ESPHome Midea UART climate**
- **DS18B20** как дополнительный внешний датчик температуры
- Автоматизации Home Assistant для ограничения минимальной температуры, управления шторками и расписания

> ⚠️ Работы выполняются внутри кондиционера. Внутри есть опасное сетевое напряжение. Перед разборкой отключайте питание кондиционера. Если нет опыта работы с электроникой и 220V — лучше привлечь специалиста.

---

## Состав проекта

```text
esphome/midea-air.yaml                         # ESPHome конфигурация ESP32
esphome/secrets.example.yaml                   # пример secrets.yaml без реальных секретов
home-assistant/automations/                    # примеры автоматизаций Home Assistant
docs/images/                                   # фото плат, компонентов и схема подключения
```

---

## Совместимость

Проверено на конфигурации:

| Компонент | Значение |
|---|---|
| Indoor unit | Midea MFM-60ARN1-R |
| Main board | 2023000945S |
| Display board | DISPLAY-DA400-JD1D.N.NXXS.XP1-1 v1.3 |
| Wi‑Fi connector | CN700 |
| UART speed | 9600 baud |
| ESP board | ESP32 DevKit / ESP‑WROOM‑32 |
| ESPHome framework | Arduino |
| Temperature sensor | DS18B20, GPIO23 |

ESPHome `midea` требует UART `9600 baud`, а Midea hardware interface обычно использует **5V logic levels**, поэтому между CN700 и ESP32 нужен level shifter. См. официальную документацию ESPHome:
https://esphome.io/components/climate/midea/

---

## Фото плат

### Основная плата

![Main board 2023000945S](docs/images/main-board-2023000945S.jpg)

### Плата дисплея

![Display board](docs/images/display-board-front.jpg)

### CN700 Wi‑Fi module interface

![CN700 front](docs/images/cn700-front.jpg)

![CN700 annotated](docs/images/cn700-front-annotated.png)

![CN700 backside annotated](docs/images/cn700-back-annotated.png)

---

## Что нужно купить

| Деталь | Количество | Комментарий |
|---|---:|---|
| ESP32 DevKit / ESP‑WROOM‑32 | 1 | Обычная ESP32-плата |
| BSS138 4-channel level shifter | 1 | Нужны только 2 канала |
| DS18B20 | 1 | Желательно влагозащищённый в металлической гильзе |
| Резистор 4.7 кОм | 1 | Между DATA DS18B20 и 3V3 |
| Провода Dupont/JST | по месту | Для подключения CN700 |
| Термоусадка/изоляция | по месту | Для безопасного монтажа |

### ESP32

![ESP32 DevKit](docs/images/esp32-devkit.jpg)

### BSS138 level shifter

![BSS138 level shifter](docs/images/bss138-level-shifter.jpg)

---

## Схема подключения

![Wiring diagram](docs/images/wiring-cn700-esp32-ds18b20.svg)

### CN700 → BSS138 → ESP32

> На некоторых платах CN700 может не иметь подписей пинов. Сначала найдите **GND** и **+5V** мультиметром. Два оставшихся пина — TX/RX. Если связи нет, поменяйте только TX/RX местами.

| CN700 / кондиционер | Level shifter | ESP32 |
|---|---|---|
| +5V | HV | VIN / 5V ESP32 |
| GND | GND | GND ESP32 |
| TX кондиционера | HV1 → LV1 | GPIO16 / RX ESP32 |
| RX кондиционера | HV2 ← LV2 | GPIO17 / TX ESP32 |
| — | LV | 3V3 ESP32 |

Важно:

- **CN700 +5V нельзя подключать к 3V3 ESP32.**
- **CN700 TX/RX нельзя подключать напрямую к GPIO ESP32.**
- Для прошивки ESP32 через USB лучше временно отключать питание `CN700 +5V → VIN/5V ESP32`, чтобы не было двух источников питания одновременно.
- `CN900 485` на дисплейной плате в этом проекте не используется.

---

## DS18B20

![DS18B20 to ESP32](docs/images/ds18b20-esp32-annotated.png)

Подключение DS18B20:

| DS18B20 | ESP32 |
|---|---|
| VCC / красный | 3V3 |
| DATA / жёлтый | GPIO23 / D23 |
| GND / чёрный | GND |

Поставьте резистор **4.7 кОм** между **DATA** и **3V3**.

ESPHome `dallas_temp` требует настроенный `one_wire` bus. См. официальные документы:
https://esphome.io/components/sensor/dallas_temp/
https://esphome.io/components/one_wire/

---

## ESPHome

Файл прошивки:

```text
esphome/midea-air.yaml
```

Перед публикацией или использованием создайте `secrets.yaml` на основе:

```text
esphome/secrets.example.yaml
```

Не публикуйте настоящий `secrets.yaml`.

`midea_air_fallback_password` в `secrets.example.yaml` — пример для ESPHome fallback hotspot. Перед реальным использованием замените его на свой пароль.

### Основные пины

| Назначение | ESP32 |
|---|---|
| Midea RX | GPIO16 |
| Midea TX | GPIO17 |
| DS18B20 DATA | GPIO23 |

### DS18B20 address

В примере указан адрес:

```yaml
address: 0xa000000035f59628
```

Если у вас другой датчик, удалите строку `address`, прошейте ESPHome и посмотрите найденный адрес в логах.

---

## Home Assistant entity IDs

В моём примере использовались:

```text
climate.midea_air_midea_air
button.midea_air_midea_air_power_toggle
```

У вас entity IDs могут отличаться. Проверьте их в:

```text
Настройки → Устройства и службы → ESPHome → Midea AIR → Объекты
```

---

## Примеры автоматизаций

Файлы находятся в:

```text
home-assistant/automations/
```

### 1. Минимальная температура 22°C

Если кто-то на панели кондиционера выставит 17–21°C, Home Assistant вернёт 22°C.

```text
home-assistant/automations/01-min-temperature-22.yaml
```

### 2. Управление шторкой после включения

После включения кондиционера:

1. Включает скорость вентилятора AUTO.
2. Включает вертикальную шторку на 12 секунд.
3. Останавливает вертикальную шторку.
4. Включает горизонтальную шторку.

```text
home-assistant/automations/02-louver-on-start.yaml
```

### 3. Выключение в 20:00

Отключает кондиционер в 20:00, если он включён и работает более 1 минуты.

```text
home-assistant/automations/03-turn-off-20-00.yaml
```

### 4. Включение в 08:15

Пример включения в 08:15, если температура выше 22°C и кондиционер был выключен более 3 минут.

```text
home-assistant/automations/04-turn-on-08-15-example.yaml
```

---

## Проверка после прошивки

1. Откройте ESPHome logs.
2. Убедитесь, что устройство подключилось к Wi‑Fi.
3. Убедитесь, что Home Assistant добавил ESPHome device.
4. Проверьте, что появился `climate` объект.
5. Проверьте DS18B20 temperature sensor.
6. Если `climate` не отвечает:
   - проверьте GND;
   - проверьте HV/LV level shifter;
   - поменяйте местами только TX/RX;
   - убедитесь, что UART `baud_rate` = `9600`.

---

## Типовые ошибки

### ESP32 есть в Wi‑Fi, но кондиционер не отвечает

Вероятные причины:

- перепутаны TX/RX;
- нет общего GND;
- TX/RX подключены без level shifter;
- HV/LV перепутаны;
- питание ESP32 подано не на VIN/5V;
- CN700 pinout определён неверно.

### DS18B20 не найден

Проверьте:

- резистор 4.7 кОм между DATA и 3V3;
- DATA подключён к GPIO23;
- датчик питается от 3V3;
- GND общий.

### `fan_mode: auto` не работает

В Home Assistant откройте:

```text
Разработчик → Состояния → climate.midea_air_midea_air
```

Проверьте атрибут `fan_modes`. Иногда значение может быть `Auto` вместо `auto`.

---

## Безопасность

Не публикуйте:

- Wi‑Fi SSID/password, если сеть приватная;
- API encryption key;
- OTA password;
- реальные `device_id` из Home Assistant;
- фото с серийными номерами, если не хотите их раскрывать.

---

## Disclaimer

This is a DIY integration. Use at your own risk. The author is not responsible for damage to the air conditioner, ESP32, electrical system, or property.

---

## License

MIT License. See [LICENSE](LICENSE).
