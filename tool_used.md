# 🛠️ Instrument & Tooling Specification

This document details the specific hardware and software utility suite used to reverse-engineer, interface with, and stress-test the Alcor Micro flash controller.

---

## 1. Primary Software Suite: AlcorMP
The central engine for all low-level communication. Unlike standard formatting tools, AlcorMP bypasses the Windows storage stack to issue direct Opcodes to the controller firmware.

* **Tool Version:** AlcorMP (Ver. 200916.MD)
* **Execution Environment:** Windows 10/11 x64 (Compatibility mode recommended)
* **Configuration File:** `AlcorMP.ini`
    * *Purpose:* Defines the Flash Vendor ID (VID/PID), memory die timings, and ECC (Error Correction Code) thresholds.
* **Key Operating Modes Used:**
    * **Pure Disk Mode:** Essential for stripping away factory-preloaded promotional CD-ROM partitions.
    * **Natural Check:** A low-voltage scan algorithm designed to verify NAND cell integrity without inducing avalanche breakdown in degraded oxide layers.

---

## 2. Hardware Interface Instrumentation
Physical access to the PCB was required to force the controller into Data Firmware Update (DFU) mode, as software-based reset commands were blocked by write-protection registers.

* **DFU Bridge Tool:** Non-conductive tipped precision stainless-steel tweezers.
    * *Application:* Bridging the `Test Points` (TP) on the PCB surface during power-on to ground the controller's ROM-initiation register.
* **Contact Alignment Shim:** Custom 0.5mm non-conductive cardstock.
    * *Application:* Used as a pressure-shim behind the NAND chip-on-board (COB) substrate to ensure full contact between the gold fingers and the host USB port’s spring-loaded pins.
* **VCC Power Delivery:** Standard USB 2.0 host controller (Direct motherboard port preferred to ensure stable 5V current delivery).

---

## 3. Diagnostic & Verification Utilities
Secondary tools used to validate the state of the controller before and after the mass production cycle.

| Tool | Purpose | Status |
| :--- | :--- | :--- |
| **Rufus** | Initial sector mapping and bootable partition verification | FAILED (Lockup) |
| **Windows Disk Management** | Logical volume identification and health check | FAILED (I/O error) |
| **ChipGenius** | Used to extract the native VID/PID and controller binary signature | SUCCESS |
| **PowerShell (Admin)** | Direct sector-level format command attempts | FAILED |



---

## 4. Telemetry Extraction Path
All logs and telemetry data were captured directly through the AlcorMP event listener window and logged into local text files:

1. **`AlcorMP.log`**: Records the exact second-by-second address mapping during the "Natural Check" scan.
2. **`Flash_Status.ini`**: Contains the final post-mortem report, including the incrementing bad block count ($39 \rightarrow 46$).

> **⚠️ WARNING:** Utilizing these mass-production tools requires precise knowledge of your specific flash chip’s toggle-NAND architecture. Loading a configuration profile incompatible with your specific Samsung K9GDGD8U0D die can permanently destroy the controller chip's internal microcode.

---

## 📂 Repository Reference
* **Utility Configuration:** [`assets/alcor_config.ini`](assets/alcor_config.ini)
* **Diagnostic Log Archive:** [`assets/assets.md`](assets/assets.md)
