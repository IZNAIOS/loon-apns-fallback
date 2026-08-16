# Loon v3

Loon v3 is the conservative APNs alternative to Loon v2. Use it if APNs proxy
routing causes delayed notifications, missing notifications, or poor app
compatibility. It keeps the same China/global split design but sends all Apple
traffic directly, approximating the APNs path used when no proxy is enabled.

It does not define or use `APNS-FALLBACK`, and it does not load the APNs
fallback plugin. This prioritizes predictable notification delivery for
mainland-China apps over forcing APNs through an airport node.

## Switching from Loon v2

Before enabling Loon v3, disable or remove any separately imported APNs Push
Fallback plugin in Loon. If that plugin remains enabled, its rules and saved
policy mapping may still capture APNs traffic even though this base
configuration sends Apple traffic to `DIRECT`.

## Policy design

- `AUTO`: latency test across every node in `mojie` and `peiqian`.
- `PROXY`: manual selection of `AUTO`, either subscription, or `DIRECT`.
- Known overseas services and unmatched traffic use `PROXY`.
- Known mainland-China services and China IPs use `DIRECT`.
- Apple domains, Apple-owned IPv4 space, and APNs IPv6 networks use `DIRECT`.

## Before importing

The public file deliberately excludes private subscription URLs and tokens.
Add these resources locally in Loon using the exact aliases:

```ini
mojie = <YOUR_MOJIE_SUBSCRIPTION_URL>
peiqian = <YOUR_PEQIAN_SUBSCRIPTION_URL>
```

Never commit the completed subscription lines to a public repository.

## Apple direct coverage

The main configuration explicitly sends the following to `DIRECT`:

- `push.apple.com`, covering push/gateway/api/sandbox APNs hosts
- Apple-owned IPv4 space `17.0.0.0/8`
- the APNs IPv6 networks from the supplied companion plugin
- common Apple, iCloud, iTunes, App Store, and Apple CDN domains
- the maintained Apple Loon rule set, evaluated before the Global rule set

There is no fallback group or APNs plugin mapping in Loon v3. Apple traffic
therefore cannot be redirected to `PROXY` by the base configuration.

## Other traffic

ChinaMax and explicit mainland-China rules use `DIRECT`. The Global rule set
and explicit overseas rules use `PROXY`. Truly unmatched traffic ends at
`FINAL,PROXY`, with `DIRECT` retained as a manual option inside `PROXY`.

## Module compatibility

Rewrite, Script, and MITM remain empty. Ad blocking and service-specific
behavior should stay in separate plugins. Avoid later plugins that add Apple or
APNs rules mapped to `PROXY` if you want to preserve Apple-direct behavior.
