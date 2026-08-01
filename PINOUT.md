# Lyrion FPGA-25 — Pin Assignment

**Source:** Xilinx ASCII Pinout File, `xc7s25csga225`, Rev 1.1, Production, 2017-12-05
**Device:** XC7S25-2CSGA225C (primary) / XC7S25-1CSGA225C (fallback)
**Package:** CSGA225, 225 balls, 13 × 13 mm, 0.8 mm pitch
**Total pins:** 225 = 150 user I/O + 21 config + 54 power/GND

> All bank 15 pins marked "7S15" in the source file are **available on XC7S25**.
> That marking means they are no-connect only on the smaller XC7S15 die.

---

## 1. Bank Summary

| Bank | VCCO | I/O Pins | Primary Use | I/O Standards |
|------|------|----------|-------------|---------------|
| 0 | 3.3 V (config) | 21 config | JTAG, QSPI mode, config control | CONFIG (fixed) |
| 14 | 3.3 V | 50 | QSPI flash data + 3.3 V GPIO | LVCMOS33 only |
| 15 | 1.8 V | 50 | HyperRAM + LVDS expansion + ADC | LVCMOS18, LVDS_18 |
| 34 | 3.3 V | 50 | FT2232H interface + general I/O | LVCMOS33 only |

**User I/O total: 150** (confirmed from pinout Rev 1.1; 50 per bank).

> **LVDS constraint:** VCCO_14 and VCCO_34 are 3.3 V, so neither bank supports LVDS
> (LVDS requires VCCO ≤ 2.5 V). **All LVDS expansion pairs must come from bank 15**
> (VCCO_15 = 1.8 V, LVDS_18). This is a hard constraint on the expansion header design.

---

## 2. Bank 0 — Configuration (21 pins, fixed by silicon)

| Ball | Pin Name | Net | Connection | Notes |
|------|----------|-----|------------|-------|
| E5 | TCK_0 | jtag_tck | FT2232H ADBUS0 | JTAG clock |
| L4 | TDI_0 | jtag_tdi | FT2232H ADBUS1 | JTAG data in |
| K4 | TDO_0 | jtag_tdo | FT2232H ADBUS2 | JTAG data out |
| K5 | TMS_0 | jtag_tms | FT2232H ADBUS3 | JTAG mode select |
| F5 | CCLK_0 | qspi_sck | S25FL256L SCK | Config clock / QSPI clock |
| M5 | M0_0 | cfg_m0 | Pull-up to 3.3 V, 4.7 kΩ | Mode bit 0 = 1 |
| N5 | M1_0 | cfg_m1 | Pull-down to GND, 4.7 kΩ | Mode bit 1 = 0 |
| M6 | M2_0 | cfg_m2 | Pull-down to GND, 4.7 kΩ | Mode bit 2 = 0 |
| G5 | DONE_0 | cfg_done | LED + 4.7 kΩ pull-up to VCCO_0 | Config done indicator |
| L5 | INIT_B_0 | cfg_init_b | 4.7 kΩ pull-up to VCCO_0 | Config error flag |
| J5 | PROGRAM_B_0 | cfg_program_b | Tact button to GND + 4.7 kΩ pull-up | Active-low config reset |
| H5 | CFGBVS_0 | cfg_cfgbvs | Tie to VCCO_0 (3.3 V) via 0 Ω | Selects 3.3 V config I/O levels |
| D5 | VCCBATT_0 | 3v3 | Tie to 3.3 V | Battery-backed key storage |
| F9 | VCCADC_0 | 1v8 | 1.8 V rail, 100 nF + 1 µF | XADC analog supply |
| F8 | GNDADC_0 | GND | Analog ground plane | XADC analog ground |
| H9 | VREFP_0 | vrefp | 100 nF to GND (internal ref mode) | XADC positive reference |
| G8 | VREFN_0 | GND | Analog ground | XADC negative reference |
| G9 | VP_0 | xadc_vp | 10 kΩ from 1v8 + 10 kΩ to GND (0.9 V) | On-chip VCCADC monitor |
| H8 | VN_0 | GND | Analog ground | XADC negative analog input |
| J9 | DXP_0 | NC | No connect | External temp diode + (not used) |
| J8 | DXN_0 | NC | No connect | External temp diode − (not used) |

