# Telemt (fork)

Fork of [telemt/telemt](https://github.com/telemt/telemt), maintained as the
MTProto Proxy backend for the vpn-ui panel.

## Upstream version

Based on **telemt 3.4.23** (`Cargo.toml` `version = "3.4.23"`). Pinned so the
per-account behavior below stays stable across upstream releases.

## Changes in this fork

vpn-ui manages MTProto per ACCOUNT (one inbound serves many accounts) and
rewrites this config on every client change. Upstream telemt exposes only
process-wide or per-upstream knobs for the things vpn-ui needs per account, so
this fork adds three opt-in, hot-reloadable keys. Absent or empty, each behaves
exactly like stock telemt, so existing configs are unaffected.

- **`[access.user_modes]`: per-account connection modes.**
  A `username -> "classic,secure,tls"` map restricting which MTProto modes an
  account may use, ANDed with the process-wide `[general.modes]`. Enforced inside
  the per-secret candidate validator, so a disallowed mode is refused on the same
  path as a wrong secret and nothing downstream learns a new failure mode. Unknown
  tokens fail closed. Without it, widening `[general.modes]` to the union of every
  account's modes (needed so each account's handshake reaches auth) would let any
  account use any other account's mode, making the per-account toggles cosmetic.
  (`src/proxy/handshake.rs`, `src/config/types.rs`)

- **`upstreams.socks_user_from_account`: per-account SOCKS5 identity.**
  A per-upstream boolean. When set, the authenticated account name is presented as
  the SOCKS5 username (RFC 1929) to the upstream, so an upstream SOCKS5 server (the
  vpn-ui Xray socks inbound) can route per client instead of per inbound. Kept
  separate from `scopes`/`selected_scope` on purpose: it carries identity only and
  never affects upstream selection, so accounts are added by a hot reload rather
  than an `[[upstreams]]` change, which is restart-only and would drop every live
  connection. (`src/transport/upstream.rs`, `src/proxy/direct_relay.rs`)

- **`[general.modes]` made hot-reloadable.**
  Upstream copied only a fixed set of fields into the live config on reload, so
  `general.modes` was effectively restart-only. Because vpn-ui derives it from the
  union of every account's modes, enabling a mode for one account rewrote the
  config but the listener kept refusing until the next restart. The reloaded value
  now applies to the next handshake. (`src/config/hot_reload.rs`)

Both new keys are also registered in the strict-key allowlist
(`src/config/load/strict_keys.rs`), so the validator no longer logs them as
unknown while in fact honoring them.

---

# Telemt - MTProxy on Rust + Tokio

[![Latest Release](https://img.shields.io/github/v/release/telemt/telemt?color=neon)](https://github.com/telemt/telemt/releases/latest) [![Stars](https://img.shields.io/github/stars/telemt/telemt?style=social)](https://github.com/telemt/telemt/stargazers) [![Forks](https://img.shields.io/github/forks/telemt/telemt?style=social)](https://github.com/telemt/telemt/network/members)

[🇷🇺 README на русском](https://github.com/telemt/telemt/blob/main/README.ru.md)

> [!NOTE]
>
> From June 5th, 2026: we are already analyzing the causes of a new wave of "malfunctions"
> 
> Telegram Clients TLS ClientHello has been banned by JA4/JA4+ Fingerprint: we are already looking for ways to solve this problem
>
> You can try build your client with our Telegram Devlibrary - [tdlib-obf](https://github.com/telemt/tdlib-obf)

<p align="center">
  <a href="https://t.me/telemtrs">
    <img src="https://github.com/user-attachments/assets/30b7e7b9-974a-4e3d-aab6-b58a85de4507" width="240"/>
  </a>
</p>

**Telemt** is a fast, secure, and feature-rich server written in Rust: it fully implements the official Telegram proxy algo and adds many production-ready improvements

### One-command Install and Update
```bash
curl -fsSL https://raw.githubusercontent.com/telemt/telemt/main/install.sh | sh
```
- [Quick Start Guide](docs/Quick_start/QUICK_START_GUIDE.en.md)
- [Инструкция по быстрому запуску](docs/Quick_start/QUICK_START_GUIDE.ru.md)

## Features
Our implementation of **TLS-fronting** is one of the most deeply debugged, focused, advanced and *almost* **"behaviorally consistent to real"**:  we are confident we have it right - [see evidence on our validation and traces](docs/FAQ.en.md#recognizability-for-dpi-and-crawler)

Our ***Middle-End Pool*** is fastest by design in standard scenarios, compared to other implementations of connecting to the Middle-End Proxy: non dramatically, but usual

- Full support for all official MTProto proxy modes:
  - Classic;
  - Secure - with `dd` prefix;
  - Fake TLS - with `ee` prefix + SNI fronting;
- Replay attack protection;
- Optional traffic masking: forward unrecognized connections to a real web server, e.g. GitHub 🤪;
- Configurable keepalives + timeouts + IPv6 and "Fast Mode";
- Graceful shutdown on Ctrl+C;
- Extensive logging via `trace` and `debug` with `RUST_LOG` method.

## FAQ
- [FAQ RU](docs/FAQ.ru.md)
- [FAQ EN](docs/FAQ.en.md)

# Learn more about Telemt
- [Our Architecture](docs/Architecture)
- [All Config Options](docs/Config_params)
- [How to build your own Telemt?](#build)
- [Running on BSD](docs/Quick_start/OPENBSD_QUICK_START_GUIDE.en.md)
- [Why Rust?](#why-rust)

## Build
```bash
# Cloning repo
git clone https://github.com/telemt/telemt 
# Changing Directory to telemt
cd telemt
# Starting Release Build
cargo build --release

# Current release profile uses lto = "fat" for maximum optimization (see Cargo.toml).
# On low-RAM systems (~1 GB) you can override it to "thin".

# Move to /bin
mv ./target/release/telemt /bin
# Make executable
chmod +x /bin/telemt
# Lets go!
telemt config.toml
```

## Why Rust?
- Long-running reliability and idempotent behavior
- Rust's deterministic resource management - RAII 
- No garbage collector
- Memory safety and reduced attack surface
- Tokio's asynchronous architecture

## Support Telemt

Telemt is free, open-source, and built in personal time.
If it helps you — consider supporting continued development.

Any cryptocurrency (BTC, ETH, USDT, 350+ coins):

<p align="center">
  <a href="https://nowpayments.io/donation?api_key=2bf1afd2-abc2-49f9-a012-f1e715b37223" target="_blank" rel="noreferrer noopener">
    <img src="https://nowpayments.io/images/embeds/donation-button-white.svg" alt="Cryptocurrency & Bitcoin donation button by NOWPayments" height="80">
  </a>
</p>

Monero (XMR) directly:

```
8Bk4tZEYPQWSypeD2hrUXG2rKbAKF16GqEN942ZdAP5cFdSqW6h4DwkP5cJMAdszzuPeHeHZPTyjWWFwzeFdjuci3ktfMoB
```

All donations go toward infrastructure, development and research


![telemt_scheme](docs/assets/telemt.png)
