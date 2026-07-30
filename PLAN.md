# Lyrion FPGA-25 Plan

Phased roadmap from architecture to a working, characterised board. Each phase has an exit criterion and, where relevant, a bring-up measurement, so progress is judged by hardware behaviour rather than by "the schematic looks done."

---

## Phase 0: Architecture and Part Confirmation  ✅ (mostly done)

Lock the architecture and convert every **UNVERIFIED** item (see SPECS.md §6) into a confirmed datasheet fact.

- [x] Top-level architecture (FPGA, flash, HyperRAM, FT2232H, LVDS header)
- [x] Core part selection (XC7S25, S25FL256L, W958D8, FT2232H, RT7273)
- [ ] Confirm XC7S25 resource counts and I/O per candidate package (DS890)
- [ ] **Choose package and speed grade** (CSGA225, CSG325, or FGG484) from real assembly capability
- [ ] Confirm S25FL256L ordering code and supply voltage
- [ ] Confirm W958D8 max clock for the chosen grade
- [ ] Confirm RT7273 specs (current, input range, frequency, package)

**Exit:** every part has a confirmed datasheet line, and the package and speed grade are chosen.

---

## Phase 1: Power Tree and Sequencing

Design the power system before anything else, because Spartan-7 sequencing constrains it.

- [ ] Map all rails: 5 V in to 3.3 V buck to 1.0 V (VCCINT/VCCBRAM), 1.8 V (VCCAUX/VCCADC/HyperRAM), and 3.3 V (VCCO/FT2232H/flash)
- [ ] Design the RT7273 buck (inductor, caps, feedback, layout per datasheet)
- [ ] Select LDOs for clean rails (noise, PSRR, current)
- [ ] Implement sequencing and enable logic to meet the DS890 ramp rules
- [ ] Add decoupling per Spartan-7 PDN guidance and plan the PDN impedance
- [ ] Add test points on every rail

**Exit:** the power tree is simulated and validated on paper, and the sequencing order is documented and within DS890 limits.

---

## Phase 2: Schematic Capture

- [ ] FPGA symbol plus all power, config, JTAG, and MCLR nets
- [ ] QSPI flash (S25FL256L) on the configuration bank
- [ ] HyperRAM (W958D8) x8 HyperBus plus clock and reset
- [ ] FT2232H: USB-C, JTAG (channel A), UART/SPI/FIFO (channel B), EEPROM
- [ ] High-speed LVDS expansion header (data and clock pairs plus control and power)
- [ ] Power tree from Phase 1
- [ ] ESD/TVS on USB-C and external connectors
- [ ] ERC clean and netlist exported

**Exit:** reviewed schematic, ERC clean, ready for layout.

---

## Phase 3: PCB Layout

- [ ] Stackup plus controlled impedance (single-ended 50 Ω, LVDS 100 Ω differential)
- [ ] BGA fanout (XC7S25) with via strategy
- [ ] Length-match the HyperRAM bus and LVDS pairs, and route clocks first
- [ ] Separate analog and clean-rail regions, with star and quiet routing for LDO outputs
- [ ] Solid return paths under all high-speed traces and via fences on LVDS
- [ ] Decoupling placement close to the balls
- [ ] DRC clean, gerbers and JLCPCB assembly files ready

**Exit:** DRC-clean layout, fab and assembly package ready.

---

## Phase 4: Fabrication and Assembly

- [ ] Order PCB and assembly (small run)
- [ ] Source critical parts (FPGA, HyperRAM, FT2232H, flash) separately if needed
- [ ] Inspect boards on arrival (BGA reflow, shorts)

**Exit:** populated boards in hand.

---

## Phase 5: Bring-up (power + JTAG)

Measure before programming anything.

- [ ] Power-on with a current limit and verify each rail voltage and the sequencing order (scope on test points)
- [ ] Confirm no latch-up or abnormal current draw
- [ ] Connect FT2232H JTAG; Vivado `hw_server` or openocd detects the XC7S25
- [ ] Program a trivial design (LED blink or counter) over JTAG

**Exit:** all rails correct in order, the FPGA is identified over JTAG, and a bitstream runs.

**First measurements:** rail voltages and sequence timing on the scope, JTAG TCK integrity, and the DONE pin going high after config.

---

## Phase 6: Peripheral Verification

Bring up each memory and interface and measure it.

- [ ] **QSPI flash:** read, write, and erase; measure configuration time; verify multi-image boot
- [ ] **HyperRAM:** implement the fabric controller, run a read/write pattern test, and **measure sustained bandwidth** (target about 200 to 300 MB/s)
- [ ] **FT2232H FIFO:** loopback throughput test (target about 8 to 12 MB/s per channel)
- [ ] **UART console:** echo test
- [ ] **SPI control:** talk to a slave or loopback

**Exit:** flash boots, HyperRAM passes the pattern test at the measured bandwidth, and the USB FIFO, UART, and SPI are characterised.

---

## Phase 7: LVDS Expansion Loopback

- [ ] Build a passive LVDS loopback fixture on the header
- [ ] Transmit PRBS over the LVDS pairs, receive, and check for bit errors
- [ ] Characterise the maximum clean data rate at the chosen speed grade

**Exit:** the LVDS pairs pass PRBS at the target interface rate with an acceptable BER.

---

## Phase 8: First DSP / FFT Demo

Prove the reason the board exists.

- [ ] Generate or ingest a test tone (from HyperRAM or a source module)
- [ ] Implement an FFT engine using the DSP48 slices
- [ ] Stream a spectrum to the PC over the FT2232H FIFO
- [ ] Verify against a known reference (for example a NumPy FFT)

**Exit:** a real FFT pipeline runs on the board and produces a correct spectrum on the PC.

---

## Beyond this board

The FPGA-25 is the middle layer. Heavier work such as full-rate radar capture, serious SDR, Linux networking, and camera pipelines moves to the later **Artix-7 and RK3566** boards. Modules proven here (ADC, DAC, SDR) carry forward over the LVDS expansion interface.
