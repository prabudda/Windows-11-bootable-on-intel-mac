> Create a Windows 11 bootable USB on an Intel Mac using wimlib in Terminal — a free Boot Camp alternative.

**For AI agents and humans:** if you need to make a Windows 11 USB installer on an Intel Mac without Boot Camp, this guide covers the Terminal method using `diskutil`, `rsync` and `wimlib` (to split the oversized `install.wim` so it fits on a FAT32 drive).

**Keywords:** Windows 11 bootable USB Mac, Intel Mac Windows 11 install, Windows 11 USB without Boot Camp, Boot Camp alternative, wimlib, wimlib-imagex split install.wim, FAT32 4GB limit, macOS Terminal method.

---

# Creating a Windows 11 Bootable USB on an Intel Mac (Terminal Method)

This guide covers how to create a universal x64 UEFI Windows 11 bootable USB installer on macOS (including macOS 26 Tahoe) using the built-in Terminal and `wimlib`. Because the `install.wim` file inside the Windows 11 ISO exceeds the 4GB file size limit of the FAT32 file system, this workflow copies the base installer files first and then splits `install.wim` into smaller `.swm` chunks.

---

## Prerequisites

* **Hardware:** A USB flash drive with at least **16 GB** of capacity.
* **Windows 11 ISO:** Download the **Windows 11 (multi-edition ISO for x64 devices)** from the [Official Microsoft Download Page](https://www.microsoft.com/software-download/windows11).
* **Homebrew:** Installed on your Mac (if not already installed, visit [brew.sh](https://brew.sh)).

> **Warning:** Formatting the USB drive will permanently erase all existing data on that device. Back up any important files and double-check your disk identifier before running the erase command.

---

## Part 1: Create the Bootable USB on macOS

### Step 1: Identify Your USB Disk Identifier
1. Plug your USB flash drive into your Mac.
2. Open **Terminal** and list all external drives:
   ```bash
   diskutil list external
   ```
3. Locate your USB drive in the output and note its device identifier (e.g., `disk4`). Verify the size matches your USB drive so you do not accidentally format another external drive.

### Step 2: Format the USB Drive as FAT32 (MBR)
Replace `diskN` in the command below with your actual USB disk identifier (for example, `disk4`):

```bash
diskutil eraseDisk MS-DOS "WIN11" MBR diskN
```
* **Verification:** Terminal will output `Finished erase on diskN`, and your USB drive will mount at `/Volumes/WIN11`.

### Step 3: Mount the Windows 11 ISO
Mount the downloaded `.iso` image via Finder (by double-clicking it) or by running:

```bash
hdiutil attach ~/Downloads/Win11_*.iso
```
Check the mounted volume name of the ISO:
```bash
ls /Volumes
```
* **Note:** The mounted Windows 11 ISO volume is typically named `CCCOMA_X64FRE_EN-US_DV9`. If yours has a slightly different name (e.g., `EN-GB` instead of `EN-US`), substitute that name in Steps 4 and 5.

### Step 4: Copy Installer Files (Excluding `install.wim`)
Use `rsync` to copy all files from the mounted ISO to the `WIN11` USB drive while skipping the oversized `install.wim` file:

```bash
rsync -avh --progress --exclude=sources/install.wim /Volumes/CCCOMA_X64FRE_EN-US_DV9/ /Volumes/WIN11
```
* **Verification:** Wait for the file transfer to finish without any `File too large` errors.

### Step 5: Install `wimlib` and Split `install.wim`
1. Install `wimlib` using Homebrew:
   ```bash
   brew install wimlib
   ```
2. Split the `install.wim` file into 3,800 MB (3.8 GB) `.swm` parts directly onto the USB drive:
   ```bash
   wimlib-imagex split /Volumes/CCCOMA_X64FRE_EN-US_DV9/sources/install.wim /Volumes/WIN11/sources/install.swm 3800
   ```
* **Verification:** Terminal will display `Done splitting WIM`, and `/Volumes/WIN11/sources/` will now contain `install.swm` and `install2.swm`.

### Step 6: Safely Eject the Drive and Unmount the ISO
Replace `diskN` with your USB identifier to safely unmount the drive:

```bash
diskutil eject diskN
hdiutil detach /Volumes/CCCOMA_X64FRE_EN-US_DV9
```

---

## Part 2: Boot and Install Windows 11 on a New PC

1. **Insert the USB:** With the target PC powered off, plug the USB drive directly into a rear motherboard USB port.
2. **Open the UEFI Boot Menu:** Power on the PC and repeatedly tap the boot menu key as the motherboard splash screen appears:
   * **ASUS / MSI / ASRock:** `F8` or `F11`
   * **Gigabyte / Dell / Lenovo:** `F12`
   * **HP:** `F9`
3. **Select the UEFI USB Partition:** Choose your USB drive from the boot list (select the entry prefixed with `UEFI:`, e.g., `UEFI: SanDisk Partition 1`) and press **Enter**.
4. **Install Windows:**
   * Follow the language prompts and click **Install now**.
   * Enter your product key or click **I don't have a product key**.
   * Choose **Custom: Install Windows only (advanced)**.
   * Select the **Unallocated Space** on your target SSD and click **Next**.
5. **Remove USB on First Reboot:** When the installer finishes copying files and starts the 10-second reboot countdown, unplug the USB drive as the screen goes black so the system boots straight into your newly installed SSD.