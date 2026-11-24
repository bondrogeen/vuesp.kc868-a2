# vuesp.kc868-a2

KinCony **KC868-A2 V2.3**: 2 реле, 2 опто-входа, 2 терминала 1-Wire, Ethernet LAN8720, RS485/Modbus, сокет GSM и локальный веб-интерфейс Vuesp.

Проект собран на базе [Vuesp](https://github.com/bondrogeen/vuesp) 5.0.x: прошивка **5.0.0**, web UI **5.0.0** на Vue 3 и `vuesp-components@5.2.2`.

## Возможности

- 2 реле 10 A 250 V (NO/COM/NC) на GPIO2 / GPIO15, активный уровень **high**;
- 2 оптоизолированных входа на GPIO36 / GPIO39, активный уровень **low**;
- 2 независимых терминала 1-Wire/GPIO: GPIO33 и GPIO14, подтяжка 4.7 кОм;
- Ethernet LAN8720A: DHCP или статический IP, настройка в Settings;
- RS485 MAX13487EESA с автонаправлением и Modbus RTU master/slave;
- сокет SIM800L 2G / SIM7600 4G: пины и разъёмы описаны, драйвер пока не реализован;
- I2C-разъём: SDA GPIO4, SCL GPIO16;
- USB Type-C, CH340C, кнопки RESET/BOOT;
- dashboard: 2 входа, 2 реле, Ethernet и Modbus;
- ScriptRunner: `input()`, `output()`, `modbus()` и готовые примеры в Scripts;
- ConfigStore, диагностика загрузки/loop, OTA, файловая система LittleFS.

## Документация

Подробное описание платы, распиновка, Ethernet, RS485, датчики, ScriptRunner и ограничения — в [DOCS.md](./DOCS.md).

## Плата и окружение

- PlatformIO env: **esp32**
- Прошивка: `platformio.ini`
- Web UI: `vue/`
- Файловая система: LittleFS, веб-сборка попадает в `data/www`
- Настройки устройства и портов: `ConfigStore` (`/tmp/config.bin`)
- Настройки Wi-Fi/Ethernet: EEPROM (`Settings`)

## Сборка

`vuesp-components` подключён как локальная file-зависимость `file:../../../vuesp-components`. Для чистой сборки нужен соседний репозиторий `vuesp-components` с выполненным `npm run build`. Для CI/релиза замените зависимость на опубликованную `5.2.2`.

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

`uploadfs` пересоздаёт LittleFS и стирает `/tmp/config.bin`. Для обновления только веб-интерфейса используйте `POST /fs`.

## Ethernet

По умолчанию Ethernet включён и использует **DHCP**. Статический режим и адреса задаются на странице **Settings → Ethernet**. После сохранения устройство перезагружается.

Аппаратная часть: LAN8720A, MDC 23, MDIO 18, CLK 17, PHY address 0, POWER не используется.

## RS485 и Modbus

RS485: **TX GPIO32**, **RX GPIO35**, MAX13487EESA с автонаправлением. По умолчанию включён **slave**, адрес `1`, `9600 8N1`. Настройки и карта регистров — на странице **`/device/modbus/`**.

## Датчики 1-Wire

TEPP1/GPIO33 и TEPP2/GPIO14 — два независимых терминала с подтяжкой 4.7 кОм к 3.3 В. Поддерживаются DS18B20 и другие датчики 1-Wire; температуры приходят в web-стор `dallas` по адресу датчика.

## Ограничения

- На A2 нет ADC, DAC, RS232, ИК, 433 МГц, RTC, SD и бузера.
- GSM-сокет пока только документация: драйвер SIM800L/SIM7600 не реализован.
- Прошивка 5.0.0 — новая конфигурация; старые настройки не мигрируются.
- Питание платы 9–22 В DC; логика 3.3 В.

## Репозиторий

<https://github.com/bondrogeen/vuesp.kc868-a2>
