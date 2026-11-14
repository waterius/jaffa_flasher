# Своя страница для изделия

Для тех, кто делает свою заводскую страницу поверх модулей страницы прошивки. Шить плату это не
нужно — для этого есть <https://waterius.github.io/jaffa_flasher/>.

Страница изделия (заводская: свои данные для USER OTP, свой учёт) берёт копию `docs/lib/` с метки
версии этого репозитория и строит свой экран поверх `flasher.js`. Модули — обычные ES-модули без
зависимостей и сборки.

```js
import { Flasher, hexDump } from './lib/flasher.js';
import { choosePort, WebSerialPort } from './lib/webserial.js';

const port = new WebSerialPort(await choosePort());   // по нажатию кнопки: Web Serial просит жест
const flasher = new Flasher(port, {
  onLine: (line, own) => console.log(line),             // own — строка программатора «# »
  onState: (s) => console.log(s.connected, s.chip),     // подключение, чип, идёт ли сеанс
});
await flasher.connect();                                // открыть порт; страница спросит chip
await flasher.command('chip py32f002b');                // любая команда строкой

const r = await flasher.flash(
  [{ name: 'fw.hex', text: fwText }, { name: 'otp.hex', text: otpText }],
  { eraseAll: true, onProgress: (p) => console.log(p.phase, p.sent, p.total) },
);
// r.ok, r.reason — причина отказа, r.lines — строки программатора и лога после сеанса

const m = await flasher.read(0x1fff0280, 128);          // m.ok, m.bytes
console.log(hexDump(0x1fff0280, m.bytes).join('\n'));
await flasher.disconnect();
```

- `Flasher` получает порт протоколом «порт программатора»: `open()`, `close()`, `write(bytes)`,
  `onData(cb)`, `onClose(cb)` — описание в начале `flasher.js`. Другой способ достать порт
  (мост настольного приложения) подставляется вместо `WebSerialPort`.
- Один сеанс за раз: пока идёт `flash` или `read`, `command` отвечает ошибкой «идёт сеанс».
- Программатор не ответил в срок — `flash` и `read` бросают «программатор не отвечает», порт закрывается.
- Строки, которые разбирает `flasher.js`: `# чип: <имя>`, `# чип не выбран`,
  `# поддержаны: …`, `# итог: ок…`, `# итог: ошибка, <причина>`.
