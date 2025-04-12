# Crazyradio PA Detection Fix Documentation

## Author: Elenge Germain
## Date: April 10, 2025 
## Device: Crazyradio PA  
## Client: cfclient  
## OS: Windows 11 (Python 3.13), macOS (Python 3.11)

---

## Overview

This document outlines the steps taken to resolve the issue of the **Crazyradio PA dongle** not being detected by `cfclient`. The issue was *not related* to the `cfclient` software, its firmware, or dependencies. The root cause was improper driver association on the system for the dongle interface, which required manual correction using **Zadig** and a manual flashing of a **.uf2** firmware into Crazyradio Dongle.

---

## Issue

- Crazyradio PA was **not detected** in `cfclient`.
- `cfclient` and associated firmware packages were functioning as intended.
- Device interfaces showed up in **Zadig** as `nRF Serial`, `nRF UF2`, `HCT USB Entry Keyboard`, and `USB Optical Mouse`.

---

## Main Cause

The Crazyradio PA dongle was incorrectly recognized by the OS and had an incompatible driver (`USBSTOR`) associated with its interface(s). This prevented it from functioning as intended in `cfclient`.

---

## Resolution Steps

###Before starting, open Device Manager and check if Crazyradio 2 appears under the **portable devices** section

### 1. **Install and Open Zadig**
```
Download from: https://zadig.akeo.ie/
Run Zadig with Admin privileges.
```

### 2. **Identify the Correct Interface**
- Open Zadig.
- Go to `Options > List All Devices`.
- In the dropdown, locate:
  - CrazyRadio 2 - **If shows up in the drop-down, skip to step 6. otherwise continue with the order**
  - `nRF Serial (Interface 0)`
  - `nRF UF2 (Interface 2)`
- These are **Crazyradio PA** related entries.

> ❗ *Note*: Interface 2 (`nRF UF2`) is related to DFU mode, not regular radio communication.

---

### 3. **Check Original Driver (if known)**
- Original driver for the Crazyradio interface was:
  ```
  USBSTOR (v10.0.22621.439...)
  ```
- I *accidentally replaced* during troubleshooting but was able to fix later.

---

### 4. **Replace Driver**
- Select: `nRF Serial (Interface 0)`
- Target driver: `WinUSB` (via `libusbK` or `libusb-win32` may also work, but `WinUSB` is what I'd recommended for compatibility).
- Click: `Replace Driver`

### 5. Enter Bootloader Mode

To flash new firmware, you must first put the Crazyradio 2.0 into **bootloader mode**:

1. **Hold the button** on the Crazyradio 2.0 dongle.
2. **Insert it into a USB port** while holding the button.
3. **Check for a red pulsing LED**:
   - A slowly pulsing red LED confirms the device is in bootloader mode.

---

### 6. Flash New Firmware

Once in bootloader mode, the device should show up as a USB drive named `Crazyradio2`.

1. **Download the firmware:**
   - Go to the [Crazyradio2 Releases Page](https://github.com/bitcraze/crazyradio2-firmware/releases/tag/1.1)
   - Download the file named:  
     `crazyradio2-CRPA-emulation-[version].uf2`
2. **Install the firmware:**
   - Drag and drop the `.uf2` file onto the `Crazyradio2` folder in File Explorer.
3. **Check installation:**
   - It installs in less than a second.
   - The device will reboot and the `Crazyradio2` USB drive will disappear.
   - A brief **white LED flash** on startup confirms a successful flash.

    > If you missed the white flash, unplug and plug it back in.


---

### 7. **Installing USB driver on Windows**
After installing the firmware, reopen Zadig and select Crazyradio 2 
1. The current driver of Crazyradio 2 should display **`(NONE)`**
2. Select the target driver to be libusb-win32 and click install

### 8. Done and now you can test the device
- Open **cfclient**
- Open the **Help** section, and click **About**
- Open the **debug** menu and the following should be shown:
  -interface status
   radio: Crazyradio version 3.2
   
Thus completing the fix.
---

## Notes

- The **problem was entirely on the Crazyradio PA dongle side**, **not** with the `cfclient`, firmware, or USB port.
- Firmware flashing is required for the crazyradio dongle to be detected
- As of this writing, **Python 3.13 is not supported** by cfclient on **macOS**. Use Python 3.12 or lower for best compatibility.

---

## Optional: Restore Original Driver (if needed)
If USBSTOR is required for other devices:
```
Device Manager > Crazyradio Device > Properties > Driver > Roll Back Driver
```
But keep `WinUSB` for the dongle's interface when used with `cfclient`.

---
And that concludes the fix
---
