# BLDC-3P25 — Hardware

Three-phase brushless ESC for a **3S LiPo (9.0–12.6 V)**, designed for
**30 A continuous** motor current. KiCad 9/10 project, schematic rev B.

| Function       | Part                                                    |
|----------------|---------------------------------------------------------|
| Gate driver    | TI DRV8302 (HTSSOP-56 DCA), integrated 5 V / 1.5 A buck |
| Power MOSFETs  | 6 × Infineon BSC009NE2LS5 (25 V, 0.9 mΩ, PG-TDSON-8)    |
| MCU            | SpotPear RP2350-Core-A castellated module + USB-C port on the ESC |
| Current sense  | 3 × 1 mΩ low-side shunts, 2 measured (phases A and B)   |
| Rotor position | Hall sensors **or** sensorless BEMF (LM339, solder jumpers) |

> **Supply range:** the DRV8302 needs PVDD ≥ 8 V and the MOSFETs are rated
> 25 V. 3S is the only LiPo configuration that fits both. Do **not** use 2S
> (driver undervoltage) or 5S+ (MOSFET breakdown).

## Files

| File | Content |
|---|---|
| `BLDC-3P25.kicad_pro` | Project |
| `BLDC-3P25.kicad_sch` | Complete schematic (A1, one sheet, 148 parts + 4 solder jumpers) |
| `BLDC-3P25.kicad_pcb` | Board — **layout not started** |
| `BLDC-3P25.kicad_sym` | Project symbols (drawn for this project) |
| `BLDC-3P25.pretty/`   | Project footprints: `Infineon_PG-TDSON-8`, `TI_HTSSOP-56-1EP_6.1x14mm_P0.5mm_DCA`, `SpotPear_RP2350-Core-A_Castellated` (default), `SpotPear_RP2350-Core-A_THT` (pin headers) |
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
| Bulk capacitors | 4 × 330 µF / 25 V polymer (≥ 4 A rms each) + 6 × 10 µF X7R | input ripple ≈ I/2 = 15 A rms at 30 A |
| ADC protection | 1 kΩ + 1 nF + BAT54S clamp on ISENSE_A/B | SO outputs can exceed 3.3 V; fc ≈ 160 kHz |
| BEMF dividers | 10 kΩ / 3.3 kΩ, 4.7 nF, virtual neutral 3 × 10 kΩ | 12.6 V → 3.13 V, fc ≈ 17 kHz |
| Throttle input | 1 kΩ series + 10 kΩ pull-down | unplugged cable reads 0 = motor stopped |
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
| GP10 | PWRGD | 5V / 3V3 | buck **or** USB VBUS via two SS14 / module LDO output |
| U+ / U− | USB D+ / D− to J7 (USB-C) | | |

### Module footprint

From the SpotPear dimension drawing: module 17.78 × 20.32 mm, all pads on a
2.54 mm grid, 1.27 mm from the edges. Top view (components up):

```
          GND  U+  U-  5V  3V3  GP29
   (LED)  [6] [5] [4] [3] [2]  [1]
GP0  [7]                       [25] GP28
GP1  [8]                       [24] GP27
GP2  [9]                       [23] GP26
GP3  [10]                      [22] GP15
GP4  [11]                      [21] GP14
GP5  [12]                      [20] GP13
     [13] [14] [15] [16] [17] [18] [19]
     GP6  GP7  GP8  GP9  GP10 GP11 GP12
```

- `SpotPear_RP2350-Core-A_Castellated` (assigned): SMD pads reaching 0.9 mm
  outside the module edge for the solder fillet. Keep the area under the
  module free of exposed copper and vias.
- `SpotPear_RP2350-Core-A_THT`: 1.0 mm holes for pin headers. Use it with
  female sockets to make the module removable (handy for students).

**The module has no USB connector**: J7 (USB-C on the ESC) is the
programming port. Press BOOT on the module while plugging USB to enter the
UF2 bootloader. R60/R61 are 0 Ω; fit 27 Ω only if the module has no USB
series resistors of its own.

## Solder jumpers (open by default)

| Jumper | Close when… |
|---|---|
| JP1 `BEC 5V` | the RC receiver on J6 must be powered by the ESC. Leave open with a self-powered source (Arduino on USB, servo tester on battery). |
| JP2–JP4 `SENSORLESS A/B/C` | the motor has **no** Hall sensors. Unplug J5 first: comparator and Hall outputs share the same lines. |

## Firmware rules imposed by the hardware

- Keep GP0–GP6 low until **PWRGD** is high (the 10 kΩ pull-downs hold the
  driver inputs low during reset and when the board runs from USB only).
- Stop regenerative braking when VBUS_SENSE exceeds **14 V**: the TVS
  clamps at 24.4 V, the MOSFETs are rated 25 V.
- Limit motor current from ISENSE_A/B (the VDS protection only catches
  hard short circuits).
- Sensorless mode: blank the comparator outputs for a few µs after each PWM
  edge.

## Wiring

Fit an **inline 40 A blade fuse** on the battery lead: a 30 A board fuse
is bulky, and the fuse protects the wiring as well as the board.

## Before layout / ordering

- [ ] Run ERC in KiCad and annotate if you add parts.
- [ ] Print the layout 1:1 and lay the real parts on it, especially the
      project footprints (MOSFET, DRV8302, RP2350-Core-A).
- [ ] Layout: Kelvin traces SP/SN to each shunt, AGND joined to GND at the
      DRV8302 PowerPAD only, 10 µF + 100 nF next to each half-bridge,
      wide copper pours for VBAT / phases / GND, NTC next to the low-side FETs.

## Datasheets

Local copies in [`../datasheet/`](../datasheet/). Latest versions online:

- [DRV8302](https://www.ti.com/lit/ds/symlink/drv8302.pdf)
- [BSC009NE2LS5](https://www.infineon.com/cms/en/product/power/mosfet/n-channel/bsc009ne2ls5/)
