# Lyrion FPGA-25 — Plan

Phased roadmap from architecture to a working, characterised board. Each phase has an exit criterion and, where relevant, a bring-up measurement so progress is judged by hardware behaviour, not by "the schematic looks done."

---

## Phase 0 — Architecture & Part Confirmation  ✅ (mostly done)

Lock the architecture and convert every **UNVERIFIED** item (see SPECS.md §6) into a confirmed datasheet fact.

- [x] Top-level architecture (FPGA + flash + HyperRAM + FT2232H + LVDS header)
- [x] Core part selection (XC7S25, S25FL256L, W958D8, FT2232H, RT7273)
- [ ] Confirm XC7S25 resource counts and I/O per candidate package (DS890)
- [ ] **Choose package + speed grade** (CSGA225 / CSG325 / FGG484) from real assembly capability
- [ ] Confirm S25FL256L ordering code + supply voltage
- [ ] Confirm W958D8 max clock for the chosen grade
- [ ] Confirm RT7273 specs (current, input range, frequency, package)

**Exit:** every part has a confirmed datasheet line; package and speed grade are chosen.

---

## Phase 1 — Power Tree & Sequencing

Design the power system before anything else, because Spartan-7 sequencing constrains it.

- [ ] Map all rails: 5 V in → 3.3 V buck → 1.0 V (VCCINT/VCCBRAM), 1.8 V (VCCAUX/VCCADC/HyperRAM), 3.3 V (VCCO/FT2232H/flash)
- [ ] Design RT7273 buck (inductor, caps, feedback, layout per datasheet)
- [ ] Select LDOs for clean rails (noise, PSRR, current)
- [ ] Implement sequencing/enable logic to meet DS890 ramp rules
- [ ] Add decoupling per Spartan-7 PDN guidance; plan PDN impedance
- [ ] Add test points on every rail

**Exit:** power tree simulated/validated on paper; sequencing order documented and within DS890 limits.

---

## Phase 2 — Schematic Capture

- [ ] FPGA symbol + all power/config/JTAG/MCLR nets
- [ ] QSPI flash (S25FL256L) on the configuration bank
- [ ] HyperRAM (W958D8) x8 HyperBus + clock + reset
- [ ] FT2232H: USB-C, JTAG (channel A), UART/SPI/FIFO (channel B), EEPROM
- [ ] High-speed LVDS expansion header (data/clock pairs + control + power)
- [ ] Power tree from Phase 1
- [ ] ESD/TVS on USB-C and external connectors
- [ ] ERC clean; netlist exported

**Exit:** reviewed schematic, ERC clean, ready for layout.

---

## Phase 3 — PCB Layout

- [ ] Stackup + controlled impedance (single-ended 50 Ω, LVDS 100 Ω diff)
- [ ] BGA fanout (XC7S25) with via strategy
- [ ] Length-match HyperRAM bus and LVDS pairs; route clocks first
- [ ] Separate analog/clean-rail regions; star/quiet routing for LDO outputs
- [ ] Solid return paths under all high-speed traces; via fences on LVDS
- [ ] Decoupling placement (close to balls)
- [ ] DRC clean; gerbers + JLCPCB assembly files

**Exit:** DRC-clean layout, fab + assembly package ready.

---

## Phase 4 — Fabrication & Assembly

- [ ] Order PCB + assembly (small run)
- [ ] Source critical parts (FPGA, HyperRAM, FT2232H, flash) separately if needed
- [ ] Inspect boards on arrival (BGA reflow, shorts)

**Exit:** populated boards in hand.

---

## Phase 5 — Bring-up (power + JTAG)

Measure before programming anything.

- [ ] Power-on with current limit; verify each rail voltage + sequencing order (scope on test points)
- [ ] Confirm no latch-up / abnormal current draw
- [ ] Connect FT2232H JTAG; Vivado `hw_server` / openocd detects the XC7S25
- [ ] Program a trivial design (LED blink / counter) over JTAG

**Exit:** all rails correct in order; FPGA identified over JTAG; a bitstream runs.

**First measurements:** rail voltages + sequence timing on scope; JTAG TCK integrity; DONE pin goes high after config.

---

## Phase 6 — Peripheral Verification

Bring up each memory and interface and measure it.

- [ ] **QSPI flash:** read/write/erase; measure configuration time; verify multi-image boot
- [ ] **HyperRAM:** implement fabric controller; run read/write pattern test; **measure sustained bandwidth** (target ~200–300 MB/s)
- [ ] **FT2232H FIFO:** loopback throughput test (target ~8–12 MB/s/channel)
- [ ] **UART console:** echo test
- [ ] **SPI control:** talk to a slave / loopback

**Exit:** flash boots, HyperRAM passes pattern test at measured bandwidth, USB FIFO/UART/SPI characterised.

---

## Phase 7 — LVDS Expansion Loopback

- [ ] Build a passive LVDS loopback fixture on the header
- [ ] Transmit PRBS over LVDS pairs, receive and check bit errors
- [ ] Characterise max clean data rate at the chosen speed grade

**Exit:** LVDS pairs pass PRBS at the target interface rate with acceptable BER.

---

## Phase 8 — First DSP / FFT Demo

Prove the board's reason for existing.

- [ ] Generate or ingest a test tone (from HyperRAM or a source module)
- [ ] Implement an FFT engine using the DSP48 slices
- [ ] Stream a spectrum to the PC over the FT2232H FIFO
- [ ] Verify against a known reference (e.g. NumPy FFT)

**Exit:** a real FFT pipeline runs on the board and produces a correct spectrum on the PC.

---

## Beyond this board

The FPGA-25 is the middle layer. Heavier work — full-rate radar capture, serious SDR, Linux networking, camera pipelines — moves to the later **Artix-7 / RK3566** boards. Modules proven here (ADC/DAC/SDR) carry forward over the LVDS expansion interface.