**Mode pins:** M[2:0] = 001 → Quad SPI (x4) master.
**UNVERIFIED:** confirm mode encoding against UG470 Table 5-3.

**XADC note:** internal reference mode selected. VREFP gets a 100 nF cap to GND;
the internal 1.25 V reference drives the pin. VP is biased to 0.9 V for VCCADC
monitoring. DXP/DXN are for an external temperature diode and are left NC.

---

## 3. Bank 14 — QSPI Flash + 3.3 V GPIO (VCCO_14 = 3.3 V)

### 3.1 Assigned: QSPI flash (S25FL256L)

| Ball | Pin Name | Net | Connection |
|------|----------|-----|------------|
| H14 | IO_L1P_T0_D00_MOSI_14 | qspi_io0 | S25FL256L IO0 (SI / MOSI) |
| H15 | IO_L1N_T0_D01_DIN_14 | qspi_io1 | S25FL256L IO1 (SO / MISO) |
| J12 | IO_L2P_T0_D02_14 | qspi_io2 | S25FL256L IO2 (WP#) |
| K13 | IO_L2N_T0_D03_14 | qspi_io3 | S25FL256L IO3 (HOLD# / RESET#) |
| L11 | IO_L6P_T0_FCS_B_14 | qspi_cs_n | S25FL256L CS# |

CCLK (qspi_sck) is on bank 0, ball F5. Six signals total for QSPI x4.

### 3.2 Available: 3.3 V single-ended GPIO (44 pins, no LVDS)

| Byte Group | Balls | Pins | Notes |
|------------|-------|------|-------|
| 0 (remaining) | K11, K12, K14, L15, J15, K15, L12 | 7 | K11 = PUDC_B (internal pull-up during config) |
| 1 | L13, L14, M13, N13, M14, M15, N14, N15, P14, P15, R13, R14 | 12 | P14/P15 = SRCC, R13/R14 = MRCC |
| 2 | M9, M10, N12, P12, N10, N11, P10, P11, R9, R10, R11, R12 | 12 | N10 = RDWR_B, N11 = DOUT/CSO_B, P10 = CSI_B (all GPIO in QSPI mode) |
| 3 | M7, M8, N6, N7, N8, N9, R7, R8, P6, P7, R5, R6 | 12 | |
| Singles | J11, L10 | 2 | IO_0_14, IO_25_14 |

All 44 pins are LVCMOS33 GPIO. **No LVDS on this bank** (VCCO = 3.3 V).

---

## 4. Bank 15 — HyperRAM + LVDS Expansion (VCCO_15 = 1.8 V)

### 4.1 Assigned: HyperRAM (W958D8, x8 HyperBus)

| Ball | Pin Name | Net | Connection |
|------|----------|-----|------------|
| C6 | IO_L1P_T0_AD0P_15 | hyper_dq[0] | W958D8 DQ0 |
| C7 | IO_L1N_T0_AD0N_15 | hyper_dq[1] | W958D8 DQ1 |
| D7 | IO_L2P_T0_AD8P_15 | hyper_dq[2] | W958D8 DQ2 |
| D8 | IO_L2N_T0_AD8N_15 | hyper_dq[3] | W958D8 DQ3 |
| B6 | IO_L3P_T0_DQS_AD1P_15 | hyper_dq[4] | W958D8 DQ4 |
| A7 | IO_L3N_T0_DQS_AD1N_15 | hyper_dq[5] | W958D8 DQ5 |
| C8 | IO_L4P_T0_AD9P_15 | hyper_dq[6] | W958D8 DQ6 |
| C9 | IO_L4N_T0_AD9N_15 | hyper_dq[7] | W958D8 DQ7 |
| A5 | IO_L5P_T0_AD2P_15 | hyper_rwds | W958D8 RWDS (read/write data strobe) |
| A8 | IO_L6P_T0_15 | hyper_cs_n | W958D8 CS# |
| A13 | IO_L12P_T1_MRCC_AD5P_15 | hyper_ck_p | W958D8 CK (MRCC clock pair) |
| A14 | IO_L12N_T1_MRCC_AD5N_15 | hyper_ck_n | W958D8 CK# |
| D11 | IO_0_15 | hyper_rst_n | W958D8 RST# |

13 pins used. CK/CK# on the MRCC pair for lowest clock jitter.
DQ[7:0] + RWDS + CS# occupy byte group 0 (10 of 12 pins).

### 4.2 Available from byte group 0 (2 pins)

| Ball | Pin Name | Net | Notes |
|------|----------|-----|-------|
| A6 | IO_L5N_T0_AD2N_15 | gpio_15_a6 | Available 1.8 V GPIO |
| A9 | IO_L6N_T0_VREF_15 | gpio_15_a9 | Available 1.8 V GPIO (VREF pin) |

### 4.3 Expansion: ADC / analog module interface (byte group 1, 10 pins)

Byte group 1 has AD-capable differential pairs. Reserved for future ADC module
interfaces over the expansion header. The MRCC pair (A13/A14, AD5) is consumed
by the HyperRAM clock (§4.1), leaving 5 AD pairs (10 pins) available.

| Ball | Pin Name | Pair | Notes |
|------|----------|------|-------|
| D9, D10 | IO_L7P/N_T1_AD10P/N_15 | AD10 | LVDS_18 or analog |
| C10, B10 | IO_L8P/N_T1_AD3P/N_15 | AD3 | LVDS_18 or analog |
| B9, A10 | IO_L9P/N_T1_DQS_AD11P/N_15 | AD11 (DQS) | LVDS_18 or analog |
| B11, B12 | IO_L10P/N_T1_AD4P/N_15 | AD4 | LVDS_18 or analog |
| A11, A12 | IO_L11P/N_T1_SRCC_AD12P/N_15 | AD12 (SRCC) | LVDS_18 or analog |

### 4.4 Expansion: LVDS data (byte group 2, 12 pins / 6 pairs)

| Ball | Pin Name | Pair | Notes |
|------|----------|------|-------|
| B13, B14 | IO_L13P/N_T2_MRCC_15 | LVDS pair 1 | MRCC clock-capable |
| C15, B15 | IO_L14P/N_T2_SRCC_15 | LVDS pair 2 | SRCC clock-capable |
| C13, C14 | IO_L15P/N_T2_DQS_15 | LVDS pair 3 | DQS strobe |
| D12, D13 | IO_L16P/N_T2_15 | LVDS pair 4 | |
| F11, E11 | IO_L17P/N_T2_15 | LVDS pair 5 | |
| E12, E13 | IO_L18P/N_T2_15 | LVDS pair 6 | |

### 4.5 Expansion: LVDS data + clock + control (byte group 3, 12 pins / 6 pairs)

| Ball | Pin Name | Pair | Notes |
|------|----------|------|-------|
| E14, D15 | IO_L19P/N_T3_VREF_15 | LVDS pair 7 | VREF pin on N side |
| G15, F15 | IO_L20P/N_T3_15 | LVDS pair 8 | |
| F14, E15 | IO_L21P/N_T3_DQS_15 | LVDS pair 9 | DQS strobe |
| G13, G14 | IO_L22P/N_T3_15 | LVDS pair 10 | |
| G11, G12 | IO_L23P/N_T3_15 | LVDS pair 11 | |
| H12, H13 | IO_L24P/N_T3_RS1/RS0_15 | LVDS pair 12 | RS1/RS0 are regular I/O on Spartan-7 (**UNVERIFIED**) |

### 4.6 Expansion single

| Ball | Pin Name | Net | Notes |
|------|----------|-----|-------|
| H11 | IO_25_15 | gpio_15_h11 | Single-ended 1.8 V GPIO |

### 4.7 Bank 15 summary

| Use | Pins | LVDS Pairs |
|-----|------|------------|
| HyperRAM (BG0 + MRCC + IO_0) | 13 | 0 |
| ADC / analog (BG1, 5 AD pairs) | 10 | 5 |
| LVDS expansion (BG2) | 12 | 6 |
| LVDS expansion (BG3) | 12 | 6 |
| GPIO singles + BG0 leftover | 3 | 0 |
| **Total** | **50** | **17** |

---

## 5. Bank 34 — FT2232H + General I/O (VCCO_34 = 3.3 V)

### 5.1 Assigned: FT2232H channel B (byte group 0 + 1 pin, 13 pins)

Supports 245-FIFO, MPSSE (SPI/I2C), and UART modes on the same pins.

| Ball | Pin Name | Net | FT2232H Pin | Function |
|------|----------|-----|-------------|----------|
| A4 | IO_L1P_T0_34 | usb_dbus0 | BDBUS0 | SCK / FIFO_CLK |
| A3 | IO_L1N_T0_34 | usb_dbus1 | BDBUS1 | MOSI / TXD / FIFO_D0 |
| B2 | IO_L2P_T0_34 | usb_dbus2 | BDBUS2 | MISO / RXD / FIFO_D1 |
| A2 | IO_L2N_T0_34 | usb_dbus3 | BDBUS3 | CS# / FIFO_D2 |
| B4 | IO_L3P_T0_DQS_34 | usb_dbus4 | BDBUS4 | GPIO / FIFO_D3 |
| B3 | IO_L3N_T0_DQS_34 | usb_dbus5 | BDBUS5 | GPIO / FIFO_D4 |
| C1 | IO_L4P_T0_34 | usb_dbus6 | BDBUS6 | GPIO / FIFO_D5 |
| B1 | IO_L4N_T0_34 | usb_dbus7 | BDBUS7 | GPIO / FIFO_D6 |
| C5 | IO_L5P_T0_34 | usb_rx_n | BCBUS0 | RXF# (FIFO) |
| B5 | IO_L5N_T0_34 | usb_tx_n | BCBUS1 | TXE# (FIFO) |
| D2 | IO_L6P_T0_34 | usb_rd_n | BCBUS2 | RD# (FIFO) |
| D1 | IO_L6N_T0_VREF_34 | usb_wr_n | BCBUS3 | WR# (FIFO) |
| D4 | IO_L7P_T1_34 | usb_oe_n | BCBUS4 | OE# (FIFO) |

In MPSSE mode (SPI/UART), only BDBUS0–3 are used; BDBUS4–7 and BCBUS0–4 are GPIO.
In 245-FIFO mode, all 13 pins are active.

### 5.2 Assigned: LEDs, buttons, I2C (byte group 1, 6 pins)

| Ball | Pin Name | Net | Connection |
|------|----------|-----|------------|
| E2 | IO_L8P_T1_34 | led[0] | LED + 330 Ω to GND |
| E1 | IO_L8N_T1_34 | led[1] | LED + 330 Ω to GND |
| E3 | IO_L9P_T1_DQS_34 | btn[0] | Tact button to GND + 10 kΩ pull-up |
| D3 | IO_L9N_T1_DQS_34 | btn[1] | Tact button to GND + 10 kΩ pull-up |
| F2 | IO_L10P_T1_34 | i2c_scl | 4.7 kΩ pull-up to 3.3 V |
| F1 | IO_L10N_T1_34 | i2c_sda | 4.7 kΩ pull-up to 3.3 V |

### 5.3 Available: 3.3 V GPIO (31 pins, no LVDS)

| Byte Group | Balls | Pins | Notes |
|------------|-------|------|-------|
| 1 (remaining) | C4, F4, F3, H1, G1 | 5 | C4 = IO_L7N, F4/F3 = SRCC, H1/G1 = MRCC |
| 2 | H4, H3, J2, H2, J4, J3, K1, J1, K3, K2, M1, L1 | 12 | H4/H3 = MRCC, J2/H2 = SRCC, J4/J3 = DQS |
| 3 | M4, M3, N2, M2, P3, N3, P1, N1, R4, R3, R2, P2 | 12 | M4/M3 = VREF, P3/N3 = DQS |
| Singles | G4, N4 | 2 | IO_0_34, IO_25_34 |

All 31 pins are LVCMOS33 GPIO. **No LVDS on this bank** (VCCO = 3.3 V).

---

## 6. Power Pins (54 pins)

### 6.1 Core and auxiliary supplies

| Rail | Pins | Balls | Decoupling (per pin) | Bulk |
|------|------|-------|----------------------|------|
| VCCINT (1.0 V) | 8 | E8, E10, F7, G10, H7, J10, K7, K9 | 100 nF MLCC | 2 × 10 µF |
| VCCBRAM (1.0 V) | 2 | G6, J6 | 100 nF MLCC | shared with VCCINT bulk |
| VCCAUX (1.8 V) | 2 | L6, L8 | 100 nF MLCC | 1 × 10 µF |
| VCCADC (1.8 V) | 1 | F9 | 100 nF + 1 µF | — |

### 6.2 I/O bank supplies

| Rail | Pins | Balls | Decoupling (per pin) | Bulk |
|------|------|-------|----------------------|------|
| VCCO_0 (3.3 V) | 2 | E6, P5 | 100 nF MLCC | 1 × 10 µF |
| VCCO_14 (3.3 V) | 3 | J14, M12, P9 | 100 nF MLCC | 1 × 10 µF |
| VCCO_15 (1.8 V) | 3 | B7, C11, F12 | 100 nF MLCC | 1 × 10 µF |
| VCCO_34 (3.3 V) | 3 | C3, G2, L3 | 100 nF MLCC | 1 × 10 µF |

### 6.3 Ground (31 pins)

A1, A15, B8, C2, C12, D6, D14, E4, E7, E9, F6, F10, F13, G3, G7, H6, H10,
J7, J13, K6, K8, K10, L2, L7, L9, M11, P4, P8, P13, R1, R15

Every GND ball connects to the ground plane with a dedicated via. No daisy-chaining.

---

## 7. Net Naming Convention

| Prefix | Domain | Example |
|--------|--------|---------|
| `qspi_` | QSPI flash bus | `qspi_io0`, `qspi_cs_n`, `qspi_sck` |
| `hyper_` | HyperRAM bus | `hyper_dq[0]`, `hyper_ck_p`, `hyper_cs_n` |
| `jtag_` | JTAG | `jtag_tck`, `jtag_tdo` |
| `cfg_` | Config control | `cfg_done`, `cfg_m0`, `cfg_program_b` |
| `usb_` | FT2232H channel B | `usb_dbus0`, `usb_rx_n` |
| `lvds_` | Expansion LVDS | `lvds_d0_p`, `lvds_clk_n` |
| `led_`, `btn_` | Board I/O | `led[0]`, `btn[1]` |
| `i2c_` | I2C bus | `i2c_scl`, `i2c_sda` |
| `gpio_` | General purpose | `gpio_15_a6` |
| (none) | Power rails | `3v3`, `1v8`, `1v0`, `5v0` |

Active-low: `_n` suffix. Differential: `_p` / `_n` suffix (Altium auto-pairs).

---

## 8. Design Notes

1. **CFGBVS = 1** (tied to VCCO_0 = 3.3 V): configuration pins operate at 3.3 V levels,
   matching the S25FL256L QSPI flash. If the flash were 1.8 V, CFGBVS would be 0.

2. **Mode pins M[2:0] = 001**: QSPI x4 master boot. M0 pulled up, M1 and M2 pulled down.
   After configuration, these pins become bank 14 GPIO (D00, D01, D02 functions).

3. **Bank 15 "7S15" pins**: all 50 bank 15 I/O are bonded on XC7S25. The "7S15"
   no-connect marking applies only to the smaller XC7S15 die. Every bank 15 pin
   must be connected or explicitly marked NC in the schematic.

4. **LVDS is bank-15-only**: VCCO_14 = 3.3 V and VCCO_34 = 3.3 V preclude LVDS on
   those banks. The expansion header LVDS pairs are LVDS_18 (1.8 V) from bank 15
   byte groups 2 and 3 (12 pairs). Byte group 1 AD pairs are reserved for analog
   module interfaces.

5. **HyperRAM clock on MRCC**: CK/CK# use the IO_L12P/N MRCC pair (A13/A14) for
   access to a global clock buffer (BUFG), minimising clock jitter on the HyperBus.

6. **VCCBATT**: tied to 3.3 V for battery-backed AES key storage. If encrypted
   bitstream is never used, this connection is harmless.

7. **XADC**: internal reference mode. VREFP has a 100 nF cap to GND. VP is biased
   to 0.9 V (10 kΩ / 10 kΩ from VCCADC) for on-chip supply monitoring. DXP/DXN
   are NC (no external temperature diode).

8. **RS1/RS0 pins** (H12/H13): named for legacy SPI fallback on larger 7-series
   devices. On Spartan-7 these are regular I/O. **UNVERIFIED** — confirm in DS890.

---

## 9. UNVERIFIED

1. Mode pin encoding M[2:0] = 001 for QSPI x4 master (UG470 Table 5-3).
2. RS1/RS0 pins are regular I/O on Spartan-7 (DS890).
3. VREFP internal-reference connection requirement (DS890 XADC section).
4. VCCBATT tie-to-3.3 V is acceptable when AES key storage is unused (DS890).
5. DXP/DXN safe to leave NC when external temp diode is not fitted (DS890).
