# SwiftNG-core — SwiftNG core bindings

gomobile bindings that compile the [SwiftNG core](https://github.com/cien121/xray-core) (Xray-core with AnyTLS patch) into `swiftng-core.aar` for [SwiftNG](https://github.com/cien121/SwiftNG).

Based on [2dust/AndroidLibXrayLite](https://github.com/2dust/AndroidLibXrayLite), via [ArasTey/AndroidLibXrayLite](https://github.com/ArasTey/AndroidLibXrayLite).

## What's added

| Binding | Description |
|---|---|
| `AwgTurnOn(fd, uapiConfig, mtu)` | Runs a **standalone AmneziaWG tunnel** on the VPN service's TUN fd — the same architecture as the AmneziaVPN Android client. The fd is wrapped with a minimal read/write TUN device (no `/dev/net/tun` ioctls — those are EACCES for unprivileged apps) and the fd is duplicated so Go and Kotlin own independent references |
| `AwgTurnOff()` / `AwgIsRunning()` | Tunnel lifecycle |
| `measureOutboundDelay` | Real-ping measurement through the core |
| `RegisterProcessFinder` | Per-app routing via UID lookup |

`go.mod` wires the Xray dependency to the patched fork:
```
replace github.com/xtls/xray-core       => github.com/ArasTey/xray-core main
replace github.com/patterniha/xray-core => github.com/ArasTey/xray-core main
```

## Build

Requirements: Go 1.25+, JDK 17, Android SDK + NDK r27, gomobile.

```bash
go install golang.org/x/mobile/cmd/gomobile@latest
go install golang.org/x/mobile/cmd/gobind@latest
gomobile init

# geo routing assets
curl -L https://github.com/Chocolate4U/Iran-v2ray-rules/releases/latest/download/geoip.dat -o assets/geoip.dat
curl -L https://github.com/Chocolate4U/Iran-v2ray-rules/releases/latest/download/geosite.dat -o assets/geosite.dat
# geoip-only-cn-private.dat ships in this repo's assets/

gomobile bind -androidapi 24 -trimpath \
  -ldflags='-s -w -buildid= -checklinkname=0' -o aras-core.aar ./
```

The output `swiftng-core.aar` is consumed by [SwiftNG](https://github.com/cien121/SwiftNG) (`V2rayNG/app/libs/`). Its GitHub Actions workflow builds this AAR from this public source, then builds the app on top of it — so every release APK is reproducible from open source.
