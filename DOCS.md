# KC868-A2 — 2 реле, 2 входа, Ethernet и Modbus

**Коротко:** `vuesp.kc868-a2` — прошивка Vuesp **5.0.0** для платы **KinCony KC868-A2 V2.3** на ESP32-WROOM-32E. Два реле и два оптоизолированных входа подключены напрямую к GPIO, есть два независимых терминала 1-Wire, Ethernet LAN8720A, RS485 Modbus RTU, I2C-разъём и сокет GSM. Web UI — **5.0.0**.

## Что это за устройство

KC868-A2 — компактный контроллер автоматизации на ESP32:

- 2 реле 10 A 250 V с клеммами NO/COM/NC;
- 2 оптоизолированных цифровых входа «сухой контакт»;
- 2 терминала для DS18B20/1-Wire или других датчиков;
- Ethernet LAN8720A 100 Мбит;
- RS485 MAX13487EESA с автонаправлением;
- сокет SIM800L 2G / SIM7600 4G;
- I2C-разъём;
- USB Type-C на CH340C;
- кнопки RESET и BOOT.

Прошивка умеет:

- включать/выключать 2 реле;
- читать 2 входа и события `port_c` / `port_l` / `port_r`;
- читать DS18B20 на двух шинах 1-Wire;
- работать по Ethernet (DHCP/static) и Wi-Fi;
- работать как Modbus RTU slave и master на RS485;
- отдавать веб-интерфейс, dashboard, скрипты, OTA и диагностику.

## Характеристики

| Параметр | Значение |
| --- | --- |
| Микроконтроллер | ESP32-WROOM-32E, 4 МБ flash, Wi-Fi/Bluetooth |
| Реле | 2, GPIO2 / GPIO15, активный уровень **high**, 10 A 250 V |
| Входы | 2, GPIO36 / GPIO39, активный уровень **low** |
| Датчики | 2 × 1-Wire: GPIO33 / GPIO14, подтяжка 4.7 кОм |
| Ethernet | LAN8720A, 100 Мбит |
| RS485 | MAX13487EESA, TX GPIO32, RX GPIO35 |
| GSM | сокеты SIM800L / SIM7600, UART GPIO34 / GPIO13 |
| I2C | SDA GPIO4, SCL GPIO16, подтяжка 3.3 кОм |
| USB | Type-C, CH340C |
| Питание | 9–22 В DC, на схеме вход V12 |
| Размер | 93 × 88 мм; корпус на DIN-рейку 116 × 90 × 41 мм |
| Среда | PlatformIO + Arduino |

## Распиновка

| Сигнал | GPIO | Примечание |
| --- | --- | --- |
| Relay 1 | **2** | high = включено |
| Relay 2 | **15** | high = включено |
| Input 1 | **36** | low = активно, input-only |
| Input 2 | **39** | low = активно, input-only |
| TEPP1 / 1-Wire 1 | **33** | подтяжка 4.7 кОм к 3.3 В |
| TEPP2 / 1-Wire 2 | **14** | подтяжка 4.7 кОм к 3.3 В |
| RS485 TX | **32** | MAX13487EESA |
| RS485 RX | **35** | через 74LVC1G125 |
| Ethernet MDC | **23** | LAN8720A |
| Ethernet MDIO | **18** | LAN8720A |
| Ethernet CLK | **17** | 50 МГц output |
| Ethernet PHY | **0** | PHYAD0 |
| GSM RX | **34** | от модуля к ESP32 |
| GSM TX | **13** | от ESP32 к модулю |
| I2C SDA | **4** | разъём P8 |
| I2C SCL | **16** | разъём P8 |
| USB UART | 1 / 3 | CH340C |
| BOOT | 0 | кнопка S2 |

> **Важно.**
> - Реле активны высоким уровнем: `output(pin, 1)` включает реле.
> - Входы активны низким уровнем: `input(pin)` возвращает `0`, когда вход активен.
> - GPIO36/39 — input-only, без внутренней подтяжки; на плате стоят внешние 4.7 кОм.
> - Для RS485 TX/RX должны быть перекрёстно: A одной платы → A другой, B → B, общий GND обязателен.

## Разъёмы

