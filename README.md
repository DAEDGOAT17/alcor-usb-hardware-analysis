# 🛠️ AlcorMP USB Controller Rescue — Hardware Hack & Post-Mortem

Ever wondered what happens when you push absolute bargain-bin flash memory to its absolute physical limits? This repository documents a deep-dive hardware investigation where we bypassed factory-level write locks on a promotional Alcor Micro USB drive by going straight to the bare metal.

When standard operating system utilities completely froze or failed to even recognize the drive, we stripped off the casing, shorted the hardware test points, and used factory mass production tools to see what went wrong under the hood. 

---

## 📊 Technical Specs at a Glance
* **Microcontroller:** Alcor Micro Corp. (AU6989SN-GTC / GTD / GTE / TA series)
* **NAND Flash Memory:** Samsung K9GDGD8U0D (Toggle MLC architecture)
* **Factory Tooling:** AlcorMP Application (Version 3.1.1.33)
* **Total Storage Geometry:** 1,348 allocatable blocks
* **Starting Health Telemetry:** 39 factory-isolated bad blocks
* **Final Health Telemetry:** 46 completely dead blocks (7 new cells collapsed under testing)
* **Project Status:** Definite Hardware Failure (An awesome post-mortem engineering lesson!)

---

## 1. The Red Flags: How the Drive Locked Us Out
Before we broke out the hardware tools, the flash drive was completely bricked by an unyielding, factory-level write-protection lock. Trying to force our way through standard Windows software felt like hitting a brick wall:

* **The Rufus Freeze:** High-level partitioning software like `Rufus` went completely unresponsive (`Not Responding`) the second it tried to clear the Master Boot Record.
* **The PowerShell Drop:** Running administrative command-line formatting (`format G: /fs:FAT32 /q /x`) either threw a frustrating `Specified drive does not exist` error or got stuck in an infinite loop asking us to `Insert new disk`.
* **The Windows Loop:** Standard File Explorer kept choking on generic I/O errors, throwing the classic modal pop-up: `Please insert a disk into Removable Disk (H:)`.

Because the drive’s internal controller chip was stuck in a permanent software lock, we realized we had to bypass the operating system layers entirely using hardware.

---

## 2. Going Bare Metal: The Hardware Hack
To force the drive to talk to us, we carefully popped open the plastic housing to expose the tiny, monolithic integrated circuit board layout.

1. **Tweezers to the Rescue (DFU Mode):** We tracked down the specialized hardware test points on the PCB. By bridging these two gold pads with precision tweezers while plugging the drive into a live `5V USB` port, we intercepted the boot loop, bypassed the corrupt internal firmware, and dropped the chip into a raw Data Firmware Update (DFU) program mode.
2. **The Custom Paper Shim:** Because bare chip-on-board promotional drives are structurally thinner than standard metal-shielded USB sticks by about 0.5mm, the contact pins weren't hugging the laptop port tightly enough. We implemented a non-conductive `0.5mm paper business card spacer` underneath the chip to push the pads firmly against the host leaf springs.

---

## 3. Configuring the Factory Mass Production Suite
Once the computer recognized our custom-shorted hardware, we fired up the factory deployment utility `AlcorMP` (Build 3.1.1.33). Because we knew the silicon was incredibly fragile, we hand-configured a highly conservative, low-stress environment profile inside `AlcorMP.ini`:

* **Operation Mode:** `Pure Disk` (Strips away bloated vendor custom partition configurations)
* **MP Target:** `Capacity Optimize`
* **Scan Method:** `Low Level Format -- Natural Check` 
* **Depth Level:** `Full Scan2`
* **Error Capacity Layer (ECC):** Set to `12` (Gives the compiler a generous 12-bit error correction window to keep weak cells alive)
* **Bad Block Tracking:** `Auto Check`

### Why these specific settings?
We switched the tool to **Natural Check**. Instead of blasting the fragile silicon gates with continuous high voltage, this tells the controller to step carefully through the address spaces to prevent a cascading avalanche breakdown.

---

## 4. Timeline of the Diagnostic Run

The entire lifecycle of the test is backed up step-by-step by the screenshots saved right here in the repository’s `/assets` directory:

### Step A: The Initial Map
* **Target File:** `assets/Screenshot 2026-07-01 164231.png`
* **What it shows:** The moment the controller woke up after the hardware short. It successfully read the baseline architecture, initialized under factory address layers, and flagged an initial **`Bad Block: 39/1348`**.

### Step B: The 100% Milestone
* **Target File:** `assets/3ff7d696-eb0d-4cc3-95f9-520ff371bd81`
* **What it shows:** The drive successfully survived an exhaustive raw sector check lasting exactly `30 minutes and 13 seconds`. The low-level scan passed 100% of the drive space, carefully mapping out where the unstable logic gates were hiding.

### Step C: Catastrophic Silicon Failure
* **Target File:** `assets/3fdb066e-fd9f-487e-a6a6-7781222d8104`
* **What it shows:** The grand finale. After completing the scan, the compiler tried to write the actual file system allocation tables (FAT32/exFAT partition boundaries) onto Sectors 0 and 1. The hardware hit a physical wall and threw a bright red error code:

```
================================================================================
              SILICON CASCADE DEGRADATION DEVIATION REPORT
================================================================================

[INITIAL HEALTH STATUS]
█ █ █ █ █ █ █ █ █ █  [39 Bad Blocks Base Mapping]
└─────────┬────────┘
          └──► Stable Firmware Isolated Gates

[POST-LLF STRESS STATE]
█ █ █ █ █ █ █ █ █ █ ░ ░ ░ ░ ░ ░ ░  [46 Bad Blocks Cumulative Cluster]
                    └───┬───┘
                        ├──► ❌ 7 NEW DIELECTRIC BREAKDOWNS DETECTED
                        └──► CRITICAL VECTOR: SECTOR 0x000000000 (MBR ALIGNMENT ZONE)

[HARDWARE FAULT MATRIX]
┌──────────────────────────────────────────────────────────────────────────────┐
│  [STATUS] CRITICAL MEMORY CONTROLLER FAULT                                  │
│  [SECTOR] INDEX TRACK LOGICAL WEAR-OUT STRATUM                              │
│  [EFFECT] INDUCTION VOLTAGE COLLAPSED PRIMARY FLOATING GATE OXIDES          │
│  [ACTION] THE SYSTEM CANNOT ALLOCATE MASTER FILE TABLES. COMPILATION ABORTED.│
└──────────────────────────────────────────────────────────────────────────────┘

```
```text
🛑 91F00: Create file system error
📊 Bad Block: 46/1348
