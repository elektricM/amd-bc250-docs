# BIOS Flashing Guide

Flashing the modded BIOS is the recommended way to unlock the BC-250's full potential. It primarily enables **dynamic VRAM allocation** and grants access to **advanced chipset settings** that are hidden in the stock configuration.

!!!tip "Not always required"
    If your only goal is changing VRAM size, you can do that from Linux on the stock P3.00 / P5.00 BIOS using [bc250_memcfg](https://github.com/fanoush/bc250_memcfg). See [VRAM Configuration](vram.md#changing-the-vram-split) for details. Flashing is only needed if you want the unlocked chipset menus or features beyond VRAM sizing.

!!!danger "Hardware Safety: GPU Power Connector"
    Ensure the 8-pin PCIe power connector is wired correctly BEFORE attempting to flash.
    - Verify pinout matches your PSU diagram (12V vs GND positions)
    - Reversed polarity can permanently damage the board
    - If unsure, verify with community Discord before powering on

!!!danger "Critical: Clear CMOS"
    Clearing CMOS after flashing resets to default settings and ensures VRAM allocation settings apply correctly. While some users report successful flashes without clearing, it is strongly recommended as best practice.

## Why Flash the BIOS?

While the stock BIOS includes standard features like fan control, the modded BIOS specifically unlocks:

- **Dynamic VRAM allocation** (512MB setting that auto-allocates between CPU/GPU)
- **Custom VRAM splits** beyond the stock 8GB/8GB and 12GB/4GB options
- **Chipset menu** access for advanced configuration options
- **All 8 CPU cores** (MeiMeiDXE only) — see [8 Core CPU Unlock](../system/8core-unlock.md)

*Note: Actual overclocking is generally not performed via the BIOS on this platform, and fan control is available on both stock and modded versions.*

### Available Modded Versions

There are two main versions of the modded BIOS floating around the community:

*   **P3.00 Chipset Menu (Recommended):** This is the community standard. It is the most stable and tested version. It successfully unlocks VRAM allocation and chipset settings without introducing unnecessary instability.
*   **P3.00 MeiMeiDXE-T-v2:** P3.00 with the chipset menu **plus a `Bc250CoreUnlockDxe` driver that enables all 8 CPU cores** at every boot. From [Forbidden-Darkness](https://github.com/Forbidden-Darkness/AMD-BC-250-UEFI-v2.2-Firmware-Menu-Script). Only worth flashing if you want the core unlock permanent — you can try the cores out first from Linux with no flash at all, see [8 Core CPU Unlock](../system/8core-unlock.md). Note it flashes the boot block, so treat it as the highest-risk option here.
*   **P5.00_clv:** Based on a newer stock code base. It specifically unlocks **Everything**—every hidden menu and setting available. This includes experimental options like ReBAR (Resizable BAR). However, because it exposes critical debug and chipset settings, it is very easy to brick the board if you change the wrong thing. **Stick to P3.00 unless you are an advanced user who knows exactly what they are doing.**

!!!warning "P5.00_clv availability"
    As of this writing, `P5.00_clv` is not published in any public repository we know of (GitHub, GitLab, archive.org, community wikis). It only circulates as a Discord attachment, which means there is no canonical hash anyone can hand you for verification. If you want to run it, the safest approach is to ask in the BC-250 Discord for at least two people running it independently, get a copy from each, and confirm both copies have the same SHA256 before flashing. If you only need VRAM unlock or chipset settings, `BC250_3.00_CHIPSETMENU.ROM` covers it and has multiple verified public sources (see below).

---

## Flashing Methods

There are three ways to flash the BIOS:

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

### Method 3: flashrom from Linux (Advanced)

**Pros:**
- No USB stick, no EFI shell, no programmer: backup and flash from the running OS
- Cannot touch the wrong chip: `-p internal` only reaches the main BIOS flash
- Staged writes can leave the boot block untouched, keeping crisis recovery intact

**Cons:**
- Advanced users only, you are writing the chip the board is running from
- Needs a pre-flight checklist and a region diff to be done safely
- Not the community-standard route

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

### Verified BIOS Sources and SHA256 Hashes

If you want to verify a file before flashing (and you really should), here are the BIOS files that are publicly hosted on at least one community repo, with hashes confirmed across multiple independent sources where available. Always run `sha256sum your_file.rom` and compare before doing anything to your board.

| File | Type | SHA256 | Sources |
|------|------|--------|---------|
| `BC250_3.00_CHIPSETMENU.ROM` | Modded P3.00 (VRAM + chipset unlock, **recommended**) | `48fbe5d366e6a56e2fdffdca848426216ba1f083610dab63db89d2f4e6c940b5` | [TuxThePenguin0 (GitLab)](https://gitlab.com/TuxThePenguin0/bc250-bios/-/blob/main/BC250_3.00_CHIPSETMENU.ROM), [forgenam](https://github.com/forgenam/BC250-Bios-Update-Guide), [tipitochen](https://github.com/tipitochen/debiantools_bc250flash), [csabakecskemeti](https://github.com/csabakecskemeti/amd_bc-250_how-to) (named `Robin3.00` in his repo), [scrakcho](https://github.com/scrakcho/BC-250), [dannybastos](https://github.com/dannybastos/bc-250-archlinux) (inside `Mod bios.zip`) |
| `Robin5.00` | Stock P5.00 (16 MB) | `0d6f136cb120cf3b2de26d5c4d7f255604fdbf4b9442af5ba55419b95b89aa82` | [forgenam](https://github.com/forgenam/BC250-Bios-Update-Guide), [MrrZed0](https://github.com/MrrZed0/bc-250-bios), [csabakecskemeti](https://github.com/csabakecskemeti/amd_bc-250_how-to), [scrakcho](https://github.com/scrakcho/BC-250) (inside the UEFI MOD zip), [dannybastos](https://github.com/dannybastos/bc-250-archlinux) (inside `Mod bios.zip`) |
| `BC250_3.00.ROM` | Stock P3.00 (16 MB) | `07595ca3aecf8a4caa28a397b5298f3946a1b769f87b16f67adc369c3f69045c` | [TuxThePenguin0 (GitLab)](https://gitlab.com/TuxThePenguin0/bc250-bios/-/blob/main/BC250_3.00.ROM) |
| `BC250_2.00.bin` | Stock P2.00 (16 MB) | `ee6150dfed33bd05ea46063a352549416fdf3f45fa0e5edac2a68ef78d71083c` | [kenavru](https://github.com/kenavru/BC-250) |
| `P5.00_clv` | Modded P5.00 (unlock everything) | not published anywhere we can find | Discord only, no public hash |

A few notes:

- The Modded P3.00 file shows up under different names in different repos (`BC250_3.00_CHIPSETMENU.ROM`, `BC250CHIPSETMENU.ROM`, `Robin3.00`). They all hash to the same value above, so don't worry about the naming.
- `Robin5.00` is the **stock** P5.00, not the modded P5.00_clv. They are different files.
- If your dump from `AfuEfix64` matches one of these hashes, you have a known-good copy. If it doesn't, your board may be running a different stock revision or a custom mod, and you should treat anything you flash with extra care.

!!!tip "Quick reference for what to flash"
    For 99% of users, the answer is `BC250_3.00_CHIPSETMENU.ROM` (hash above). It unlocks VRAM allocation and the chipset menu, which is what people actually want from a modded BIOS.

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

## Method 3: Flashing from Linux with flashrom (Advanced)

The board's BIOS chip can be read and written from the running OS with `flashrom -p internal`. No USB stick, no EFI shell, no programmer. This procedure was verified start to finish on a real board (staged write, byte-identical read-back, clean reboot onto the new firmware). Tested by @Weijtmans on Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29.

!!!danger "Advanced users only"
    You are erasing and rewriting the flash chip of the board you are booted from. Linux keeps running from RAM while you do it, but a mistake still means the next boot fails, and then your only ways back are USB recovery or a hardware programmer. Read the whole section before typing anything.

A safety property worth knowing: the board has [two flash chips](../hardware/pinouts.md): the 16MB BIOS chip on the FCH SPI bus and the 512KB SuperIO chip (fan control) on LPC. `-p internal` can physically only reach the FCH SPI chip, so the "accidentally flashed the SuperIO" failure mode of the programmer route cannot happen here.

### 1. Back up your current BIOS, twice

```bash
sudo flashrom -p internal -r bios-backup-1.rom
sudo flashrom -p internal -r bios-backup-2.rom
cmp bios-backup-1.rom bios-backup-2.rom
# No output = the two reads are byte-identical = the dump is trustworthy
sha256sum bios-backup-1.rom
```

Two independent, identical reads prove the dump is good. Without a hardware programmer this backup is your **only** recovery image. Record the hash and store a copy off the board.

flashrom will identify the chip. Boards vary: some carry a Winbond W25Q128, others a Macronix (`MX25L12835F/MX25L12873F`, 16384 kB). Pass the identified chip explicitly with `-c` in every later command.

### 2. Pre-flight checklist

Each of these can make an internal flash fail or refuse:

| Check | Command | Must be |
|---|---|---|
| Kernel lockdown | `cat /sys/kernel/security/lockdown` | `[none]` |
| Secure Boot | `mokutil --sb-state` | disabled |
| Firmware daemons | `systemctl stop fwupd` | stopped during the flash |
| Chip write protection | flashrom prints the status register | SRWD and BP0–BP3 all clear |

flashrom shows a scary warning about internal flashing on laptops with an EC sharing the flash. On the BC-250 the IMC (AMD's embedded controller) is not active and no EC shares the BIOS chip, which is what makes that warning moot on this board.

### 3. Diff first, then write only what differs

flashrom cannot enumerate AMD FCH protected ranges, so a naive full-chip write could die mid-chip on a protected region and leave a partial image, in other words a brick. The safe pattern is to write **only the regions that actually differ** between your dump and the target image, in stages, starting with a small low-risk region:

```bash
# Which 64KB blocks differ between current BIOS and target?
cmp -l bios-backup-1.rom new-bios.rom | awk '{print int(($1-1)/65536)}' | uniq
```

For the common community 3.00-lineage images, the differences fall entirely in the NVRAM region (`0x000000–0x01ffff`) and a varstore/DXE range in the middle of the image; the top 512KB **boot block is byte-identical across the lineage**. That matters: an untouched boot block keeps AMI's USB crisis recovery working even if a later stage fails.

!!!warning "If the boot block differs, stop"
    If the block diff shows differences in the last 512KB of the chip, this method's safety argument does not hold for your image pair. Use the USB method instead.

Write with a flashrom layout file, verifying between stages. NVRAM first (proves erase/write/verify works on your board while the rest is untouched), then the remaining differing regions:

```bash
cat > layout.txt <<'EOF'
000000:01ffff nvram
0ab0000:0abffff varstore
0ae0000:0c2ffff dxe
EOF
# Region bounds above are for the community 3.00-lineage images.
# Derive your own from the block diff if yours differ.

sudo flashrom -p internal -c "<your chip>" -l layout.txt -i nvram -w new-bios.rom
sudo flashrom -p internal -c "<your chip>" -l layout.txt -i varstore -i dxe -w new-bios.rom
```

### 4. Verify before rebooting

```bash
sudo flashrom -p internal -r readback.rom
sha256sum readback.rom new-bios.rom
# Hashes match = the chip now holds exactly the target image. Only then reboot.
```

On the verified run the full-chip read-back hash matched the target ROM exactly, with the boot block never erased or written.

### 5. CMOS clear and reconfigure

Same as the USB method: clear CMOS afterwards and redo your BIOS settings, see [The Critical CMOS Clear](#step-6-the-critical-cmos-clear).

To restore your original BIOS later (as long as Linux still boots):

```bash
sudo flashrom -p internal -c "<your chip>" -w bios-backup-1.rom
```

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
