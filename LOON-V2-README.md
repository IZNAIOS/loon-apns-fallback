# Loon v2

This is a public-safe Loon configuration for two private subscription resources
named `mojie` and `peiqian`.

## Policy design

- `AUTO`: URL latency test across every node in `mojie` and `peiqian`.
- `PROXY`: manual selection of `AUTO`, either subscription, or `DIRECT`.
- `APNS-FALLBACK`: always tries `PROXY` first and falls back to `DIRECT` only
  when the proxy candidate is unavailable.
- Ordinary proxy rules always target `PROXY`; they never target `AUTO` directly.

The three-group design is sound. Keeping `AUTO` behind `PROXY` provides one
stable policy name for rules and plugins while preserving manual override.

## Before importing

The reference configuration contained secret subscription tokens. They are not
included in the public file. In Loon, add the two Remote Proxy resources using
the exact aliases below:

```ini
mojie = <YOUR_MOJIE_SUBSCRIPTION_URL>
peiqian = <YOUR_PEQIAN_SUBSCRIPTION_URL>
```

Do this locally in Loon and never commit the completed lines to a public repo.

The included plugin entry maps the plugin's `PROXY` policy to
`APNS-FALLBACK`. If Loon asks for policy mapping during a separate manual plugin
import, select `APNS-FALLBACK` before enabling the plugin. Do not reverse the
group order: its required definition is `fallback,PROXY,DIRECT`.

## APNs coverage

`DOMAIN-SUFFIX,push.apple.com` covers all four supplied hosts:

- `push.apple.com`
- `gateway.push.apple.com`
- `api.push.apple.com`
- `sandbox.push.apple.com`

The companion plugin's APNs IPv4 and IPv6 networks are also mapped to
`APNS-FALLBACK` in the main configuration.

## Unknown destinations

Known mainland-China destinations use `DIRECT`; known overseas destinations
use `PROXY`; China IPs use `DIRECT`; and truly unmatched ordinary traffic ends
at `FINAL,PROXY`. If that proxy path is unsuitable, `DIRECT` remains available
as a manual choice inside the `PROXY` select group. APNs is matched earlier and
remains independently proxy-first through `APNS-FALLBACK`.

Loon cannot retry the current destination through another rule policy after a
proxy connection fails. A `fallback` policy checks candidates against one fixed
health-check URL, so using it for unknown websites would not reliably detect
whether each target is reachable. The configuration therefore gives unmatched
sites `PROXY` priority and retains `DIRECT` as a manual fallback without
pretending that automatic per-site retry exists.

## Module compatibility

The base file leaves Rewrite, Script, and MITM empty. Install ad blocking and
service-specific behavior as separate plugins, keep their host lists scoped,
and avoid adding a broad `FINAL` rule inside a plugin.
