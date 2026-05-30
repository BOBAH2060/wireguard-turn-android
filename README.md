# WG Turn Wrap

Android WireGuard client fork with built-in TURN proxy transport and optional WRAP obfuscation.

The app is based on the official [WireGuard Android](https://git.zx2c4.com/wireguard-android) client and adds a native TURN/DTLS proxy path inside `libwg-go.so`. This makes it possible to run a normal WireGuard tunnel while the client sends packets through a compatible TURN proxy server.

## Highlights

- WireGuard Android UI with integrated TURN proxy settings.
- TURN transport implemented in the native `libwg-go.so` path.
- `proxy_v2` and `proxy_v1` DTLS modes.
- Multi-stream balancing for higher throughput and connection recovery.
- TURN settings stored inside normal WireGuard `.conf` files as `#@wgt:` metadata.
- Optional WRAP mode over DTLS/TURN using ChaCha20-Poly1305.
- Android app label: `WG Turn Wrap`.

## Download

Prebuilt APKs are published on the GitHub Releases page:

https://github.com/BOBAH2060/wireguard-turn-android/releases

Use the release APK for normal installation. Debug APKs are only for local testing.

## Configuration

Open a tunnel in the app, enable TURN Proxy in the advanced settings, and fill in the TURN fields. The same settings can also be stored directly in the WireGuard config:

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

WRAP adds an extra authenticated encryption layer around DTLS/TURN packets. Client and server must use the same 32-byte key encoded as 64 hexadecimal characters.

WRAP is intended for compatible DTLS proxy modes:

- `proxy_v2`
- `proxy_v1`

It is not used for `PeerType = wireguard`.

Example compatible server options:

```bash
./server \
  -listen 0.0.0.0:56000 \
  -connect 127.0.0.1:51820 \
  -wrap \
  -wrap-key <64-hex-characters>
```

If the key differs between client and server, the server will reject packets with an AEAD authentication error.

## Build

Requirements:

- JDK 17
- Android SDK
- Android NDK 29
- Go 1.25+
- Git submodules

```bash
git clone --recurse-submodules https://github.com/BOBAH2060/wireguard-turn-android.git
cd wireguard-turn-android
./gradlew :ui:assembleDebug
```

Debug APK:

```text
ui/build/outputs/apk/debug/ui-debug.apk
```

Release APK:

```bash
./gradlew :ui:assembleRelease
```

```text
ui/build/outputs/apk/release/ui-release.apk
```

## Release Notes

When creating a GitHub release, attach the release APK and mention:

- package name: `com.wgturn.android`
- app label: `WG Turn Wrap`
- supported ABIs: `arm64-v8a`, `armeabi-v7a`, `x86`, `x86_64`
- whether the APK is signed or a local test build

## Privacy

The TURN/WRAP server can see connection metadata such as client IP address, connection time, packet rate, and TURN credentials if the server logs them. It should not see the contents of traffic inside the WireGuard tunnel.

Do not publish real WireGuard private keys, preshared keys, TURN passwords, VK links, or WRAP keys in issues, logs, screenshots, or release notes.

## Credits

This project is based on:

- [WireGuard Android](https://git.zx2c4.com/wireguard-android)
- [vk-turn-proxy](https://github.com/cacggghp/vk-turn-proxy)
- [vk-turn-proxy WRAP fork](https://github.com/samosvalishe/vk-turn-proxy)
- [lionheart](https://github.com/jaykaiperson/lionheart)

## License

See [COPYING](COPYING). Individual components may be distributed under their own upstream licenses.
