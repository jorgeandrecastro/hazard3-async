# hazard3-async

> **Independent RISC-V async embedded Rust ecosystem.**
> Maintained by [Jorge Andre Castro](https://github.com/jorgeandrecastro) at <https://github.com/jorgeandrecastro/Hazard3>.
>
> `hazard3-async` is a pure RISC-V ecosystem built for the **RP2350 Hazard3 cores**. It removes all Arm Cortex-M and RP2040 overhead to stay lean, fast, and focused on modern RISC-V hardware—featuring **native bidirectional ESP-NOW over CYW43**, custom RISC-V linking, and standalone peripheral drivers.

---

## Table of Contents

- [Vision & Architecture](#vision--architecture)
- [Flagship Feature: Native ESP-NOW over CYW43](#flagship-feature-native-esp-now-over-cyw43)
- [Ecosystem Crates](#ecosystem-crates)
  - [Core Runtime, HAL & Networking](#core-runtime-hal--networking)
  - [Standalone Drivers, Sensors & Protocols](#standalone-drivers-sensors--protocols)
  - [Tooling & Linker](#tooling--linker)
- [Quick Start](#quick-start)
- [Migration from Legacy `*-riscv` Packages](#migration-from-legacy--riscv-packages)
- [Hardware Validation & Benchmarks](#hardware-validation--benchmarks)
- [Contributing](#contributing)
- [License & Acknowledgements](#license--acknowledgements)

---

## Vision & Architecture

`hazard3-async` is a **RISC-V-first** embedded async stack designed to give developers total agility and zero legacy bloat on Hazard3 silicon:

- **100% RISC-V Only:** All Arm Cortex-M and RP2040 code, interrupt vector tables, and conditional `cfg` flags are stripped out. For Arm or RP2040 targets, use upstream [Embassy](https://github.com/embassy-rs/embassy).
- **Source-Compatible Library Names:** Core crates preserve standard Rust library names (`embassy_rp`, `embassy_time`, `embassy_executor`, `embassy_net`, `cyw43`, `rp_pac`) so existing async Rust code works seamlessly by aliasing `package = "hazard3-*"` in `Cargo.toml`.
- **Modular & Fast-Moving:** Independent standalone crates for sensors, protocols, and wireless links can be updated and shipped rapidly without upstream bottlenecks.

> **Important:** Always use the core `hazard3-*` crates **together** to avoid duplicate or incompatible copies of underlying runtime dependencies.

---

## Flagship Feature: Native ESP-NOW over CYW43

[`hazard3-cyw43`](https://crates.io/crates/hazard3-cyw43) and [`hazard3-cyw43-pio`](https://crates.io/crates/hazard3-cyw43-pio) deliver bidirectional **ESP-NOW** communication directly on the **Raspberry Pi Pico 2 W** (RP2350 Hazard3 + CYW43439 radio over RISC-V PIO transport)—eliminating heavy Wi-Fi association overhead for low-latency peer-to-peer IoT links.

- **Hardware-validated** with an ESP32-C3 Mini using PING/PONG action frames, sequence-number validation, timeout detection, and round-trip-time (RTT) tracking.
- **1,000-exchange benchmark:** 100% delivery rate with **0 TX errors**, **0 invalid/unexpected PONGs**, **0 timeouts**, and an average RTT of **13.732 ms**.

---

## Ecosystem Crates

### Core Runtime, HAL & Networking

| Crate | Rust Library Name | Role |
| :--- | :--- | :--- |
| [`hazard3-rp`](https://crates.io/crates/hazard3-rp) | `embassy_rp` | RP2350 async HAL (RISC-V Hazard3 only) |
| [`hazard3-rp-pac`](https://crates.io/crates/hazard3-rp-pac) | `rp_pac` | RP2350 Peripheral Access Crate |
| [`hazard3-cyw43`](https://crates.io/crates/hazard3-cyw43) | `cyw43` | CYW43 Wi-Fi, BLE & native ESP-NOW driver |
| [`hazard3-cyw43-pio`](https://crates.io/crates/hazard3-cyw43-pio) | `cyw43_pio` | RP2350 PIO transport for CYW43 |
| [`hazard3-net-async`](https://crates.io/crates/hazard3-net-async) | `embassy_net` | Async TCP/UDP/DNS networking stack |
| [`hazard3-net`](https://crates.io/crates/hazard3-net) | `xarxa` | Xarxa-derived core network stack |
| [`hazard3-net-driver`](https://crates.io/crates/hazard3-net-driver) | `xarxa_driver` | Network driver interface |
| [`hazard3-net-driver-channel`](https://crates.io/crates/hazard3-net-driver-channel) | `embassy_net_driver_channel` | Channel-based network drivers |
| [`hazard3-executor`](https://crates.io/crates/hazard3-executor) | `embassy_executor` | Async task executor for RISC-V |
| [`hazard3-executor-macros`](https://crates.io/crates/hazard3-executor-macros) | `embassy_executor_macros` | Procedural macros for `hazard3-executor` |
| [`hazard3-executor-timer-queue`](https://crates.io/crates/hazard3-executor-timer-queue) | `embassy_executor_timer_queue` | Timer queue backend for executor |
| [`hazard3-time`](https://crates.io/crates/hazard3-time) | `embassy_time` | Async timers, delays, and duration types |
| [`hazard3-time-driver`](https://crates.io/crates/hazard3-time-driver) | `embassy_time_driver` | Hardware time driver traits |
| [`hazard3-time-queue-utils`](https://crates.io/crates/hazard3-time-queue-utils) | `embassy_time_queue_utils` | Timer queue utilities |
| [`hazard3-sync`](https://crates.io/crates/hazard3-sync) | `embassy_sync` | Async synchronization primitives (channels, mutexes, signals) |
| [`hazard3-embedded-hal`](https://crates.io/crates/hazard3-embedded-hal) | `embassy_embedded_hal` | `embedded-hal` and `embedded-hal-async` adapters |
| [`hazard3-usb-driver`](https://crates.io/crates/hazard3-usb-driver) | `embassy_usb_driver` | USB device driver traits |
| [`hazard3-hal-internal`](https://crates.io/crates/hazard3-hal-internal) | `embassy_hal_internal` | Shared HAL internal utilities |
| [`hazard3-unitrait`](https://crates.io/crates/hazard3-unitrait) | `unitrait` | Global implementation traits |
| [`hazard3-unitrait-macros`](https://crates.io/crates/hazard3-unitrait-macros) | `unitrait_macros` | Procedural macros for `hazard3-unitrait` |

### Standalone Drivers, Sensors & Protocols

| Crate | Role |
| :--- | :--- |
| [`Hazard3-bmp280`](https://crates.io/crates/Hazard3-bmp280) | Async `no_std` driver for the Bosch BMP280 temperature & pressure sensor |
| [`hazard3-am2302`](https://crates.io/crates/hazard3-am2302) | Async `no_std` driver for the AM2302 / DHT22 temperature & humidity sensor |
| [`hazard3-mqtt-lite`](https://crates.io/crates/hazard3-mqtt-lite) | Lightweight `no_std` async MQTT client built on `embedded-io-async` |

### Tooling & Linker

| Crate | Role |
| :--- | :--- |
| [`rp2350-riscv-linker`](https://crates.io/crates/rp2350-riscv-linker) | Custom linker scripts, `IMAGE_DEF` blocks, and memory layout for RP2350 RISC-V |

---

## Quick Start

### 1. Install the RISC-V Target

```sh
rustup target add riscv32imac-unknown-none-elf
```

### 2. Configure `Cargo.toml`

Alias the `hazard3-*` packages to keep standard imports (`embassy_rp::*`, `embassy_time::*`, `embassy_executor::*`):

```toml
[dependencies]
embassy-rp = { package = "hazard3-rp", version = "0.2", features = ["rp235xa", "time-driver", "critical-section-impl", "imagedef-none"] }
embassy-time = { package = "hazard3-time", version = "0.1" }
embassy-executor = { package = "hazard3-executor", version = "0.1", features = ["platform-riscv32", "executor-thread"] }
embassy-sync = { package = "hazard3-sync", version = "0.1" }
rp2350-riscv-linker = "1.0.0"

# Optional sensors & protocols
hazard3-am2302 = "0.8"
hazard3-mqtt-lite = "0.4"
```

### 3. Build for RP2350 Hazard3

```sh
cargo build --release --target riscv32imac-unknown-none-elf
```

---

## Migration from Legacy `*-riscv` Packages

All legacy `*-riscv` packages have been yanked from crates.io and superseded by `hazard3-*`. Keep your dependency keys unchanged and point `package = "..."` to the new crate names:

| Former Package (Yanked) | Current Hazard3 Package |
| :--- | :--- |
| `embassy-rp-riscv` | `hazard3-rp` |
| `rp-pac-riscv` | `hazard3-rp-pac` |
| `embassy-net-riscv` | `hazard3-net-async` |
| `xarxa-riscv` | `hazard3-net` |
| `xarxa-driver-riscv` | `hazard3-net-driver` |
| `embassy-net-driver-channel-riscv` | `hazard3-net-driver-channel` |
| `cyw43-riscv` | `hazard3-cyw43` |
| `cyw43-pio-riscv` | `hazard3-cyw43-pio` |
| `embassy-executor-riscv` | `hazard3-executor` |
| `embassy-executor-macros-riscv` | `hazard3-executor-macros` |
| `embassy-executor-timer-queue-riscv` | `hazard3-executor-timer-queue` |
| `embassy-time-riscv` | `hazard3-time` |
| `embassy-time-driver-riscv` | `hazard3-time-driver` |
| `embassy-time-queue-utils-riscv` | `hazard3-time-queue-utils` |
| `embassy-sync-riscv` | `hazard3-sync` |
| `embassy-embedded-hal-riscv` | `hazard3-embedded-hal` |
| `embassy-usb-driver-riscv` | `hazard3-usb-driver` |
| `embassy-hal-internal-riscv` | `hazard3-hal-internal` |
| `unitrait-riscv` | `hazard3-unitrait` |
| `unitrait-macros-riscv` | `hazard3-unitrait-macros` |

---

## Hardware Validation & Benchmarks

- **RP2350 Core (`hazard3-rp`):** Validated via `cargo check` for `rp235xa` and `rp235xb` on `riscv32imac-unknown-none-elf` (including `RUSTFLAGS="-D unexpected-cfgs"`), multicore via RISC-V `SIO_IRQ_FIFO`, and real hardware execution on Raspberry Pi Pico 2 / Pico 2 W.
- **Wireless & ESP-NOW (`hazard3-cyw43` / `hazard3-cyw43-pio`):** Validated on Raspberry Pi Pico 2 W hardware with bidirectional ESP-NOW PING/PONG against an ESP32-C3 Mini (1,000 exchanges, 0 errors, 13.732 ms average RTT), CYW43 LED control, and OLED display integration.
- **Sensors & MQTT (`Hazard3-bmp280`, `hazard3-am2302`, `hazard3-mqtt-lite`):** Built as standalone `no_std` async crates using `embedded-hal-async` and `embedded-io-async` traits.

---

## Contributing

Bug reports, hardware test feedback, and issues—especially around Hazard3 interrupt handling, peripherals, or new sensor drivers—are welcome on the GitHub issue tracker. Pull requests reintroducing Arm or RP2040 support are out of scope.

---

## License & Acknowledgements

- **Core Embassy-derived crates:** Dual-licensed under Apache License, Version 2.0 (`LICENSE-APACHE`) or MIT License (`LICENSE-MIT`) at your option.
- **Networking crates (`hazard3-net`, `hazard3-net-driver`):** 0-Clause BSD License.
- **PAC (`hazard3-rp-pac`):** BSD-3-Clause License.

`hazard3-async` is an independent project derived from Embassy and Xarxa. All original copyright notices and author credits are preserved. This project is not affiliated with or endorsed by the Embassy, Xarxa, or Raspberry Pi teams.