# Lyrion FPGA-25 Specification Sheet

Working specification for the Spartan-7 XC7S25 platform. Numbers marked **UNVERIFIED** must be confirmed against the relevant datasheet before being treated as final.

---

## 1. FPGA Resources: XC7S25 Spartan-7

| Resource | Value | Notes |
|----------|-------|-------|
| Logic cells | ~23,300 | **UNVERIFIED** (DS890) |
| LUTs | ~14,700 | **UNVERIFIED** |
| Flip-flops | ~29,400 | **UNVERIFIED** |
| DSP48E1 slices | 90 | 25×18 multiply plus 48-bit accumulate |
| Block RAM | ~90 Kb | 36 Kb and 18 Kb configurable blocks; **UNVERIFIED** |
| Distributed RAM | ~ (LUT-based) | derived from LUT count |
| Clock buffers (BUFG) | device-dependent | **UNVERIFIED** |
| I/O (max) | package-dependent | CSGA225 < CSG325 < FGG484; **UNVERIFIED** |
| High-speed transceivers | **0** | Spartan-7 has no GTP/GTH |
| Configuration | QSPI x1/x2/x4, Master SPI | native |

### Ordering code (decided)

| Item | Value |
|------|-------|
| Primary part | `XC7S25-2CSGA225C` |
| Fallback (primary out of stock) | `XC7S25-1CSGA225C` |
| Package | CSGA225, 225 balls, 13×13 mm, 0.8 mm pitch |
| Speed grade | -2 (primary); -1 (fallback) |
| Temperature grade | C: 0 to +85 °C junction |
| Footprint / bitstream | identical across both parts |

**Fallback rule:** when the primary `XC7S25-2CSGA225C` is out of stock, the approved replacement is `XC7S25-1CSGA225C`. Same package, same footprint, same configuration, so the swap is a pure BOM-line change. The two parts differ only in speed grade; a board built with the fallback must close timing at the **-1** grade, and the LVDS maximum data rate characterisation (PLAN Phase 7) must be repeated for the -1 part.

### Rough DSP throughput (design-dependent)

At a nominal 250 MHz fabric clock, 90 DSP48E1 slices give on the order of:

```
90 × 250e6 ≈ 22.5 × 10⁹ multiply-accumulates/s  (real, single-precision-equivalent)
```

A full complex multiply needs about 4 real multiplies (roughly 4 DSP48, or fewer with folding), so complex MAC throughput is roughly a quarter of that. **These are order-of-magnitude figures**, and actual throughput depends on clock closure, pipelining, and resource packing. **UNVERIFIED.**

---

## 2. Memory

### 2.1 Configuration flash: S25FL256L

| Parameter | Value |
|-----------|-------|
| Density | 256 Mbit (32 MB) |
| Interface | Quad-SPI |
| Read clock | up to ~50 MHz std or ~108 MHz fast-quad (**UNVERIFIED**) |
| Quad read rate | ~25 MB/s at 50 MHz, ~54 MB/s at 108 MHz |
| Supply | 3.0 V (family), **UNVERIFIED** ordering code `S25FL256LPNFM010` |
| Package | WSON-8 |

**Configuration time estimate:** the XC7S25 bitstream size is about 12 Mbit (**UNVERIFIED**). Over QSPI x4 at ~50 MHz (about 25 MB/s) that is **under 100 ms**. Confirm with the measured bitstream size and the actual QSPI clock.

### 2.2 Runtime memory: Winbond W958D8 HyperRAM

| Parameter | Value |
|-----------|-------|
| Density | 256 Mbit (32 MB usable) |
| Interface | HyperBus, x8, DDR |
| Supply | 1.8 V |
| Clock | 200 MHz class (**UNVERIFIED** per grade) |
| Peak bandwidth | 200 MHz × 2 × 8 bit = **400 MB/s** |
| Realistic sustained | **about 200 to 300 MB/s** (latency and refresh) |
| Access model | burst-oriented, initial latency per access |
| Controller | **soft (fabric FSM/IP)**, no hard controller in Spartan-7 |

**Bandwidth derivation:**

```
200e6 clk/s × 2 (DDR) × 8 bit = 3.2e9 bit/s = 400 MB/s (peak, contiguous burst)
```

HyperRAM is a register-file memory, so each transaction pays an initial latency of several clocks before bursting. It excels at sequential workloads such as FFT ping-pong, contiguous IQ capture, and frames, and it is poor for scattered small random accesses.

---

## 3. Interface Data Rates

