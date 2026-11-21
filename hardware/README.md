# BLDC-3P25 — Hardware

Three-phase brushless ESC, KiCad 10 project.

| Function       | Part                                  |
|----------------|---------------------------------------|
| Gate driver    | TI DRV8302 (DCA package)              |
| Power MOSFETs  | Infineon BSC009NE2LS5 (25 V, 0.9 mΩ)  |

> **Voltage limit:** the MOSFETs are rated 25 V V<sub>DS</sub>. Keep the bus at
> 4S LiPo (16.8 V) or below to leave margin for switching overshoot.

## Files

- `BLDC-3P25.kicad_pro` — project
- `BLDC-3P25.kicad_sch` — schematic
- `BLDC-3P25.kicad_pcb` — board (not started yet)
- `untitled.kicad_sch` — draft sheet, not yet linked to the main schematic

## Datasheets

- [DRV8302](https://www.ti.com/lit/ds/symlink/drv8302.pdf)
- [BSC009NE2LS5](https://www.infineon.com/cms/en/product/power/mosfet/n-channel/bsc009ne2ls5/)

## Libraries

The schematic embeds its symbols, so it opens without extra files.
The project-local `Driver.pretty` footprint library and `driver.kicad_sym`
are not committed: they come from SamacSys, whose licence forbids
redistributing the models as a library. Download them again from
[Component Search Engine](https://componentsearchengine.com/) and place them
next to the project file.
