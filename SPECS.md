# Lyrion FPGA-25 — Specification Sheet

Working specification for the Spartan-7 XC7S25 platform. Numbers marked **UNVERIFIED** must be confirmed against the relevant datasheet before being treated as final.

---

## 1. FPGA Resources — XC7S25 Spartan-7

| Resource | Value | Notes |
|----------|-------|-------|
| Logic cells | ~23,300 | **UNVERIFIED** (DS890) |
| LUTs | ~14,700 | **UNVERIFIED** |
| Flip-flops | ~29,400 | **UNVERIFIED** |
| DSP48E1 slices | 90 | 25×18 multiply + 48-bit accumulate |
| Block RAM | ~90 Kb | 36 Kb / 18 Kb configurable blocks; **UNVERIFIED** |
| Distributed RAM | ~ (LUT-based) | derived from LUT count |
| Clock buffers (BUFG) | device-dependent | **UNVERIFIED** |
| I/O (max) | package-dependent | CSGA225 < CSG325 < FGG484; **UNVERIFIED** |
| High-speed transceivers | **0** | Spartan-7 has no GTP/GTH |
| Configuration | QSPI x1/x2/x4, Master SPI | native |

### Rough DSP throughput (design-dependent)

At a nominal 250 MHz fabric clock, 90 DSP48E1 slices give on the order of:

```
90 × 250e6 ≈ 22.5 × 10⁹ multiply-accumulates/s  (real, single-precision-equivalent)
```

A full complex multiply needs ~4 real multiplies (≈4 DSP48, or fewer with folding), so complex MAC throughput is roughly a quarter of that. **These are order-of-magnitude figures** — actual throughput depends on clock closure, pipelining, and resource packing. **UNVERIFIED.**

---

## 2. Memory

### 2.1 Configuration flash — S25FL256L

| Parameter | Value |
|-----------|-------|
| Density | 256 Mbit (32 MB) |
| Interface | Quad-SPI |
| Read clock | up to ~50 MHz std / ~108 MHz fast-quad (**UNVERIFIED**) |
| Quad read rate | ~25 MB/s @ 50 MHz, ~54 MB/s @ 108 MHz |
| Supply | 3.0 V (family) — **UNVERIFIED** ordering code `S25FL256LPNFM010` |
| Package | WSON-8 |

**Configuration time estimate:** XC7S25 bitstream size ≈ ~12 Mbit (**UNVERIFIED**). Over QSPI x4 at ~50 MHz (~25 MB/s): ≈ **< 100 ms**. Confirm with measured bitstream size and actual QSPI clock.

### 2.2 Runtime memory — Winbond W958D8 HyperRAM

| Parameter | Value |
|-----------|-------|
| Density | 256 Mbit (32 MB usable) |
| Interface | HyperBus, x8, DDR |
| Supply | 1.8 V |
| Clock | 200 MHz class (**UNVERIFIED** per grade) |
| Peak bandwidth | 200 MHz × 2 × 8 bit = **400 MB/s** |
| Realistic sustained | **~200–300 MB/s** (latency + refresh) |
| Access model | burst-oriented, initial latency per access |
| Controller | **soft (fabric FSM/IP)** — no hard controller in Spartan-7 |

**Bandwidth derivation:**

```
200e6 clk/s × 2 (DDR) × 8 bit = 3.2e9 bit/s = 400 MB/s (peak, contiguous burst)
```

HyperRAM is a register-file memory: each transaction pays an initial latency (several clocks) before bursting. It excels at sequential workloads (FFT ping-pong, contiguous IQ capture, frames) and is poor for scattered small random accesses.

---

## 3. Interface Data Rates

| Interface | Theoretical | Practical sustained | Notes |
|-----------|-------------|---------------------|-------|
| USB 2.0 Hi-Speed (FT2232H) | 480 Mbps (60 MB/s) | ~35–40 MB/s aggregate | shared across both channels |
| FT2232H 245-FIFO (per channel) | — | ~8–12 MB/s | byte mode |
| QSPI flash (x4 @ 50 MHz) | 200 Mbps | ~25 MB/s | config + storage |
| HyperBus (x8 @ 200 MHz DDR) | 3.2 Gbps | ~200–300 MB/s | see §2.2 |
| LVDS SelectIO | ~1.25 Gbps/pin class | design-dependent | ISERDES/OSERDES, speed-grade limited |

