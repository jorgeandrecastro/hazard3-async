# hazard3-async

[![Target](https://img.shields.io/badge/target-riscv32imac--unknown--none--elf-blue)](#quick-start)
[![Chip](https://img.shields.io/badge/chip-RP2350%20Hazard3-green)](#what-is-hazard3-async)
[![no_std](https://img.shields.io/badge/no__std-async-orange)](#ecosystem-at-a-glance)
[![License](https://img.shields.io/badge/license-MIT%20%2F%20Apache--2.0-lightgrey)](#license)

> **An independent async Rust ecosystem for the RP2350's Hazard3 RISC-V cores.**
>
> Built and maintained by [Jorge Andre Castro](https://github.com/jorgeandrecastro).
> Project hub: [github.com/jorgeandrecastro/Hazard3](https://github.com/jorgeandrecastro/Hazard3).

A big thank-you to the Rust embedded community and to everyone building, testing, documenting, and sharing open-source tools. Your work makes projects like this possible.

`hazard3-async` brings together a RISC-V-focused runtime, hardware access, networking, and optional building blocks for sensors, displays, and embedded applications. Use only the pieces your project needs.

---

## Start here

| If you want to… | Start with… |
|---|---|
| Build firmware for the RP2350 Hazard3 core | [Quick start](#quick-start) |
| Use async tasks, timers, and synchronization | [Core runtime](#core-runtime-and-hal) |
| Connect a Pico 2 W to Wi-Fi or use ESP-NOW | [Networking](#networking) |
| Read sensors or handle input | [Sensors and input](#sensors-and-input) |
| Drive an OLED or color TFT | [Displays](#displays) |
| Use math helpers or DMA cancellation safety | [Utilities](#utilities) |
| Understand which packages work together | [Compatibility guidance](#compatibility-guidance) |

## What is hazard3-async?

`hazard3-async` is an ecosystem of independently published Rust crates targeting the **Hazard3 RISC-V cores in the RP2350**.

Its design goals are:

- **RISC-V-first:** focused on `riscv32imac-unknown-none-elf` and RP2350 Hazard3 hardware, not a universal RP2040/Arm HAL.
- **Async and `no_std`:** designed for bare-metal firmware and async embedded applications.
- **Modular:** use the runtime and HAL as a foundation, then add only the drivers and utilities required by your board.
- **Familiar APIs:** several packages retain familiar Rust crate/library names through Cargo dependency aliases.
- **Independent releases:** drivers and utilities can evolve without requiring every application to adopt every component.

This is an ecosystem, not a single monolithic framework. Each crate has its own API, feature flags, version, and documentation. Check the linked crate page before choosing a dependency.

## Ecosystem at a glance

```text
Your firmware
├── Application logic
├── Sensors, input and protocols
│   ├── BMP280 · AM2302 · GL5528 · GY-BMI160 · TTP223B
│   ├── Hall sensor · MQTT · ESP networking helpers
│   └── DMA cancellation guard
├── Displays
│   ├── SSD1306 → graphics → physics / 3D
│   └── ST7789V → sprites / plotting
├── Networking
│   ├── CYW43 → PIO transport → Wi-Fi / BLE / ESP-NOW
│   └── async network stack
├── Async runtime
│   └── executor · time · sync · embedded-hal adapters
├── RP2350 Hazard3 HAL
└── PAC · linker · board configuration
```

The diagram is a conceptual map, not a dependency graph: optional crates do not necessarily depend on one another.

## Quick start

### 1. Install the target

```sh
rustup target add riscv32imac-unknown-none-elf
```

### 2. Add the core dependencies

The package names below follow the ecosystem's Cargo aliases. Confirm feature names and compatible versions on each crate's crates.io page.

```toml
[dependencies]
embassy-rp = {
    package = "hazard3-rp",
    version = "0.2",
    features = ["rp235xa", "time-driver", "critical-section-impl", "imagedef-none"]
}

embassy-time = { package = "hazard3-time", version = "0.1" }
embassy-executor = {
    package = "hazard3-executor",
    version = "0.1",
    features = ["platform-riscv32", "executor-thread"]
}
embassy-sync = { package = "hazard3-sync", version = "0.1" }

rp2350-riscv-linker = "0.2"
```

> **Versioning tip:** the versions above reflect the example dependency set in this project README. Before copying them into a new firmware project, check the current published versions and feature names on crates.io. Keep the core `hazard3-*` runtime crates on a mutually compatible release set.

### 3. Build

```sh
cargo build --release --target riscv32imac-unknown-none-elf
```

For board-specific linker, memory, and runner configuration, follow the instructions provided by `hazard3-rp` and `rp2350-riscv-linker`.

## Core runtime and HAL

These crates provide the foundation for an async RP2350 Hazard3 application.

| Crate | Library name / alias | What it provides |
|---|---|---|
| [`hazard3-rp`](https://crates.io/crates/hazard3-rp) | `embassy_rp` | RP2350 async HAL for Hazard3 RISC-V |
| [`hazard3-rp-pac`](https://crates.io/crates/hazard3-rp-pac) | `rp_pac` | RP2350 peripheral access crate |
| [`hazard3-executor`](https://crates.io/crates/hazard3-executor) | `embassy_executor` | Async task executor |
| [`hazard3-executor-macros`](https://crates.io/crates/hazard3-executor-macros) | `embassy_executor_macros` | Executor procedural macros |
| [`hazard3-executor-timer-queue`](https://crates.io/crates/hazard3-executor-timer-queue) | `embassy_executor_timer_queue` | Executor timer queue backend |
| [`hazard3-time`](https://crates.io/crates/hazard3-time) | `embassy_time` | Timers, delays, and duration types |
| [`hazard3-time-driver`](https://crates.io/crates/hazard3-time-driver) | `embassy_time_driver` | Time-driver traits |
| [`hazard3-time-queue-utils`](https://crates.io/crates/hazard3-time-queue-utils) | `embassy_time_queue_utils` | Timer queue utilities |
| [`hazard3-sync`](https://crates.io/crates/hazard3-sync) | `embassy_sync` | Channels, mutexes, signals, and synchronization |
| [`hazard3-embedded-hal`](https://crates.io/crates/hazard3-embedded-hal) | `embassy_embedded_hal` | Embedded-HAL adapters |
| [`hazard3-usb-driver`](https://crates.io/crates/hazard3-usb-driver) | `embassy_usb_driver` | USB device-driver traits |
| [`hazard3-hal-internal`](https://crates.io/crates/hazard3-hal-internal) | `embassy_hal_internal` | Shared HAL internals |
| [`hazard3-unitrait`](https://crates.io/crates/hazard3-unitrait) | `unitrait` | Shared implementation traits |
| [`hazard3-unitrait-macros`](https://crates.io/crates/hazard3-unitrait-macros) | `unitrait_macros` | Procedural macros for `unitrait` |

## Networking

### CYW43: Wi-Fi, BLE and ESP-NOW

| Crate | What it provides |
|---|---|
| [`hazard3-cyw43`](https://crates.io/crates/hazard3-cyw43) | CYW43 wireless driver, including the native ESP-NOW work described below |
| [`hazard3-cyw43-pio`](https://crates.io/crates/hazard3-cyw43-pio) | RP2350 PIO transport for the CYW43 |
| [`hazard3-net-async`](https://crates.io/crates/hazard3-net-async) | Async TCP/UDP/DNS networking stack |
| [`hazard3-net`](https://crates.io/crates/hazard3-net) | Xarxa-derived network stack |
| [`hazard3-net-driver`](https://crates.io/crates/hazard3-net-driver) | Network-driver interface |
| [`hazard3-net-driver-channel`](https://crates.io/crates/hazard3-net-driver-channel) | Channel-based network driver implementation |

### Native ESP-NOW on Pico 2 W

The `hazard3-cyw43` and `hazard3-cyw43-pio` crates provide the basis for native ESP-NOW communication on the **Raspberry Pi Pico 2 W** (RP2350 Hazard3 + CYW43439).

This is useful when devices need direct peer-to-peer wireless messaging without depending on a conventional Wi-Fi network connection.

**Hardware validation reported by this project:**

- Bidirectional PING/PONG communication with an ESP32-C3 Mini.
- 1,000 exchanges.
- 0 TX errors, 0 invalid/unexpected PONGs, and 0 timeouts.
- Average round-trip time: **13.732 ms**.

These figures describe the reported test setup; they are not a guarantee of latency or delivery under different radio conditions.

### ESP networking helper

[`esp-net-easy`](https://crates.io/crates/esp-net-easy) is listed as an optional networking helper for ESP-based projects. Consult its crate documentation for its supported chips, API, and compatibility requirements. It is not presented as a replacement for the RP2350 Hazard3 CYW43 stack.

## Sensors and input

Standalone drivers let applications add hardware support without pulling in unrelated display or networking components.

| Crate | Device / purpose |
|---|---|
| [`hazard3-bmp280`](https://crates.io/crates/hazard3-bmp280) | Bosch BMP280 temperature and pressure sensor |
| [`hazard3-am2302`](https://crates.io/crates/hazard3-am2302) | AM2302 / DHT22 temperature and humidity sensor |
| [`hazard3-gl5528`](https://crates.io/crates/hazard3-gl5528) | GL5528 photoresistor (LDR) |
| [`hazard3-gy-bmi160`](https://crates.io/crates/hazard3-gy-bmi160) | GY-BMI160 inertial measurement module |
| [`hazard3-ttp223b`](https://crates.io/crates/hazard3-ttp223b) | TTP223B capacitive touch sensor |
| [`hazard3-hall-analog`](https://crates.io/crates/hazard3-hall-analog) | Hall-effect sensor / analog sensing crate |
| [`hazard3-mqtt-lite`](https://crates.io/crates/hazard3-mqtt-lite) | Lightweight MQTT client for embedded applications |

For wiring, bus requirements, supported sensor variants, and feature flags, use each crate's own documentation. The table identifies the intended component; it does not imply that every driver has been validated on every board.

## Displays

Choose a display driver first. Add optional rendering layers only if your application needs them.

### SSD1306 OLED

| Crate | Layer | What it provides |
|---|---|---|
| [`hazard3-ssd1306`](https://crates.io/crates/hazard3-ssd1306) | Driver | SSD1306 OLED display driver |
| [`hazard3-ssd1306-graphics`](https://crates.io/crates/hazard3-ssd1306-graphics) | Graphics | Drawing and rendering helpers |
| [`hazard3-ssd1306-physics`](https://crates.io/crates/hazard3-ssd1306-physics) | Physics | Lightweight physics / animation layer |
| [`hazard3-ssd1306-3d`](https://crates.io/crates/hazard3-ssd1306-3d) | 3D | 3D rendering helpers |

```text
ssd1306-3d ───────┐
                  ├──> ssd1306-graphics ──> ssd1306 driver ──> RP2350 I2C/SPI
ssd1306-physics ──┘
```

### ST7789V color TFT

| Crate | Layer | What it provides |
|---|---|---|
| [`hazard3-st7789v`](https://crates.io/crates/hazard3-st7789v) | Driver | ST7789V color TFT/LCD driver |
| [`hazard3-st7789v-sprite`](https://crates.io/crates/hazard3-st7789v-sprite) | Sprites | Sprite rendering and framebuffer-based animation |
| [`hazard3-st7789v-plot`](https://crates.io/crates/hazard3-st7789v-plot) | Plotting | Cartesian plots and graphing helpers |

```text
st7789v-sprite ──┐
                 ├──> st7789v driver ──> RP2350 SPI
st7789v-plot ────┘
```

The rendering crates are optional. Start with the driver if you only need to initialize the display and draw pixels.

## Utilities

| Crate | What it provides |
|---|---|
| [`embedded-maths`](https://crates.io/crates/embedded-maths) | Mathematical building blocks for embedded applications |
| [`dma-cancel-guard`](https://crates.io/crates/dma-cancel-guard) | Guard utility for handling DMA operation cancellation safely; see crate documentation for its exact guarantees and integration model |

## Tooling and linker

| Crate | What it provides |
|---|---|
| [`rp2350-riscv-linker`](https://crates.io/crates/rp2350-riscv-linker) | Linker support for RP2350 Hazard3 RISC-V firmware |
| [`rp2350-linker`](https://crates.io/crates/rp2350-linker) | Separate RP2350 linker package; check its target architecture and intended use before selecting it |

**Important:** use the RISC-V linker package documented for your Hazard3 target. Do not assume that two packages with similar names target the same architecture or are interchangeable.

## Compatibility guidance

A few practical rules help avoid dependency and integration problems:

1. **Keep the core runtime set aligned.** The HAL, executor, time, sync, and networking crates should use compatible versions.
2. **Do not add every crate by default.** Add only the drivers and utilities your firmware actually uses.
3. **Check target support.** A crate published in the wider ecosystem is not automatically compatible with `riscv32imac-unknown-none-elf`.
4. **Check `no_std` and async support.** Verify the crate's features and trait requirements before integrating it into a bare-metal application.
5. **Treat helper crates as optional.** `esp-net-easy`, `dma-cancel-guard`, and `embedded-maths` are separate utilities, not mandatory dependencies of the Hazard3 runtime.
6. **Use each crate's own docs.** The crates.io page and docs.rs documentation are the source of truth for current versions, features, and examples.

## Migration from legacy `*-riscv` packages

The following package names are the legacy names listed by this project. Replace the package name in your dependency declaration while keeping your Rust import alias where appropriate.

| Legacy package | Current package |
|---|---|
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

## Hardware validation and benchmarks

The project reports the following validation results:

- **RP2350 core (`hazard3-rp`):** `cargo check` for `rp235xa` and `rp235xb` on `riscv32imac-unknown-none-elf`, including `RUSTFLAGS="-D unexpected-cfgs"`, multicore work using RISC-V `SIO_IRQ_FIFO`, and execution on Raspberry Pi Pico 2 / Pico 2 W hardware.
- **Wireless (`hazard3-cyw43` / `hazard3-cyw43-pio`):** bidirectional ESP-NOW PING/PONG against an ESP32-C3 Mini, plus CYW43 LED control and OLED integration.
- **ESP-NOW benchmark:** 1,000 exchanges, 0 TX errors, 0 invalid/unexpected PONGs, 0 timeouts, and 13.732 ms average RTT.
- **Other crates:** sensor, display, math, and utility packages are listed as independent ecosystem components. Their individual hardware-validation status should be checked in their respective repositories and crate documentation.

## Contributing

Issues, reproducible bug reports, documentation improvements, and hardware test results are welcome.

Useful contributions include:

- Testing on additional RP2350 boards and peripherals.
- Reporting target, feature, and dependency compatibility.
- Improving examples and onboarding documentation.
- Adding standalone drivers and small, reusable embedded utilities.

The Hazard3-specific core is intentionally RISC-V-focused. Changes that reintroduce unrelated Arm or RP2040 support into that core are outside its scope.

## License

The ecosystem contains crates with different upstream origins and license terms. Check the license metadata and notices of each individual crate before redistributing it.

The project README previously identified these license families:

- Core Embassy-derived crates: MIT OR Apache-2.0.
- `hazard3-net` and `hazard3-net-driver`: 0-Clause BSD.
- `hazard3-rp-pac`: BSD-3-Clause.

Preserve the original license and copyright notices of every upstream-derived component. This project is independent and is not affiliated with or endorsed by the Embassy, Xarxa, or Raspberry Pi teams.

## Acknowledgements

Thanks to the Rust embedded community, the maintainers of the upstream projects this ecosystem builds upon, and everyone who tests hardware, reports issues, and shares improvements.