<p align="center">
  <h1 align="center">Lyrion FPGA-25</h1>
  <p align="center">
    <strong>Spartan-7 FPGA Platform for DSP, SDR & High-Speed Digital Work</strong><br>
    XC7S25 · 32 MB HyperRAM · 256 Mbit QSPI flash · FT2232H USB/JTAG · LVDS expansion
  </p>
</p>

<p align="center">
  <a href="#what-is-it"><img alt="Status: Planning" src="https://img.shields.io/badge/Status-Planning-bf8700?style=flat-square"></a>
  <a href="#core-hardware"><img alt="FPGA: Spartan-7" src="https://img.shields.io/badge/FPGA-Spartan--7-0969da?style=flat-square"></a>
  <a href="#memory"><img alt="Memory: 32 MB HyperRAM" src="https://img.shields.io/badge/Memory-32%20MB%20HyperRAM-6639ba?style=flat-square"></a>
  <a href="#usb-and-debug"><img alt="Interface: USB-JTAG" src="https://img.shields.io/badge/Interface-USB--JTAG-1a7f37?style=flat-square"></a>
  <a href="#license"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-1a7f37?style=flat-square"></a>
</p>

---

## Table of Contents

- [What is it?](#what-is-it)
- [Key Features](#key-features)
- [System Overview](#system-overview)
- [Core Hardware](#core-hardware)
  - [FPGA](#fpga)
  - [Memory](#memory)
  - [USB and Debug](#usb-and-debug)
  - [Expansion](#expansion)
  - [Power System](#power-system)
- [Ecosystem Positioning](#ecosystem-positioning)
- [Design Decisions](#design-decisions)
- [Status & Roadmap](#status--roadmap)
- [Known Limitations & Open Questions](#known-limitations--open-questions)
- [License](#license)

---

## What is it?

Lyrion FPGA-25 is a **compact, general-purpose FPGA development board** built around the **AMD/Xilinx Spartan-7 XC7S25**. It is the first reusable FPGA platform in the Lyrion ecosystem — the shared base for everything that is too fast, too parallel, or too timing-sensitive for an STM32, while staying small and cheap enough to build, break, and learn on.

It is **not** the final radar board and **not** a heavy-compute platform. It is the deterministic hardware-acceleration layer that sits between simple MCU nodes and the later Artix / RK3566-class boards.

Target work:

- FPGA bring-up and timing closure
- FFT / DSP pipeline experiments
- SDR baseband work
- High-speed digital interface bring-up (LVDS ADC/DAC, parallel buses)
- ADC/DAC module testing
- Future Lyrion accelerator boards

---

## Key Features

| Feature | Specification |
|---------|---------------|
| **FPGA** | AMD/Xilinx Spartan-7 XC7S25 |
| **Logic** | ~14,700 LUTs / ~29,400 flip-flops (≈23,300 logic cells) |
| **DSP** | 90 × DSP48E1 slices |
| **Block RAM** | ~90 Kb internal |
| **Config flash** | S25FL256L — 256 Mbit (32 MB) Quad-SPI, WSON-8 |
| **Runtime memory** | Winbond W958D8 — 256 Mbit (32 MB) HyperRAM, x8, 1.8 V, 200 MHz class |
| **USB / debug** | FTDI FT2232H — JTAG, UART, SPI, FIFO streaming (no external programmer) |
| **Expansion** | High-speed header: LVDS data/clock pairs, SPI, I²C, UART, GPIO, power |
| **Power** | Richtek RT7273 buck + post-LDO clean rails |
| **Input** | USB-C (5 V) |
| **Target cost** | ~€75–90 per board (small JLCPCB assembly run) |

> **Resource figures** (LUTs/FF/DSP/BRAM) are the commonly published XC7S25 values. **UNVERIFIED** — confirm the exact numbers and the chosen package/speed grade against the Spartan-7 datasheet (DS890) before finalising the design.

---

## System Overview

```mermaid
flowchart TB
    USBC(["USB-C<br/>5 V"])
    FT["FT2232H<br/>USB ↔ JTAG/UART/SPI/FIFO"]
    FPGA["XC7S25<br/>Spartan-7 FPGA"]
    FLASH[/"S25FL256L<br/>256 Mbit QSPI flash"\]
    HYP[/"W958D8<br/>32 MB HyperRAM"\]
    HDR[/"High-speed<br/>LVDS header"\]
    MOD(["ADC / DAC / RF / SDR<br/>modules"])

    USBC --> FT
    FT -->|JTAG / SPI / FIFO| FPGA
    FPGA -->|QSPI x4| FLASH
    FPGA -->|HyperBus x8| HYP
    FPGA -->|LVDS + control| HDR
    HDR --> MOD

    style USBC fill:#fff,stroke:#24292f,color:#24292f
    style FT fill:#fff,stroke:#24292f,color:#24292f
    style FPGA fill:#fff,stroke:#24292f,color:#24292f
    style FLASH fill:#fff,stroke:#24292f,color:#24292f
    style HYP fill:#fff,stroke:#24292f,color:#24292f
    style HDR fill:#fff,stroke:#24292f,color:#24292f
    style MOD fill:#fff8c5,stroke:#d4a72c,color:#24292f
```

### Block summary

| Block | Function | Key parts |
|-------|----------|-----------|
| **FPGA** | Programmable logic, DSP, interfaces, optional soft CPU | XC7S25 Spartan-7 |
| **Configuration** | Bitstream storage, alternate images, calibration/persistent data | S25FL256L 256 Mbit QSPI flash |
| **Runtime memory** | FFT buffers, IQ sample buffering, DSP scratch, frame storage | W958D8 256 Mbit HyperRAM |
| **USB / debug** | JTAG programming + ILA debug, UART console, SPI control, FIFO streaming | FT2232H |
| **Expansion** | LVDS + control + power to future modules | High-speed header |
| **Power** | Efficient bulk conversion + low-noise analog rails | RT7273 buck + LDOs |

### Data & control flow

1. **USB-C** powers the board and carries the USB 2.0 link to the **FT2232H**.
2. The **FT2232H** exposes the FPGA over **JTAG** (programming + debug), **UART** (console), **SPI** (control), and optionally a **245-FIFO** stream to the PC.
3. The **XC7S25** boots its bitstream from **QSPI flash** (master SPI mode).
4. Running designs use **HyperRAM** over HyperBus for large buffers (FFT, IQ, frames).
5. The **high-speed header** breaks out LVDS data/clock pairs plus SPI/I²C/UART/GPIO/power to ADC, DAC, RF, and SDR modules.

---

## Core Hardware

### FPGA

**AMD/Xilinx Spartan-7 XC7S25** — the main programmable logic device: DSP and interface engine, optional soft-CPU host, and high-speed data handling.

| Parameter | Value |
|-----------|-------|
| Logic cells | ~23,300 |
| LUTs / flip-flops | ~14,700 / ~29,400 |
| DSP48E1 slices | 90 |
| Block RAM | ~90 Kb |
| Configuration | QSPI (x1/x2/x4), Master SPI |
| High-speed serial transceivers | **None** (Spartan-7 has no GTP/GTH) |
| SelectIO | LVDS_25, OSERDES/ISERDES, ~1.25 Gbps class |

**Why this device:** low cost versus larger Artix parts, enough logic and DSP for real FFT/filtering work, enough BRAM for internal buffering, and a good learning target before moving to bigger boards.

> **Critical architectural fact:** Spartan-7 has **no hard high-speed serial transceivers**. There is no native PCIe / USB3 / GigE / SFP / JESD204 — those require an external PHY over SelectIO, or a larger Artix/Kintex. High-speed work on this board means **parallel LVDS**, not multi-gigabit serial. This is by design and matches the board's scope.

> **Package / speed grade — open decision.** Candidate packages include CSGA225 (225-ball, 0.8 mm pitch, most compact), CSG325 (325-ball, 0.8 mm), and FGG484 (484-ball, 1.0 mm, easiest to route and hand-assemble). **UNVERIFIED** — confirm I/O counts per package and pick based on the assembly capability you actually have. 0.8 mm BGA is not a friendly hand-solder target.

### Memory

#### Configuration flash — S25FL256L

| Parameter | Value |
|-----------|-------|
| Density | 256 Mbit (32 MB) |
| Interface | Quad-SPI (QSPI) |
| Package | WSON-8 |
| Supply | 3.0 V (S25FL256L family) |

Planned use: boot configuration, alternate FPGA images, calibration tables, small persistent settings.

> **UNVERIFIED** — the exact ordering code (`S25FL256LPNFM010`) and the 3.0 V supply must be confirmed against the datasheet. If the flash is 3.0 V, the FPGA configuration bank (and the flash VCC) must run at 3.3 V — plan the power tree and bank voltage accordingly.

#### Runtime memory — Winbond W958D8 HyperRAM

| Parameter | Value |
|-----------|-------|
| Density | 256 Mbit (32 MB usable) |
| Interface | HyperBus, x8 |
| Supply | 1.8 V |
| Clock | 200 MHz class (DDR) |
| Peak bandwidth | ~400 MB/s (200 MHz × 2 × 8 bit) |

Planned use: FFT buffers, IQ sample buffering, DSP scratch, frame storage, soft-CPU memory expansion.

> **HyperRAM is not low-latency random-access SRAM.** It is a burst-oriented, register-file memory with an initial access latency (several clocks) before each burst. It is **excellent** for sequential/burst workloads (FFT ping-pong buffers, contiguous IQ capture, frame storage) and **poor** for scattered small random accesses. Sustained throughput with latency and refresh overhead is realistically ~200–300 MB/s, not the 400 MB/s peak.
>
> Spartan-7 has **no hard HyperBus controller** — the interface must be implemented in fabric (soft IP or a custom FSM). **UNVERIFIED** — confirm the W958D8 max clock for the specific grade from the Winbond datasheet.

### USB and Debug

**FTDI FT2232H** — dual-channel USB 2.0 Hi-Speed (480 Mbps) bridge.

| Channel use | Mode |
|-------------|------|
| JTAG programming + ILA debug | MPSSE (supported by Vivado hw_server / openocd) |
| UART console | UART |
| SPI control | MPSSE SPI |
| PC streaming (optional) | 245 synchronous FIFO |

This means the board needs **no external programmer** for normal development.

> **Throughput reality check.** Practical 245-FIFO throughput is on the order of **8–12 MB/s per channel**, with the two channels sharing one USB 2.0 Hi-Speed link (~35–40 MB/s aggregate, best case). This is ample for JTAG, console, control, and modest IQ/spectrum streaming. It is **not** a path for sustained multi-hundred-MB/s raw capture (e.g. a 250 MSPS × 14-bit × 2 radar stream is ~700 MB/s) — that workload does not belong on this board, which is consistent with the stated scope.

### Expansion

A high-speed connector turns the board into a **platform** rather than a one-off dev board.

| Signal class | Examples |
|--------------|----------|
| High-speed data | LVDS data pairs, LVDS clock pairs |
| Control | SPI, I²C, UART, GPIO |
| Power | Module rails (filtered) |

Use cases: ADC modules, DAC modules, SDR/IQ modules, camera or sensor interfaces, custom Lyrion modules.

> **Signal-integrity note.** LVDS interfaces (e.g. an AD9643-class ADC at 250 MSPS DDR) are feasible on Spartan-7 SelectIO using ISERDES/OSERDES, but timing closure at speed grade −1 needs care, controlled-impedance routing, and a **proper high-density connector** (not 2.54 mm pin header) to preserve the differential pairs. Connector and pinout selection is an open task.

### Power System

| Stage | Part | Role |
|-------|------|------|
| Bulk conversion | Richtek RT7273 | Efficient synchronous step-down from USB-C 5 V |
| Clean rails | LDOs (TBD) | Low-noise post-regulation for analog-sensitive rails |
| Module rails | Filtered | Future ADC/DAC/RF module supplies |

Spartan-7 rail set to provide: **VCCINT 1.0 V**, **VCCBRAM 1.0 V**, **VCCAUX 1.8 V**, **VCCADC 1.8 V**, and **VCCO** bank rails at 1.8 / 2.5 / 3.3 V as required by the chosen I/O standards (HyperRAM 1.8 V, QSPI flash 3.0/3.3 V, FT2232H 3.3 V).

> **Power sequencing is a first-class design task.** Spartan-7 has explicit ramp/sequencing recommendations (DS890). The RT7273 + LDO enable ordering must respect them, or the device can latch up or fail to configure. **UNVERIFIED** — confirm the RT7273 exact specs (current rating, input range, switching frequency, package) from the Richtek datasheet.

---

## Ecosystem Positioning

The FPGA-25 is the **middle layer** between simple MCU nodes and heavy compute boards.

| Platform | Role | Typical work |
|----------|------|--------------|
| **STM32 boards** | Control / sensing / comms / low-power nodes | CC1101, LR2021 radio devices, general control |
| **FPGA-25 (this board)** | Deterministic hardware acceleration | DSP, FFT, SDR baseband, high-speed interfaces |
| **Later Artix / RK3566 boards** | Heavy compute | Serious radar/SDR, Linux networking, camera processing |

---

## Design Decisions

### 1. Spartan-7 XC7S25 (not Artix-7)

Artix-7 (used on the Lyrion Radar) brings more logic, more DSP, more BRAM, and — critically — hard GTP transceivers. That is overkill and over-cost for a learning/experiment platform. The XC7S25 is cheap, has enough resources for real FFT/filtering and interface work, and is the right size to bring up and master before committing to larger, more expensive boards. The trade-off (no transceivers, fewer resources) is deliberate and matches the scope.

### 2. HyperRAM (not DDR3) for runtime memory

DDR3 gives higher sustained bandwidth and true random access, but demands a hard memory controller (MIG), more pins, tighter routing, and more power. HyperRAM uses a compact x8 HyperBus, far fewer pins, and is trivial to route relative to DDR. For the target workloads — sequential FFT buffers, contiguous IQ capture, frame storage — burst-oriented HyperRAM is a good fit and keeps the board small and buildable. The cost is latency and no hard controller (implemented in fabric).

### 3. FT2232H for USB/JTAG (no external programmer)

A single FT2232H provides JTAG (Vivado/openocd), UART console, SPI control, and optional FIFO streaming over one USB-C cable. This removes the need for a separate programmer and makes the board self-contained for daily development. The accepted limitation is USB 2.0 Hi-Speed throughput — fine for debug and modest streaming, not for raw high-rate capture.

### 4. QSPI flash for configuration and storage

A 256 Mbit QSPI flash holds the boot bitstream plus room for alternate images, calibration tables, and persistent settings. QSPI is natively supported for Spartan-7 master configuration, and 32 MB is generous for this class of device.

### 5. Switching buck + LDO clean rails

The RT7273 buck does efficient bulk conversion from USB-C 5 V; LDOs clean up the analog-sensitive rails afterward. This is the standard cost/efficiency/noise compromise: switching for efficiency and heat, LDOs where noise matters (and for future ADC/DAC/RF module rails).

### 6. High-speed LVDS expansion header

Breaking out LVDS data/clock pairs plus control and power makes the board a reusable platform for future Lyrion modules (ADC, DAC, SDR/IQ, sensors) instead of a single-purpose dev board. This is the feature that gives the board a life beyond its first bring-up.

---

## Status & Roadmap

| Phase | Status |
|-------|--------|
| Architecture definition | ✅ Done |
| Core component selection | ✅ Done |
| Package / speed-grade selection | ⬜ Pending |
| Power tree + sequencing design | ⬜ Pending |
| Schematic capture | ⬜ Pending |
| PCB layout (controlled-Z LVDS, BGA fanout) | ⬜ Pending |
| Prototype fabrication + assembly | ⬜ Pending |
| Bring-up: power sequencing + JTAG | ⬜ Pending |
| QSPI config + flash verify | ⬜ Pending |
| HyperRAM controller + bandwidth test | ⬜ Pending |
| FT2232H FIFO / UART / SPI verify | ⬜ Pending |
| LVDS expansion loopback test | ⬜ Pending |
| First DSP/FFT demo design | ⬜ Pending |

---

## Known Limitations & Open Questions

| Issue | Status |
|-------|--------|
| **No high-speed serial transceivers** | Spartan-7 has no GTP/GTH. No native PCIe/USB3/GigE/SFP/JESD204 — parallel LVDS only. By design. |
| **XC7S25 resource figures** | LUT/FF/DSP/BRAM values are commonly published numbers — **confirm against DS890** before finalising. |
| **Package / speed grade not chosen** | 0.8 mm BGA (CSGA225/325) is compact but hard to hand-assemble; FGG484 (1.0 mm) is easier. Decide from real assembly capability. |
| **HyperRAM ≠ random-access SRAM** | Burst-oriented with initial latency; implement HyperBus controller in fabric; sustained ~200–300 MB/s realistic. |
| **W958D8 max clock** | 200 MHz class assumed — **confirm grade from Winbond datasheet.** |
| **S25FL256L supply / ordering code** | 3.0 V family assumed; exact ordering code and config-bank voltage **must be confirmed.** |
| **FT2232H throughput ceiling** | USB 2.0 HS limits sustained streaming to ~tens of MB/s; not for raw high-rate capture. |
| **RT7273 specs** | Current rating, input range, frequency, package **TBD from Richtek datasheet.** |
| **Power sequencing** | Spartan-7 ramp/sequence rules (DS890) must be met by the RT7273 + LDO enable ordering. |
| **LVDS timing closure at speed grade −1** | High-speed ADC interfaces need careful constraints, controlled-Z routing, and a proper connector. |
| **Cost estimate** | ~€75–90/board is a rough target for a small JLCPCB run — **verify** against real BOM + assembly quotes. |

---

## License

MIT License — see [LICENSE](LICENSE) for details.