| Разъём | Контакты | Назначение |
| --- | --- | --- |
| P11 | NO1 / COM1 / NC1 | реле 1 |
| P12 | NO2 / COM2 / NC2 | реле 2 |
| P9 | INPUT_D1 / INPUT_D2 / GND | два входа |
| P6 | A (A1/RXI), B (B1/TXO), 3V3, TEPP2, TEPP1, GND, 12VIN | RS485, датчики, питание |
| P8 | 3V3 / GND / I2C_SDA / I2C_SCL | I2C |
| P1 | USB-C | питание/прошивка |
| P2/P3/P4/P5 | GSM | SIM800L / SIM7600 |

## Ethernet LAN8720A

| Параметр | Значение |
| --- | --- |
| Тип | LAN8720A |
| MDC | 23 |
| MDIO | 18 |
| CLK | 17, `ETH_CLOCK_GPIO17_OUT` |
| PHY address | 0 |
| POWER | не используется |

По умолчанию **DHCP**. Статический IP, маска, шлюз и DNS задаются в **Settings → Ethernet**. После сохранения устройство перезагружается.

## RS485 и Modbus RTU

RS485: **TX GPIO32**, **RX GPIO35**, MAX13487EESA с автонаправлением — отдельный DE/RE не нужен. По умолчанию включён **slave**, адрес `1`, `9600 8N1`. Настройки и карта регистров — на странице **`/device/modbus/`**.

Карта slave:

| Тип | Адрес | Значение |
| --- | --- | --- |
| Coil | 0–1 | реле 1–2, R/W |
| Discrete | 0–1 | входы 1–2, R; `0` = активен |
| Holding | 0 | маска входов |
| Holding | 1 | маска реле |
| Holding | 2–4 | версия прошивки |
| Holding | 5–6 | uptime |
| Holding | 7 | Modbus-адрес |
| Holding | 8 | скорость |
| Holding | 9 | роль |
| Holding | 10 | последняя ошибка |
| Holding | 11–12 | запросы / ошибки |
| Holding | 13–14 | Unix time |
| Holding | 15 | сырая маска входов |
| Holding | 16 | сырая маска реле |

## Датчики 1-Wire

TEPP1/GPIO33 и TEPP2/GPIO14 — две независимые шины. На каждой стоит подтяжка 4.7 кОм к 3.3 В. Прошивка ищет DS18B20 на обеих шинах и отправляет данные в web-стор `dallas` по адресу датчика. Поддерживаются несколько датчиков на одной шине.

## GSM

На плате есть сокеты **SIM800L 2G** и **SIM7600 4G**:

- UART: ESP32 RX — GPIO34, TX — GPIO13;
- питание модуля — VCC4V;
- `RST_GSM` выведен на разъёмы, к GPIO ESP32 не подключён.

Драйвер GSM/SMS/GPRS в этой версии **не реализован**: пины и сокеты описаны для будущей поддержки. Не подавайте питание на модуль без учёта тока — SIM800L требует импульсный БП.

## I2C

Разъём P8: 3.3 В, GND, SDA GPIO4, SCL GPIO16. На линии стоят подтяжки 3.3 кОм. Готового I2C-драйвера/страницы в Vuesp нет; разъём можно использовать в собственных модулях.

## Как устроен проект

- `src/device/device.cpp` — реле, входы, события, `input()`/`output()`;
- `src/gpio/gpio.cpp` — два порта 1-Wire, кнопка BOOT, поиск DS18B20;
- `src/modbus/` — Modbus RTU slave/master, профиль, raw-тест;
- `src/init/init.cpp` — Wi-Fi, Ethernet, статус в `Info`;
- `src/const/define.h` — пины платы;
- `src/const/protocol.h` — `Settings`, `Info`, `Device`, Modbus и служебные структуры;
- `vue/json/dashboard.json` — плитки входов, реле, Ethernet и Modbus;
- `vue/src/components/service/ServiceModbus.vue` — настройки Modbus и карта регистров;
- `data/www` — собранный веб-интерфейс.

## ScriptRunner

| Функция | Назначение |
| --- | --- |
| `input(pin)` | прочитать вход 1–2; `0` — активен |
| `output(pin, value)` | включить/выключить реле 1–2; `1` — включено |
| `var(id)` | прочитать/записать Modbus-переменную |
| `modbus(device, point)` | прочитать/записать точку Modbus |
| `fade(...)` | глобальная функция Vuesp; на A2 нет PWM-выходов |
| `udp(id, text)` | отправить текст через Discovery |