### Context: why this is not a full radar-capture board

An AD9643-class dual ADC at 250 MSPS, 14-bit produces:

```
250e6 × 14 bit × 2 ch = 7.0e9 bit/s = 875 MB/s (raw)
```

That exceeds both the HyperRAM sustained bandwidth and the USB link by a wide margin. Full-rate raw capture belongs on a larger Artix board with DDR3 + a real high-speed host path. The FPGA-25 is for **processed/downconverted** data (post-DDC, post-FFT), not raw front-end capture at full rate. This is consistent with the stated scope.

---

## 4. Power

### 4.1 Rails

| Rail | Voltage | Loads | Source |
|------|---------|-------|--------|
| Input | 5 V | board | USB-C |
| Bulk | 3.3 V (intermediate) | distribution | RT7273 buck |
| VCCINT / VCCBRAM | 1.0 V | FPGA core / BRAM | LDO (post-buck) |
| VCCAUX / VCCADC | 1.8 V | FPGA aux / analog | LDO |
| VCCO (banks) | 1.8 / 2.5 / 3.3 V | FPGA I/O | per I/O standard |
| HyperRAM | 1.8 V (VCC + VCCQ) | W958D8 | LDO / buck |
| Flash | 3.0 V | S25FL256L | 3.3 V rail |
| FT2232H | 3.3 V (VCORE) + VIO | USB bridge | 3.3 V rail |
| Module rails | filtered | expansion header | buck + filtering |

### 4.2 Budget (rough)

| Block | Estimated power |
|-------|-----------------|
| XC7S25 (typical, moderate util.) | ~0.3–0.8 W (**UNVERIFIED**) |
| HyperRAM (active) | ~tens of mW |
| FT2232H | ~0.1–0.2 W |
| Flash + misc | small |
| **Board total** | **~1–2 W** → ~0.2–0.4 A @ 5 V (**UNVERIFIED**) |

Well within USB-C 5 V capability. Confirm with the FPGA power estimator (Vivado `report_power`) once a real design exists.

### 4.3 Sequencing

Spartan-7 has explicit ramp/sequencing rules (DS890). The RT7273 + LDO enable ordering must respect them (VCCINT/VCCBRAM relative to VCCAUX/VCCO within specified limits) to avoid latch-up or configuration failure. **Sequencing is a design task, not an afterthought.**

---

## 5. Estimated BOM Cost

Rough target for a small JLCPCB assembly run, critical parts sourced separately:

| Item | Role | Approx. cost class |
|------|------|--------------------|
| XC7S25 | FPGA | highest |
| W958D8 | HyperRAM | mid |
| FT2232H | USB/JTAG | mid |
| S25FL256L | config flash | mid |
| RT7273 + LDOs + passives | power | low–mid |
| PCB + assembly (BGA/QFN) | fabrication | significant |

**Target: ~€75–90 per board.** This is a **rough estimate — UNVERIFIED.** Final cost depends on PCB layer count, connector choice, assembly fees, and FPGA sourcing. Get real BOM + assembly quotes before committing.

---

## 6. UNVERIFIED Summary

Confirm before finalising the design:

1. XC7S25 exact resource counts and I/O per package (DS890).
2. Package + speed-grade selection vs. real assembly capability.
3. S25FL256L ordering code and supply voltage (3.0 V family assumed).
4. W958D8 max clock for the specific grade (Winbond datasheet).
5. RT7273 current rating, input range, switching frequency, package (Richtek datasheet).
6. Spartan-7 power sequencing requirements (DS890) and resulting enable logic.
7. Bitstream size and resulting QSPI configuration time.
8. Board power budget (Vivado `report_power` with a real design).
9. BOM + assembly cost (real quotes).
