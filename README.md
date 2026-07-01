# Alcor MP USB Flash Controller - Hardware Post-Mortem Analysis

This repository documents the low-level hardware troubleshooting and structural failure analysis of a write-protected, promotional Alcor Micro USB flash drive. 

## Technical Overview
* **Controller:** Alcor Micro (AU6989 series or similar)
* **Goal:** Bypassing factory write-protection to re-purpose the drive for a bootable OS environment.
* **Result:** Hardware-level cascading NAND cell failure (Post-Mortem).

## Troubleshooting & Methodology
1. **Hardware DFU Mode:** The ultra-thin COB (Chip-on-Board) module enclosure was stripped. Two gold hardware test points on the PCB were physically shorted to force the controller into Data Firmware Update (DFU) / Safe Mode, bypassing standard OS write-protection blocks.
2. **Physical Modification:** Implemented a custom card/paper shim/spacer to ensure physical alignment between the thin PCB contact pads and the host laptop's internal USB port pins.
3. **Mass Production Tooling:** Utilized `AlcorMP.exe` to execute a low-level format (Natural Check, ECC = 12, Pure Disk mode) to rewrite Sector 0 firmware and isolate known bad blocks.

## Diagnostics & Failure Analysis
During the deep low-level format stress test, the drive successfully reached **100.0%** completion but threw a final execution error:
* **Error Code:** `91F00: Create file system error`
* **NAND Degradation:** The bad block count cascaded from an initial `39` to `46/1348` physical blocks under voltage stress.

### Conclusion
The analysis proved that the promotional drive utilized low-grade, downgraded silicon wafers. While completely functional for standard low-voltage reading, the silicon cells at Sector 0/1 permanently degraded and collapsed under the high-voltage write sequences required to compile a fresh Master Boot Record (MBR) and partition scheme.