События входов: `port_c` (клик), `port_l` (долгое нажатие), `port_r` (повтор). Параметры: `$e0` — индекс входа 0–1, `$e1` — сырое значение, `$e2` — счётчик.

### Примеры скриптов

Стартовый набор лежит в `vue/public/scripts.txt` и после `build-web` попадает на устройство как `/www/scripts.txt`:

| ID | Имя | Что делает |
| ---: | --- | --- |
| 0 | `boot` | пишет приветствие в лог и регистрирует `input_event` |
| 1 | `relay1` | включает реле 1 на 1 секунду |
| 2 | `relay_walk` | поочерёдно включает оба реле по 500 мс |
| 3 | `input_scan` | выводит в лог состояние обоих входов |
| 4 | `input_event` | логирует `port_c`: индекс, сырое значение и счётчик |
| 5 | `relay_toggle` | переключает реле 1 коротким нажатием на IN1 |

`boot` не включает реле: скрипты 1, 2 и 5 запускаются вручную. Подсказки редактора: `input`, `output`, `var`, `modbus` и события `port_c/port_l/port_r` лежат в `vue/json/suggestions.json`; глобальные `fade()`, `udp()` и `http()` приходят из `vuesp-components`.

## Веб-интерфейс

- `/dashboard/` — 2 входа, 2 реле, Ethernet и Modbus;
- `/device/gpio/` — TEPP1/TEPP2 и кнопка BOOT;
- `/device/settings/` — Wi-Fi, безопасность, Ethernet, OTA;
- `/device/modbus/` — Modbus RTU и карта регистров;
- `/device/storage/` — файлы LittleFS;
- `/script/` — редактор ScriptRunner.

## Первичная настройка по USB

Если устройство ещё не в сети, его можно настроить через USB-serial (115200):

| Команда | Действие |
| --- | --- |
| `VUESP:PING` | проверить связь (`@VUESP:PONG`) |
| `VUESP:INFO` | IP Wi-Fi STA и AP (`0.0.0.0,192.168.4.1`) |
| `VUESP:SCAN` | список Wi-Fi сетей |
| `VUESP:SET<TAB>ssid<TAB>pass` | подключить Wi-Fi STA и перезагрузить |
| `VUESP:ETH` | link/скорость/текущий IP Ethernet |
| `VUESP:ETHSET<TAB>ip<TAB>gateway<TAB>mask` | задать статический Ethernet и перезагрузить |
| `VUESP:ETHDHCP` | вернуть Ethernet на DHCP |
| `VUESP:IO` | маски входов/выходов |
| `VUESP:OUT<TAB>n<TAB>0/1` | включить/выключить реле n (1–2) |

После `VUESP:ETHSET` или `VUESP:ETHDHCP` устройство перезагружается.

## Сборка и прошивка

```bash
cd vue
npm install
npm run protocol
npm run build-web

cd ..
~/.platformio/penv/bin/pio run -e esp32
~/.platformio/penv/bin/pio run -e esp32 -t buildfs
```

Прошивка и файловая система:

```bash
~/.platformio/penv/bin/pio run -e esp32 -t upload
~/.platformio/penv/bin/pio run -e esp32 -t uploadfs
```

> `pio run -t uploadfs` пересоздаёт LittleFS целиком: пропадут `/tmp/config.bin` (Wi-Fi, Ethernet, порты, Modbus) и загруженные файлы. Для обновления только веб-интерфейса загружайте файлы через `POST /fs`.

## Ограничения

- На A2 нет ADC, DAC, RS232, ИК, 433 МГц, RTC, SD и бузера.
- GSM-сокет пока только документация: драйвер SIM800L/SIM7600 не реализован.
- Ethernet и Wi-Fi работают одновременно; приоритет зависит от настроек и наличия link.
- I2C-разъём не имеет отдельной страницы в Vuesp.
- Старые настройки других плат не мигрируются: версия `Settings` новая.

## Источники

- [KC868-A2 schematic (PDF)](https://www.kincony.com/download/KC868-A2-schematic.pdf)
- [KinCony KC868-A2 product page](https://shop.kincony.com/products/kc868-a2-2-channel-esp32-4g-relay-board)
- [Habr: KC868-A2 — ESP32 + LAN8720A + GSM](https://habr.com/ru/companies/timeweb/articles/777334/)
- [ESPHome Devices: KinCony KC868-A2](https://devices.esphome.io/devices/kincony-kc868-a2)
