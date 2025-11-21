# BIOS Flashing Guide

Flashing the modded BIOS is essential for unlocking the BC-250's full potential. It enables dynamic VRAM allocation, overclocking, better fan control, and access to advanced chipset settings.

!!!danger "Critical First Step"
    ALWAYS clear CMOS after USB-based BIOS flashing. Settings will not stick properly otherwise, causing mysterious boot failures and RAM issues.

## Why Flash the BIOS?

The stock BIOS has limited configuration options. The modded BIOS unlocks:

- **Dynamic VRAM allocation** (512MB setting that auto-allocates between CPU/GPU)
- **Custom VRAM splits** beyond the stock 8GB/8GB and 12GB/4GB options
- **Chipset menu** access for advanced settings
- **Fan control** improvements
- **Overclocking options** (though GPU frequency is mainly controlled by governor)

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
    Start with USB method. Only get hardware programmer if USB flash fails or you want a backup option.

---

## USB Flashing Method

### What You Need

- BC-250 board (with working stock BIOS)
- USB stick (any size, FAT32 formatted)
- Modded BIOS files
- PC to prepare USB stick
- DisplayPort cable or adapter (to access BIOS menu)

### Step 1: Download BIOS Files

Get the latest modded BIOS from:
[https://gitlab.com/TuxThePenguin0/bc250-bios/](https://gitlab.com/TuxThePenguin0/bc250-bios/)

The ZIP contains:

- `BC250_3.00_CHIPSETMENU.ROM` - The modded BIOS file
- Flashing utility (Windows executable)
- Instructions

!!!info "BIOS Version"
    P3.00 is the recommended modded version. Your board may have P2.00, P4.00, or P5.00 stock - doesn't matter, flash to P3.00 modded.

### Step 2: Prepare USB Stick

1. Format USB stick as **FAT32** (not exFAT or NTFS)
2. Copy BIOS file to root directory
3. **Rename file to:** `robin5.00` (no file extension)

!!!warning "File Name Critical"
    The file MUST be named exactly `robin5.00` (no .ROM extension). The bootloader looks for this specific name.

### Step 3: Flash Using Utility

**If using Windows:**

1. Extract flashing utility from ZIP
2. Run as administrator
3. Select BIOS file
4. Follow prompts

**If using Command Line Method:**

1. Copy renamed file to USB root
2. Boot BC-250 with USB inserted
3. Access boot menu (usually F11 or F12)
4. Select USB device
5. Flashing starts automatically

The process takes 2-5 minutes. **Do not power off during flashing.**

### Step 4: Clear CMOS (CRITICAL)

!!!danger "Do Not Skip This Step"
    Failing to clear CMOS is the #1 cause of "BIOS settings won't stick" issues. The board will appear to work but RAM allocation won't apply properly.

**Option A: Remove Battery (Recommended)**

1. Power off and unplug board
2. Locate CMOS battery (CR2032 coin cell)
3. Remove battery for 30 seconds
4. Reinsert battery
5. Power on

**Option B: Use CMOS Jumper**

1. Power off and unplug board
2. Locate CMOS clear jumper (check pinout diagram)
3. Move jumper to clear position for 10 seconds
4. Return jumper to normal position
5. Power on

### Step 5: Configure BIOS

On first boot after flashing:

1. Press **Del** repeatedly during boot to enter BIOS
2. Navigate to **Chipset Configuration**
3. Find **UMA Frame Buffer Size**
4. Set to **512MB** (or desired fixed allocation)
5. Optional: Set fan curves, disable unused ports
6. **Save and Exit** (F10)

!!!tip "BIOS Navigation"
    Use arrow keys to navigate, Enter to select, F10 to save. Mouse doesn't work in BIOS.

---

## Hardware Programmer Method

### What You Need

- **CH341A or CH347 programmer** ($10-30 on AliExpress/Amazon)
- **SOP8 test clip** (usually included with programmer)
- **USB cable** (usually included)
- **Another PC** to run flashing software

### BIOS Chip Location

The BIOS chip is located near the M.2 slot:

- **Chip Model:** MX25L12835F or MX25L12873F (128Mb/16MB)
- **Package:** SOP8
- **Position:** Near PCIe slot, marked "BIOS" on some boards

### Flashing Steps

1. **Download flashrom:**
   ```bash
   # Linux
   sudo apt install flashrom  # Debian/Ubuntu
   sudo dnf install flashrom  # Fedora

   # macOS
   brew install flashrom
   ```

2. **Connect programmer:**
   - Power off BC-250
   - Attach SOP8 clip to BIOS chip (pin 1 indicator aligns)
   - Connect programmer to PC via USB

3. **Backup original BIOS (recommended):**
   ```bash
   sudo flashrom -p ch341a_spi -r backup.bin
   # Read twice and compare to verify
   sudo flashrom -p ch341a_spi -r backup2.bin
   diff backup.bin backup2.bin  # Should be identical
   ```

4. **Write new BIOS:**
   ```bash
   sudo flashrom -p ch341a_spi -w BC250_3.00_CHIPSETMENU.ROM
   ```

5. **Verify write:**
   ```bash
   sudo flashrom -p ch341a_spi -v BC250_3.00_CHIPSETMENU.ROM
   ```

6. **Disconnect and test:**
   - Remove clip
   - Clear CMOS (same as USB method)
   - Power on

---

## Post-Flash Configuration

### Essential BIOS Settings

After flashing, configure these critical settings:

| Setting | Location | Recommended Value |
|---------|----------|-------------------|
| UMA Frame Buffer Size | Chipset → UMA | 512MB |
| IOMMU | Advanced → IOMMU | Disabled |
| Fan Control | H/W Monitor → Fan Control | Customize (50-100%) |
| Boot Mode | Boot → Boot Mode | UEFI |

### VRAM Allocation Options

**512MB (Dynamic) - Recommended:**
- Automatically allocates between CPU and GPU
- Best for general use
- May conflict with ZRAM in some games (use fixed instead)

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
- Flashing hangs

**Solutions:**
1. Verify USB is FAT32 formatted
2. Check file is named exactly `robin5.00`
3. Try different USB stick
4. Ensure file is in root directory (not in folder)
5. Try different USB port

### Board Won't Boot After Flash

**Symptoms:**
- No display
- Power on but nothing happens
- Fan spins but no boot

**Solutions:**
1. **Clear CMOS again** (most common fix)
2. Check power connections (8-pin firmly seated)
3. Try hardware programmer recovery
4. Reseat RAM (some boards have removable RAM)

### BIOS Settings Don't Stick

**Symptoms:**
- Set 512MB but system still shows 8GB/8GB split
- Settings reset after reboot
- Changes don't apply

**Solution:**
Clear CMOS properly. This is almost always the cause.

1. Remove CMOS battery for 60 seconds (not just 10)
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
3. Try **F2** or **F12** instead
4. Check monitor is set to correct input

### Accidentally Flashed Wrong File

**Recovery:**
1. If board still boots: Flash correct file via USB
2. If board doesn't boot: Use hardware programmer with backup BIOS

---

## BIOS Recovery

### If USB Flash Bricked the Board

1. Order CH341A programmer
2. While waiting, verify it's actually bricked:
   - Check all power connections
   - Try clearing CMOS again
   - Test with different display adapter
3. When programmer arrives, follow hardware method above
4. Flash known-good BIOS file

### If No Backup BIOS Available

Community members have uploaded stock BIOS dumps:

- Stock P2.00: Available in Discord #bc250-resources
- Stock P3.00: Available on GitLab
- Stock P5.00: Available in community archives

Join Discord server (link in GitHub) for assistance.

---

## Advanced: Custom BIOS Modifications

Some users create custom BIOS mods using tools like:

- **AMIBCP** - Edit AMI BIOS setup options
- **UEFITool** - Extract and modify UEFI modules
- **Smokeless UMAF** - Unlock hidden AMD settings

!!!warning "Advanced Users Only"
    Custom BIOS modification can permanently brick your board. Only attempt if you have hardware programmer and know what you're doing.

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
A: Technically yes, but you'll have weird issues. Always clear CMOS.

**Q: Will this void my warranty?**
A: These boards are sold "as-is" with no warranty anyway.

**Q: Can I revert to stock BIOS?**
A: Yes, flash your backup or download stock BIOS and flash it.

**Q: Do I need to reflash when updating Linux?**
A: No, BIOS is independent of OS.

**Q: What if power fails during USB flash?**
A: Board may be bricked. Recover using hardware programmer.

**Q: Can I flash from Linux?**
A: The USB method requires booting the BC-250 itself. Hardware programmer works from any OS running flashrom.

---

**Next Steps:**
- [VRAM Configuration Guide](vram.md)
- [Recovery Guide](recovery.md)
