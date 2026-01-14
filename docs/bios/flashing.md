# BIOS Flashing Guide

Flashing the modded BIOS is essential for unlocking the BC-250's full potential. It primarily enables **dynamic VRAM allocation** and grants access to **advanced chipset settings** that are hidden in the stock configuration.

!!!danger "Critical: Clear CMOS"
    **ALWAYS** clear CMOS after flashing. While the board may post without it, settings (especially VRAM allocation) will not stick properly unless the CMOS is cleared, leading to confusion and "mysterious" bugs.

## Why Flash the BIOS?

While the stock BIOS includes standard features like fan control, the modded BIOS specifically unlocks:

- **Dynamic VRAM allocation** (512MB setting that auto-allocates between CPU/GPU)
- **Custom VRAM splits** beyond the stock 8GB/8GB and 12GB/4GB options
- **Chipset menu** access for advanced configuration options

*Note: Actual overclocking is generally not performed via the BIOS on this platform, and fan control is available on both stock and modded versions.*

### Available Modded Versions

There are two main versions of the modded BIOS floating around the community:

*   **P3.00 Chipset Menu (Recommended):** This is the community standard. It is the most stable and tested version. It successfully unlocks VRAM allocation and chipset settings without introducing unnecessary instability.
*   **P5.00_clv:** Based on a newer stock code base. It specifically unlocks **Everything**—every hidden menu and setting available. This includes experimental options like ReBAR (Resizable BAR). However, because it exposes critical debug and chipset settings, it is very easy to brick the board if you change the wrong thing. **Stick to P3.00 unless you are an advanced user who knows exactly what they are doing.**

---

## Flashing Methods

There are two ways to flash the BIOS:

### Method 1: USB Flashing (Recommended)

**Pros:**
- No special hardware needed
- Fast
- Works on most boards

**Cons:**
- Requires working board
- Small risk of bricking (recoverable with hardware method)
- MUST clear CMOS afterward

### Method 2: Hardware Programmer

**Pros:**
- Can recover from failed USB flash
- Most reliable method
- Can backup original BIOS

**Cons:**
- Requires CH341A/CH347 programmer
- More technical
- Slower process

!!!tip "Recommendation"
    While USB flashing is convenient, owning a CH347 programmer before you start is highly recommended as a safety net. If USB flashing fails, the board is unusable until you use a hardware programmer.

---

## Method 1: USB Flashing (EFI Shell Method)

This is the standard way to flash the BC-250. It uses the internal EFI Shell rather than a Windows application.

### Prerequisites
*   **USB Stick:** formatted to **FAT32** (Max 32GB recommended).
*   **BC-250 Board:** Must be in working order.
*   **Display:** Direct DisplayPort connection is highly recommended.
    *   *Warning:* Active/Passive HDMI adapters can cause black screens in the BIOS menu.

### Step 1: Download Files

You need two things: the **Flashing Tools** (EFI shell utilities) and the **Modded BIOS File** itself.

