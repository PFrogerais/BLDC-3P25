# BLDC-3P25 — Hardware

Three-phase brushless ESC for a **3S LiPo (9.0–12.6 V)**, KiCad 9/10 project.

| Function       | Part                                                    |
|----------------|---------------------------------------------------------|
| Gate driver    | TI DRV8302 (HTSSOP-56 DCA), integrated 5 V / 1.5 A buck |
| Power MOSFETs  | 6 × Infineon BSC009NE2LS5 (25 V, 0.9 mΩ, PG-TDSON-8)    |
| MCU            | SpotPear RP2350-Core-A castellated module               |
| Current sense  | 3 × 1 mΩ low-side shunts, 2 measured (phases A and B)   |

> **Supply range:** the DRV8302 needs PVDD ≥ 8 V and the MOSFETs are rated
> 25 V. 3S is the only LiPo configuration that fits both. Do **not** use 2S
> (driver undervoltage) or 5S+ (MOSFET breakdown).

## Files

| File | Content |
|---|---|
| `BLDC-3P25.kicad_pro` | Project |
| `BLDC-3P25.kicad_sch` | Complete schematic (A2, one sheet, 92 parts) |
| `BLDC-3P25.kicad_pcb` | Board — **layout not started** |
| `BLDC-3P25.kicad_sym` | Project symbols (drawn for this project) |
| `BLDC-3P25.pretty/`   | Project footprints: `Infineon_PG-TDSON-8`, `TI_HTSSOP-56-1EP_6.1x14mm_P0.5mm_DCA` |
| `BLDC-3P25_BOM.csv`   | Bill of materials grouped by value |

Every other footprint comes from the standard KiCad libraries.

## Design values

| Block | Choice | Result |
|---|---|---|
| Buck feedback | 53.6 kΩ / 10.2 kΩ, V<sub>REF</sub> 0.8 V | 5.00 V |
| Buck frequency | R<sub>T</sub> 200 kΩ | ≈ 585 kHz |
| Buck inductor | 22 µH, I<sub>sat</sub> > 2.5 A | ripple 0.23 A at 12.6 V |
| Buck soft start | 10 nF | ≈ 4 ms |
| Buck compensation | 43.2 kΩ + 4.7 nF, 12 pF | f<sub>c</sub> ≈ 14.5 kHz with 2 × 22 µF |
| Shunt amplifiers | 1 mΩ, GAIN high (40 V/V), REF = 3V3 | ±41 A full scale, 0 A = 1.65 V |
| VDS overcurrent | OC_ADJ = 0.148 V, M_OC latch | trips at ≈ 165 A cold / 115 A hot — short-circuit only |
| Dead time | R<sub>DTC</sub> 10 kΩ | ≈ 80 ns |
| VBUS sense | 47 kΩ / 10 kΩ | 2.21 V at 12.6 V, full scale 18.8 V |
| TVS | SMBJ15A | standoff 15 V, clamp 24.4 V < 25 V |

The buck compensation uses the TPS54160 method (same core as the DRV8302
buck, see TI SLVSB56). Check it on the bench with a load step.

## RP2350-Core-A pin use

| GPIO | Signal | GPIO | Signal |
|---|---|---|---|
| GP0 / GP1 | INH_A / INL_A (PWM slice 0) | GP11–GP13 | HALL_A / B / C |
| GP2 / GP3 | INH_B / INL_B (PWM slice 1) | GP14 | THROTTLE (servo PWM, 1 kΩ series) |
| GP4 / GP5 | INH_C / INL_C (PWM slice 2) | GP15 | free |
| GP6 | EN_GATE | GP26 (ADC0) | ISENSE_A |
| GP7 | DC_CAL | GP27 (ADC1) | ISENSE_B |
| GP8 | nFAULT | GP28 (ADC2) | VBUS_SENSE |
| GP9 | nOCTW | GP29 (ADC3) | TEMP (NTC 10 k) |
| GP10 | PWRGD | 5V / 3V3 | from buck via SS14 / module LDO output |

Symbol pin numbers follow the module edge counter-clockwise from the
top-left pad (silkscreen side, photo orientation): 1 = GP29, 2 = 3V3,
3 = 5V, 4 = U−, 5 = U+, 6 = GND, 7–12 = GP0–GP5, 13–19 = GP6–GP12,
20–22 = GP13–GP15, 23–25 = GP26–GP28.

## Before layout / ordering

- [ ] **Draw the RP2350-Core-A footprint** from the SpotPear outline drawing
      (not available yet: the footprint field is empty).
- [ ] Run ERC in KiCad and annotate if you add parts.
- [ ] Check the project footprints against the datasheets
      (`Infineon_PG-TDSON-8`, `TI_HTSSOP-56…DCA`).
- [ ] Layout: Kelvin traces SP/SN to each shunt, AGND joined to GND at the
      DRV8302 PowerPAD only, 10 µF + 100 nF next to each half-bridge,
      wide copper pours for VBAT / phases / GND, NTC next to the low-side FETs.

## Datasheets

Local copies in [`../datasheet/`](../datasheet/). Latest versions online:

- [DRV8302](https://www.ti.com/lit/ds/symlink/drv8302.pdf)
- [BSC009NE2LS5](https://www.infineon.com/cms/en/product/power/mosfet/n-channel/bsc009ne2ls5/)
