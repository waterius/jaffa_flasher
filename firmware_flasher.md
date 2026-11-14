# Прошивка программатора Jaffa Flasher (ESP32-S3 Super Mini)

## Первая прошивка

Проще всего — готовым файлом, без PlatformIO: [README, «1. Прошить ESP32-S3»](README.md#1-прошить-esp32-s3),
файл [`jaffa_flasher-esp32s3.bin`](https://github.com/waterius/jaffa_flasher/releases/latest/download/jaffa_flasher-esp32s3.bin)
с адреса `0x0`. После него чип в программаторе не выбран — выбрать на странице или `chip py32f002b`.

Из исходников — чистая плата или плата с чужой прошивкой:

1. Зажать BOOT, вставить USB (или нажать RESET, не отпуская BOOT) — плата в загрузчике ESP.
2. Залить:

   ```sh
   pio run -e esp32s3 -t upload
   ```

3. Плата перезагрузится и появится на другом порту — это уже порт программатора.

## Обновление

Та же команда, кнопки не нужны:

```sh
pio run -e esp32s3 -t upload
```

PlatformIO открывает порт программатора на скорости 1200 — он уходит в загрузчик ESP, — ждёт порт
загрузчика и заливает (`use_1200bps_touch` в `platformio.ini`). Монитор порта перед этим закрыть.
Если не вышло — кнопка BOOT, как при первой прошивке.

## Порты

| | Загрузчик ESP | Программатор (gdb, лог, команды, страница) |
|---|---|---|
| macOS | `/dev/cu.usbmodem1101` (номер зависит от гнезда USB) | `/dev/cu.usbmodem<серийный номер>`, например `/dev/cu.usbmodemD40592487DF01` |
| Windows | `COMn`, «USB JTAG/serial debug unit» в диспетчере устройств | `COMn`, «Устройство с последовательным интерфейсом USB» — другой номер, чем у загрузчика |

Список портов: `pio device list`. COM10 и дальше в Windows пишется как `\\.\COM10`.

## Выпуск релиза

```sh
pio run -e esp32s3            # .pio/build/esp32s3/firmware.factory.bin — образ с адреса 0x0
cp .pio/build/esp32s3/firmware.factory.bin jaffa_flasher-esp32s3.bin
gh release create vX.Y.Z -R waterius/jaffa_flasher --target <полный SHA master> jaffa_flasher-esp32s3.bin
```

Имя файла не меняется от версии к версии: ссылка в README `releases/latest/download/…` всегда ведёт
на последний релиз.