1.  **Download the Flashing Tools (EFI Kit):**
    *   [**Click here to download (4U12G BIOS Update.zip)**](https://github.com/kenavru/BC-250/raw/refs/heads/main/4U12G%20BIOS%20Update.zip)
    *   *This zip contains the essential `AfuEfix64.efi` and `Flash.nsh` scripts.*
    *   **Note:** This zip also contains a **Stock P5.00 BIOS**. Do not use this file if you intend to flash the modded version.

2.  **Download the Modded BIOS ROM:**
    *   [**TuxThePenguin0 GitLab**](https://gitlab.com/TuxThePenguin0/bc250-bios/)
    *   Download the recommended version (**BC250_3.00_CHIPSETMENU.ROM**).

### Stock BIOS Sources (For Recovery/Reversion)
If you need to revert to stock or recover a board, stock files can be found here:
*   **Stock P5.00:** Included in the flashing tool ZIP linked above.
*   **Stock P3.00:** Available from user **Segfault**.
*   **Other Versions:** The community **Discord** archives contain various other stock versions (P2.00, etc.).

### Step 2: Prepare the USB Stick

1.  Format your USB stick to **FAT32**.
2.  **Extract the Tools:** Unzip the contents of `4U12G BIOS Update.zip`, and copy the contents of `BIOS EFI` to the **root** of the USB stick.
3.  **Save the Stock BIOS:** Move the `Robin5.00` file somewhere safe (this is stock P5.00).
4.  **Copy the Modded BIOS:** Place your downloaded modded BIOS file (e.g., `BC250_3.00_CHIPSETMENU.ROM`) onto the root of the USB stick.
5.  **Rename/Configure:**
    *   **Rename your modded BIOS file** to `Robin5.00` (remove the `.ROM` extension).
    *   *Alternatively, edit `Flash.nsh` to match your filename.*

    **Your USB Root should typically contain:**

    *   `AfuEfix64.efi`
    *   `Flash.nsh`
    *   `amdvbflash.efi`
    *   `Robin5.00` (Your renamed modded BIOS file)
    *   `EFI` (folder)

### Step 3: Boot to EFI Shell

The easiest way to boot the tool is to force the board to look for the USB stick automatically.

1.  **Unplug all Drives and SSDs.**
    *   If no OS drive is detected, the BC-250 will automatically default to the EFI Shell/USB stick.
2.  Insert the USB stick.
3.  Power on the BC-250.
4.  The system should bypass the standard boot order and load directly into the EFI Shell (Yellow text on black background).

### Step 4: Execute the Flash

Once you are at the yellow `Shell>` prompt, follow this exact sequence:

1.  Type `blk0:` and press **Enter**.
    *   **Ensure you add a space after the colon**
    *   *This selects your USB drive.*
2.  Type `Flash.nsh` and press **Enter**.
    *   *This executes the flashing script.*
3.  **WAIT.** You will see the AMI Firmware Update Utility run.
    *   *Do not touch the keyboard.*
    *   *Do not power off.*
    *   *If the process appears to hang during the flash, wait at least 15 minutes. Powering off while writing will brick the board.*
4.  The system will reboot automatically (or ask you to reboot) when finished.

### Step 5: Power Down & Remove USB

Once the flashing process finishes and the system attempts to reboot:

1.  **Power off the BC-250 immediately.**
2.  **Remove the USB stick.**
    *   *This prevents the system from accidentally booting back into the EFI shell or attempting to flash again.*

### Step 6: The Critical CMOS Clear

**Do not skip this.**

**Option A: Remove Battery (Recommended)**

1.  **Remove the CMOS Battery** (CR2032) for at least 60 seconds.
2.  (Optional) Press the power button a few times while unplugged to discharge capacitors.
3.  Reinsert battery.

**Option B: Use CMOS Jumper**

1.  **Locate CMOS clear jumper**
2.  Move jumper to clear position for 20 seconds
3.  Return jumper to normal position.
4.  Power on.

### Step 7: BIOS Configuration

1.  Power on and spam **Del** to enter BIOS.
2.  Verify CMOS was cleared. The time/clock should be wrong. If not repeat Step 6 (The Critical CMOS Clear).
3.  Navigate to: **Chipset** → **GFX Configuration**.
4.  Set **Integrated Graphics Controller** to **Forces**.
5.  Set **UMA Mode** to **UMA_SPECIFIED**.
6.  Set **UMA Frame Buffer Size** to **512MB** (Recommended) or your preferred fixed size.
7.  Navigate to: **Advanced** → **CPU Configuration**.
8.  Set **IOMMU** to **Disabled**.
9.  Press **F10** to Save and Exit.

---

## Method 2: Hardware Programmer (Recovery & Backup)

This method writes directly to the SPI flash chip, bypassing the CPU. It is the **only** way to unbrick a board that will not POST.

**Credits:** Massive thanks to **Segfault** for the reverse engineering, pinout documentation, and maintaining the repository of modified firmware images.

### Critical Warnings

1.  **The 5V Trap:**
    *   **Do NOT use black-PCB CH341A programmers** (commonly found on Amazon/AliExpress). They often output **5V logic** even when set to 3.3V mode.
    *   The BC-250 BIOS chip operates at **3.3V**. Using 5V logic can fry the chip or the connected chipset.
2.  **Identify the Correct Chip (Don't Brick the SuperIO):**
    *   The board has *two* flash chips. Flashing the wrong one will brick the SuperIO controller (fan control/sensors).
    *   **✅ TARGET:** `BIOS_A1` (16MB capacity). Usually Winbond or Macronix.
    *   **❌ AVOID:** `SIO1_R` (512KB capacity). This is a small Macronix chip nearby. **Do not touch this.**

### 1. Tools & Hardware

*   **Programmer:**
    *   **WCH CH347** (Recommended - Native 3.3V, fast).
    *   **Raspberry Pi Pico** (Excellent 3.3V alternative using `serprog` firmware).
    *   *Avoid standard CH341A unless you have verified 3.3V logic levels.*
*   **Connection:** Female-to-Female DuPont wires (for J4004 header) or an SOP8 Test Clip.

### 2. Chip Identification & Pinout

**Target Chip (`BIOS_A1`):**
*   **Likely Model:** Winbond **W25Q128JVSQ** (128M-bit / 16MB)
    *   *Note: Some community docs typo this as "25Q168". The correct density code for 16MB is 128.*
*   **Alternative Model:** Macronix **MX25L12835F** (found on some batches).
*   **Location:** Component `BIOS_A1`, near the PCIe slot/M.2 area.

**Programming Header (`J4004`):**
The board features a 2.54mm header specifically for flashing. This is safer than a clip.

**J4004 Pinout:**

| Pin | Function | | Function | Pin |
| :-: | :-: | :-: | :-: | :-: |
| **2** | **GND** | `[` `]` | **VCC (3.3V)** | **1** |
| **4** | **SCLK** | `[` `]` | **CS** | **3** |
| **6** | **MOSI** | `[` `]` | **MISO** | **5** |
| **8** | *(UNK)* | `[` `]` | *(UNK)* | **7** |

*   **Orientation:** Pin 1 (VCC) is marked by the arrow `>` or a square pad on the PCB.
*   **Note:** Pins 7 & 8 are grounded via 10kΩ resistors and are unused for flashing.

### 3. Flashing Process

**Prerequisites:**
*   **Unplug the PSU from the wall.**
*   Press the power button several times to discharge capacitors.
*   **ALWAYS** create a backup.

#### Software Steps (Linux/Flashrom)

1.  **Install Flashrom:**
    ```bash
    sudo apt install flashrom
    ```
2.  **Test Connection & Identify Chip:**
    ```bash
    # Replace 'ch347_spi' with your programmer (e.g., 'serprog' for Pi Pico)
    sudo flashrom -p ch347_spi
    ```
    *   *If it detects "Winbond W25Q128..." or "Macronix MX25L128...":* **Success.** You are on the right chip.
    *   *If it detects "Macronix MX25L4005..." (512KB):* **STOP.** You are attached to the SuperIO chip. Move to the other chip.
3.  **Backup (Essential):**
    ```bash
    sudo flashrom -p ch347_spi -r backup_stock.bin
    # Verify backup integrity
    sudo flashrom -p ch347_spi -r backup_verify.bin
    diff backup_stock.bin backup_verify.bin
    ```
4.  **Flash Firmware:**
    ```bash
    sudo flashrom -p ch347_spi -w BC250_3.00_CHIPSETMENU.ROM
    ```

### 4. Post-Flash Configuration

1.  Enter BIOS → **Chipset** → **GFX Configuration**.
2.  Set **Integrated Graphics Controller** to **Forces**.
3.  Set **UMA Mode** to **UMA_SPECIFIED**.
4.  Set **UMA Frame Buffer Size** to **512M**.
5.  Navigate to: **Advanced** → **CPU Configuration**.
6.  Set **IOMMU** to **Disabled**.


!!!warning "Safety Notice"
    The modded BIOS exposes many settings that are untested. Changing random voltages, timings, or unknown chipset options can permanently damage the board. **If you don't know what it does, do not touch it.**

## Post-Flash Configuration

### Essential BIOS Settings

After flashing, configure these critical settings:

| Setting | Location | Recommended Value |
|---------|----------|-------------------|
| UMA Frame Buffer Size | Chipset → UMA | 512MB |
| IOMMU | Advanced → IOMMU | Disabled |
| Boot Mode | Boot → Boot Mode | UEFI |

### VRAM Allocation Options

**512MB (Dynamic) - Recommended:**
- Automatically allocates between CPU and GPU
- Best for general use

**Fixed Allocations:**
- 10GB RAM / 6GB VRAM - Good for AAA games
- 8GB RAM / 8GB VRAM - Balanced
- 12GB RAM / 4GB VRAM - Light gaming, more system RAM

[Detailed VRAM guide →](vram.md)

---

## Troubleshooting

### USB Flash Failed / No Response

**Symptoms:**

- USB boot doesn't start

**Solutions:**

1. Verify USB is FAT32 formatted
2. Check file is named exactly `Robin5.00`
3. Try different USB stick
4. Ensure file is in root directory (not in folder)
5. Try different USB port

### Flash Hangs

**Symptoms:**
- Progress bar freezes or system becomes unresponsive.

**Solutions:**

*   **Hangs before utility starts:** You can reboot safely.
*   **Hangs during flash:** Do **NOT** reboot. Wait 15 minutes.

### Board Won't Boot After Flash

**Symptoms:**

- No display
- Power on but nothing happens
- Fan spins but no boot

**Solutions:**

1. **Clear CMOS again** (most common fix)
2. Check power connections (8-pin firmly seated)
3. Try hardware programmer recovery

### BIOS Settings Don't Stick

**Symptoms:**

- Set 512MB but system still shows 8GB/8GB split
- Settings reset after reboot
- Changes don't apply

**Solution:**
Clear CMOS properly. This is almost always the cause.

1. Remove CMOS battery for 60 seconds
2. With battery removed, press power button 5 times (discharges capacitors)
3. Reinsert battery
4. Boot and reconfigure

### Display Shows But BIOS Menu Won't Appear

**Symptoms:**

- Board boots to black screen
- No BIOS logo
- Can't access BIOS setup

**Solutions:**

1. Try different display cable/adapter
2. Spam **Del** key earlier (right when powering on)
3. Check monitor is set to correct input

### Accidentally Flashed Wrong File

**Recovery:**

1. If board still boots: Flash correct file via USB
2. If board doesn't boot: Use hardware programmer with backup BIOS

---

## BIOS Recovery

### If USB Flash Bricked the Board

1. Order CH347 programmer
2. While waiting, verify it's actually bricked:
   - Check all power connections
   - Try clearing CMOS again
   - Test with different display adapter
3. When programmer arrives, follow hardware method above
4. Flash known-good BIOS file

Join Discord server (link in GitHub) for assistance.

---

## Verification

After successful flash and configuration:

```bash
# Check VRAM allocation in Linux
cat /proc/meminfo | grep -i mem
# Should show ~10-12GB depending on your split

# Check GPU detected
lspci | grep VGA
# Should show AMD Radeon Graphics

# Verify BIOS version
sudo dmidecode -t bios
# Should show P5.00 or your modded version
```

---

## FAQ

**Q: Can I flash without clearing CMOS?**

A: Technically yes, but you may have weird issues. Clearing CMOS is recommended.

**Q: Will this void my warranty?**

A: These boards are sold "as-is" with no warranty anyway.

**Q: Can I revert to stock BIOS?**

A: Yes, flash your backup or download stock BIOS and flash it.

**Q: Do I need to reflash when updating Linux?**

A: No, BIOS is independent of OS.

**Q: What if power fails during USB flash?**

A: The board may be bricked, and may require a hardware flash to recover.

**Q: Can I flash from Linux?**

A: The USB method requires booting the BC-250 itself. Hardware programmer works from any OS running flashrom.

---

**Next Steps:**
- [VRAM Configuration Guide](vram.md)
- [Recovery Guide](recovery.md)
