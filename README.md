# WireGuard Android с VK TURN Proxy

Форк официального клиента [WireGuard Android](https://git.zx2c4.com/wireguard-android) с поддержкой TURN-прокси для транспорта WireGuard через DTLS/TURN.

Главная фишка этого форка - **WRAP-обфускация**: дополнительная упаковка DTLS/TURN-трафика через ChaCha20-Poly1305, совместимая с серверной частью `vk-turn-proxy` при одинаковом WRAP key на клиенте и сервере.

Проект предназначен для исследований, тестирования и частных стендов. Используйте только те TURN-серверы и сервисы, на которые у вас есть разрешение.

## Что умеет

- запускать WireGuard через встроенный TURN-клиент в `libwg-go.so`;
- получать TURN credentials через режимы `vk_link` и `wb`;
- работать с несколькими DTLS-потоками и балансировкой трафика;
- автоматически переподключаться при смене сети;
- хранить TURN-настройки прямо в `.conf` файлах WireGuard через комментарии `#@wgt:`;
- использовать WRAP-обфускацию поверх DTLS/TURN - это основное отличие форка.

## Сборка

Нужны:

- JDK 17;
- Android SDK;
- Android NDK 29;
- Go 1.25+;
- Git submodules.

```bash
git clone --recurse-submodules https://github.com/<owner>/<repo>.git
cd <repo>
./gradlew :ui:assembleDebug
```

Готовый debug APK появится в:

```text
ui/build/outputs/apk/debug/ui-debug.apk
```

Для release-сборки используйте:

```bash
./gradlew :ui:assembleRelease
```

## Настройка в приложении

Откройте туннель WireGuard, включите TURN Proxy в расширенных настройках и заполните параметры:

- `Mode`: `vk_link` или `wb`;
- `Peer type`: обычно `proxy_v2`;
- `Streams`: количество параллельных потоков;
- `Local port`: локальный UDP-порт прокси;
- `VK Link` или параметры WB;
- при необходимости включите `WRAP obfuscation` и укажите WRAP key.

Настройки сохраняются в конфиге WireGuard как метаданные:

```ini
[Peer]
PublicKey = <peer-public-key>
Endpoint = <server-ip>:51820
AllowedIPs = 0.0.0.0/0

#@wgt:EnableTURN = true
#@wgt:UseUDP = false
#@wgt:IPPort = <proxy-server-ip>:56000
#@wgt:VKLink = https://vk.com/call/join/...
#@wgt:Mode = vk_link
#@wgt:PeerType = proxy_v2
#@wgt:StreamNum = 4
#@wgt:LocalPort = 9000
#@wgt:StreamsPerCred = 4
#@wgt:WatchdogTimeout = 30
#@wgt:Wrap = true
#@wgt:WrapKey = <64-hex-characters>
```

## WRAP

WRAP добавляет дополнительную обфускацию DTLS-пакетов через ChaCha20-Poly1305. Клиент и сервер должны использовать один и тот же ключ длиной 32 байта, записанный как 64 hex-символа.

Пример запуска совместимого сервера:

```bash
./server \
  -listen 0.0.0.0:56000 \
  -connect 127.0.0.1:51820 \
  -wrap \
  -wrap-key <64-hex-characters>
```

WRAP работает только с DTLS-режимами `proxy_v1` и `proxy_v2`. Для `PeerType = wireguard` он отключен.

## Совместимость

Рекомендуемый сервер для режима `proxy_v2`: форки `vk-turn-proxy` с поддержкой Session ID и WRAP. Для старых серверов можно использовать `proxy_v1`, если они не поддерживают v2 handshake.

## Проверка

Перед публикацией или релизом желательно выполнить:

```bash
./gradlew :ui:assembleDebug
./gradlew :tunnel:testDebugUnitTest
```

Затем установите APK на устройство или эмулятор и проверьте запуск туннеля с реальным TURN-сервером.

## Благодарности

Проект основан на:

- [WireGuard Android](https://git.zx2c4.com/wireguard-android);
- [vk-turn-proxy](https://github.com/cacggghp/vk-turn-proxy);
- [vk-turn-proxy WRAP fork](https://github.com/samosvalishe/vk-turn-proxy);
- [lionheart](https://github.com/jaykaiperson/lionheart).

## Лицензия

Смотрите [COPYING](COPYING). Отдельные компоненты могут распространяться на условиях своих исходных лицензий.