| Interface | Theoretical | Practical sustained | Notes |
|-----------|-------------|---------------------|-------|
| USB 2.0 Hi-Speed (FT2232H) | 480 Mbps (60 MB/s) | about 35 to 40 MB/s aggregate | shared across both channels |
| FT2232H 245-FIFO (per channel) | n/a | about 8 to 12 MB/s | byte mode |
| QSPI flash (x4 at 50 MHz) | 200 Mbps | about 25 MB/s | config and storage |
| HyperBus (x8 at 200 MHz DDR) | 3.2 Gbps | about 200 to 300 MB/s | see §2.2 |
| LVDS SelectIO | about 1.25 Gbps per pin class | design-dependent | ISERDES/OSERDES, speed-grade limited |

### Context: why this is not a full radar-capture board

An AD9643-class dual ADC at 250 MSPS and 14-bit produces:

```
250e6 × 14 bit × 2 ch = 7.0e9 bit/s = 875 MB/s (raw)
```

That exceeds both the HyperRAM sustained bandwidth and the USB link by a wide margin. Full-rate raw capture belongs on a larger Artix board with DDR3 and a real high-speed host path. The FPGA-25 is for **processed and downconverted** data (post-DDC, post-FFT), not raw front-end capture at full rate. This is consistent with the stated scope.

---

## 4. Power

### 4.1 Rails

| Rail | Voltage | Loads | Source |
|------|---------|-------|--------|
| Input | 5 V | board | USB-C |
| Bulk | 3.3 V (intermediate) | distribution | RT7273 buck |
| VCCINT and VCCBRAM | 1.0 V | FPGA core and BRAM | LDO (post-buck) |
| VCCAUX and VCCADC | 1.8 V | FPGA aux and analog | LDO |
| VCCO (banks) | 1.8, 2.5, or 3.3 V | FPGA I/O | per I/O standard |
| HyperRAM | 1.8 V (VCC and VCCQ) | W958D8 | LDO or buck |
| Flash | 3.0 V | S25FL256L | 3.3 V rail |
| FT2232H | 3.3 V (VCORE) plus VIO | USB bridge | 3.3 V rail |
| Module rails | filtered | expansion header | buck plus filtering |

### 4.2 Budget (rough)

| Block | Estimated power |
|-------|-----------------|
| XC7S25 (typical, moderate utilisation) | about 0.3 to 0.8 W (**UNVERIFIED**) |
| HyperRAM (active) | tens of mW |
| FT2232H | about 0.1 to 0.2 W |
| Flash and misc | small |
| **Board total** | **about 1 to 2 W**, roughly 0.2 to 0.4 A at 5 V (**UNVERIFIED**) |

This is well within USB-C 5 V capability. Confirm with the FPGA power estimator (Vivado `report_power`) once a real design exists.

### 4.3 Sequencing

Spartan-7 has explicit ramp and sequencing rules in DS890. The RT7273 and LDO enable ordering must respect them, keeping VCCINT and VCCBRAM relative to VCCAUX and VCCO within the specified limits, to avoid latch-up or configuration failure. **Sequencing is a design task, not an afterthought.**

---

## 5. Estimated BOM Cost

Rough target for a small JLCPCB assembly run, with critical parts sourced separately:

| Item | Role | Approx. cost class |
|------|------|--------------------|
| XC7S25-2CSGA225C (fallback -1CSGA225C) | FPGA | highest |
| W958D8 | HyperRAM | mid |
| FT2232H | USB/JTAG | mid |
| S25FL256L | config flash | mid |
| RT7273, LDOs, and passives | power | low to mid |
| PCB and assembly (BGA/QFN) | fabrication | significant |

**Target: roughly €75 to €90 per board.** This is a **rough estimate and UNVERIFIED.** Final cost depends on PCB layer count, connector choice, assembly fees, and FPGA sourcing. Get real BOM and assembly quotes before committing.

> **FPGA price observation (LCSC, 2026-08-01):** `XC7S25-2CSGA225C` $14.38/1+ (8 pcs in stock), `XC7S25-1CSGA225C` $31.26/1+ (145 pcs in stock). Prices and stock move, so re-verify at order time. Note the fallback is **not** the cheaper part — it is the availability fallback for when the -2C is out of stock.

---

## 6. UNVERIFIED Summary

Confirm before finalising the design:

1. XC7S25 exact resource counts and I/O per package (DS890).
2. User-I/O count for the chosen CSGA225 package (DS890), and timing closure of the fallback -1 speed grade.
3. S25FL256L ordering code and supply voltage (3.0 V family assumed).
4. W958D8 max clock for the specific grade (Winbond datasheet).
5. RT7273 current rating, input range, switching frequency, and package (Richtek datasheet).
6. Spartan-7 power sequencing requirements (DS890) and the resulting enable logic.
7. Bitstream size and the resulting QSPI configuration time.
8. Board power budget (Vivado `report_power` with a real design).
9. BOM and assembly cost (real quotes).
