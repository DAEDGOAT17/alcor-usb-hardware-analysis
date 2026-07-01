# Repository Assets & Hardware Telemetry Log

This file archives the media assets, hardware pinouts, and low-level diagnostic logs captured during the Alcor Micro flash controller reconstruction project.

---

## 1. Physical Hardware Layout Reference

Before running the software utility, the flash memory drive's external housing was extracted to expose the raw physical logic layers. 

### Chip-On-Board (COB) / UDP Form Factor Pinout
The drive utilizes an ultra-thin monolithic UDP architecture where the flash memory wafer and controller are integrated into a single sealed substrate. 

```text
       +-----------------------+
       |   [ [ [ [ USB ] ] ] ] |  <-- Standard USB Type-A Interface Contacts
       +-----------------------+
       |                       |
       |  +---+                |
       |  |   | NAND Flash     |
       |  +---+ Wafer Area     |
       |                       |
       |    [ Pad A ]          |  <-- Hardware Test Points
       |    [ Pad B ]          |  <-- Shorting paths forces DFU / Safe Mode
       |                       |
       +-----------------------+
