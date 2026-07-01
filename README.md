# AlcorMP USB Flash Controller Reconstruction — Post-Mortem Analysis

This repository documents the low-level hardware diagnostics, custom profiling configurations, and eventual logical failure analysis of a hardware-locked, promotional Alcor Micro USB flash drive. 

Standard operating system tools and third-party high-level formatters completely locked up or failed to detect the flash volume, necessitating bare-metal hardware access and custom mass production tooling manipulation.

## Technical Metadata & Specifications
* **Controller Manufacturer:** Alcor Micro Corp.
* **Target Controller Series:** AU6989SN-GTC / GTD / GTE / TA (Detected via binary signatures)
* **Flash Memory Die:** Samsung K9GDGD8U0D (Toggle MLC wafer configuration)
* **Initial Geometry Mapping:** 1,348 Total Addressable Allocation Blocks
* **Baseline Telemetry:** 39 Confirmed Hardware-Isolated Bad Blocks
* **Post-Stress Test Telemetry:** 46 Confirmed Dead Blocks (7 additional cell failures)
* **Terminal Status:** Physical Structural Degradation (Post-Mortem)

---

## 1. Initial Symptoms & Standard Software Isolation Failures
The drive initially presented an immutable, factory-level write-protection lock. Attempts to overwrite Sector 0 via commercial logical layout interfaces resulted in instant software-level execution lockups:

* **High-Level Format Interventions:** Partitioning suites like `Rufus` completely lost responsiveness (`Not Responding` state) when trying to establish clear master boot layers.
* **Kernel Command Line Failures:** Direct force-formatting sequences initiated within an administrative `Windows PowerShell 5.1` instance via standard commands (`format G: /fs:FAT32 /q /x`) repeatedly dropped execution loops, reporting `Specified drive does not exist` or forcing an infinite `Insert new disk` instruction loop.

Because the logical operating system layer was blocked by internal controller flags, physical hardware shorting was utilized to bypass standard security registers.

---

## 2. Hardware Test Point Interface & DFU Initialization
To strip the proprietary vendor firmware loop, the external enclosure shell was removed to expose the monolithic integrated substrate layer. 

1. **DFU Bootloader Safe Mode Trigger:** A hard connection bridge was established across the specialized onboard hardware test pads using precision tweezers during the module's initial `VCC 5V` insertion. This bypassed the corrupt initialization sector and exposed a clean microcontroller state.
2. **Contact Pressure Alignment Shim:** Due to structural deficits inherent to exposed flash chips, a non-conductive `0.5mm physical paper shim` was introduced behind the primary interface pins to maintain adequate physical contact geometry inside the host port.

---

## 3. Low-Level Mass Production Environment Setup
With the controller locked into an open programming environment, the factory utility suite `AlcorMP` was configured with highly conservative parameters to prevent early sector dropout:

* **Mode:** `Pure Disk`
* **MP Mode:** `Capacity Optimize`
* **Scan Mode:** `Low Level Format -- Natural Check`
* **Scan Level:** `Full Scan2`
* **ECC Bit Correction:** `12`
* **Bad Block Management:** `Auto Check`

### Strategic Parameter Rationales:
* **Natural Check Scan Mode:** Instructs the controller engine to pace individual write-verify voltages over the silicon arrays to prevent high-voltage avalanche breakdowns in unstable sectors.
* **ECC = 12:** Grants a high 12-bit Error Correction capacity per page block, allowing the compiler to retain control of slightly degraded cells instead of declaring immediate execution faults.

---

## 4. Analytical Chronology & Failure Progression

The structural analysis of the execution lifecycle is preserved step-by-step using the assets tracked inside the repository directory structure:

### Phase A: Environment Mapping
* **Source Asset:** `assets/Screenshot 2026-07-01 164231.png`
* **Telemetry State:** The controller parses baseline properties, validating a foundational layout of **`Bad Block: 39/1348`**. The device registers under default hardware address layers.

### Phase B: Continuous Sector Scan (100.0% Mark)
* **Source Asset:** `assets/Screenshot 2026-07-01 174425.png`
* **Telemetry State:** The raw sector verification sequence runs for an exhaustive execution timeline of `00:30:13`. The algorithm sweeps through 100.0% of the addresses, successfully logging unstable logical boundaries.

### Phase C: Structural File System Compilation Failure
* **Source Asset:** `assets/Screenshot 2026-07-01 174425.png` (Terminal Segment)
* **Telemetry State:** The compiler attempts to build the final high-level allocation infrastructure (FAT32/exFAT allocation chains) on Sector 0/1. The operation fails catastrophically, throwing an explicit red terminal code:

```text
91F00: Create file system error
Bad Block: 46/1348
