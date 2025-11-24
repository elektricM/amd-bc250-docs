# BIOS Recovery Guide

Guide to recovering from a bad BIOS flash or corrupted BIOS on the BC-250.

!!!danger "Last Resort Guide"
    Only use this guide if your BC-250 won't boot after a BIOS flash. Prevention is always better than recovery.

## When You Need Recovery

### Symptoms of Corrupted BIOS

- No display output (black screen)
- No POST (no beep, no activity)
- Board powers on but nothing happens
- Fans spin but no boot
- BIOS settings don't persist after reboot

### Causes

1. **Bad BIOS flash:** Flashing incorrect BIOS file
2. **Interrupted flash:** Power loss during flashing
3. **Corrupted BIOS file:** Downloaded corrupted file
4. **Wrong flash method:** Used incompatible flashing tool
5. **CMOS not cleared:** After USB flash, settings don't stick

## Recovery Methods

### Method 1: Clear CMOS (Try First)

**When to use:** Settings don't stick, board acts strange after flash

**Process:**

1. Power off board completely
2. Unplug power cable
3. Locate CMOS battery (small coin cell on board)
4. Remove CMOS battery
5. Wait 60 seconds
6. Reinsert CMOS battery
7. Reconnect power
8. Boot and reconfigure BIOS

**OR use CMOS jumper:**

1. Power off and unplug
2. Locate CMOS clear jumper (near battery)
3. Move jumper to clear position
4. Wait 10 seconds
5. Move jumper back
6. Reconnect power and boot

!!!tip "USB Flash CMOS Issue"
    If you flashed via USB and settings don't persist, clearing CMOS usually fixes it. This is a known issue with USB flashing.

### Method 2: Re-flash via USB (Partial Brick)

**When to use:** Board boots but unstable, or partial BIOS corruption

**Requirements:**
- USB flash drive (FAT32 formatted)
- Correct BIOS file (BC250_3.00_CHIPSETMENU recommended)
- Flasher program

**Steps:**

1. Format USB drive as FAT32
2. Copy BIOS files to root of USB:
   - `SHELL.efi` or flasher executable
   - BIOS file (rename to required name if needed)
3. Insert USB into BC-250
4. Power on
5. Access BIOS recovery mode (if available)
6. Flash BIOS from USB

**Note:** This only works if board still partially boots

### Method 3: Hardware Programmer (Full Brick)

**When to use:** Board completely dead, no activity

**Required Equipment:**
- CH341A programmer (~$5-15) OR
- CH347T programmer (~$15-30, recommended)
- SOP8 test clip OR
- SOIC8 chip clip
- Jumper wires (if needed)

#### SPI Flash Chip Details

**Location:** Near BIOS chip on board

**Chip Model:**
- **MX25L12873F** (most common)
- MX25L12835F (some boards)
- Winbond 25Q128 (rare)

**Chip Package:** SOP8 or SOIC8 (8-pin)

**Pinout (Standard SPI):**
```
Pin 1: CS   (Chip Select)
Pin 2: DO   (Data Out)
Pin 3: WP   (Write Protect)
Pin 4: GND  (Ground)
Pin 5: DI   (Data In)
Pin 6: CLK  (Clock)
Pin 7: HOLD (Hold)
Pin 8: VCC  (3.3V)
```

#### Hardware Programmer Setup

**CH341A Programmer:**

!!!danger "CH341A 5V Risk"
    Some CH341A programmers output 5V instead of 3.3V. This can damage the SPI flash chip. Verify voltage before connecting!

**Voltage Mod (if needed):**
1. Locate voltage regulator (AMS1117)
2. Replace with 3.3V variant OR
3. Use voltage divider/mod

**Safer Option:** Use CH347T (native 3.3V support)

**CH347T Programmer (Recommended):**
- Native 3.3V support
- Faster flash speeds
- Better software support
- No voltage mod needed

#### Flashing Process

**Software Options:**
- **flashrom** (Linux, recommended)
- **CH341A Programmer Software** (Windows)
- **AsProgrammer** (Windows)

**Linux flashrom method:**

```bash
# Install flashrom
sudo apt install flashrom  # Debian/Ubuntu
sudo dnf install flashrom  # Fedora
sudo pacman -S flashrom    # Arch

# Connect programmer to PC
# Connect clip to SPI chip on BC-250

# Detect chip
sudo flashrom -p ch341a_spi

# Should detect: MX25L12873F or similar

# Backup current BIOS (important!)
sudo flashrom -p ch341a_spi -r backup.bin

# Verify backup
sudo flashrom -p ch341a_spi -v backup.bin

# Flash new BIOS
sudo flashrom -p ch341a_spi -w P3.00_mod.bin

# Verify flash
sudo flashrom -p ch341a_spi -v P3.00_mod.bin
```

**Connection:**

1. Identify Pin 1 on SPI chip (dot or notch)
2. Align clip carefully (Pin 1 to Pin 1)
3. Ensure good contact (clip fully seated)
4. Connect programmer to USB
5. Flash using software

!!!warning "Power Considerations"
    Some guides suggest powering BC-250 during flash. **This is not recommended**. Flash with board powered off and only programmer power.

#### Troubleshooting Hardware Flash

**Chip not detected:**
- Check clip alignment (Pin 1 correct?)
- Ensure clip fully seated on chip
- Try different USB port
- Check programmer voltage (3.3V required)
- Clean chip pins (isopropyl alcohol)

**Flash verification fails:**
- Re-seat clip
- Try slower flash speed
- Check for poor connection
- Replace clip if damaged

**Board still dead after flash:**
- Verify BIOS file is correct
- Try different BIOS version
- Check for other hardware damage
- May need professional repair

### Method 4: Dual BIOS (If Available)

**Some BC-250 boards may have dual BIOS:**

1. Locate BIOS switch (small switch near BIOS chip)
2. Power off board
3. Toggle switch to backup BIOS
4. Boot from backup BIOS
5. Reflash main BIOS from within working system

**Note:** Not all BC-250 boards have dual BIOS. Check your specific board.

## Prevention

### Before Flashing

**Checklist:**
- [ ] Downloaded correct BIOS file (P3.00 Segfault mod)
- [ ] Verified file hash/checksum
- [ ] Have backup of current BIOS
- [ ] Fully charged laptop/UPS power (no power interruption risk)
- [ ] Read flashing guide completely
- [ ] Have recovery hardware available (just in case)

### Safe Flashing Practices

1. **Never interrupt flash:** Wait for completion
2. **Use correct file:** Verify filename and version
3. **Stable power:** Use UPS or fully charged laptop
4. **Follow instructions:** Don't improvise
5. **Clear CMOS after:** If flashing via USB

### Backup Current BIOS

**Before modifying anything:**

```bash
# If using hardware programmer
sudo flashrom -p ch341a_spi -r backup_$(date +%Y%m%d).bin

# Verify backup
sudo flashrom -p ch341a_spi -v backup_$(date +%Y%m%d).bin

# Store safely (copy to multiple locations)
```

**Via USB flash method:**
- Some flasher tools support backup
- Save before flashing new BIOS

## Recovery Scenario Examples

### Scenario 1: USB Flash, Settings Don't Stick

**Symptoms:**
- Flashed BIOS via USB
- Boot into BIOS, change VRAM to 512MB
- After restart, back to 8GB RAM / 8GB VRAM

**Solution:**
1. Clear CMOS (remove battery or use jumper)
2. Boot into BIOS
3. Set VRAM to 512MB
4. Save and exit
5. Settings should now persist

**Cause:** USB flashing doesn't clear NVRAM, causing conflicts

### Scenario 2: Wrong BIOS File Flashed

**Symptoms:**
- Flashed wrong BIOS
- Board powers on, no display

**Solution:**
1. Acquire hardware programmer (CH347T recommended)
2. Connect clip to SPI flash chip
3. Flash correct BIOS file (BC250_3.00_CHIPSETMENU)
4. Verify flash successful
5. Clear CMOS
6. Boot and configure

### Scenario 3: Power Loss During Flash

**Symptoms:**
- Power cut out during BIOS flash
- Board completely dead

**Solution:**
1. Use hardware programmer
2. Flash known-good BIOS file
3. May need to flash twice (some report first flash partially works)
4. Clear CMOS
5. Boot and reconfigure

### Scenario 4: Experimental BIOS Brick

**Symptoms:**
- Tried experimental/custom BIOS
- Board unstable or won't boot

**Solution:**
1. Flash back to known-good BIOS (BC250_3.00_CHIPSETMENU)
2. Use hardware programmer if board won't boot
3. Stick to community-tested BIOS versions

## BIOS File Information

### Recommended BIOS

**BC250_3.00_CHIPSETMENU (Recommended):**
- Unlocks dynamic VRAM allocation
- Exposes chipset menu and overclocking options
- Most tested and stable
- Available from [bc250-bios repository](https://gitlab.com/TuxThePenguin0/bc250-bios/)

**File naming:**
- Named `BC250_3.00_CHIPSETMENU.ROM`
- Rename to `Robin5.00` before flashing via USB

### Stock BIOS

**P2.11 (Stock):**
- Original BIOS from mining use
- Limited options
- 8GB/8GB or 12GB/4GB fixed splits only

**P3.00 (Stock):**
- Updated official BIOS
- Still limited compared to modded version

### Verifying BIOS Files

**Check file hash:**

```bash
# Linux
sha256sum P3.00_mod.bin

# Windows (PowerShell)
Get-FileHash P3.00_mod.bin -Algorithm SHA256

# Compare with known-good hash from community
```

**File size:**
- Typical BIOS file: 8-16 MB
- If file is wrong size, it's likely corrupted

## Community Resources

### Where to Get Help

1. **BC-250 Discord Server**
   - #bios-help channel
   - Experienced users can guide recovery

2. **GitHub Repositories**
   - BC-250 documentation repos
   - BIOS files and flasher tools

3. **Reddit**
   - r/BC250 (if exists)
   - r/sffpc (general SFF PC help)

### Hardware Programmer Sources

**CH347T Programmer:**
- Available on AliExpress and Amazon

**CH341A Programmer:**
- Available on AliExpress and Amazon
- **Verify 3.3V output before use**

**Test Clips:**
- SOIC8 test clip
- SOP8 test clip
- Get both if unsure of chip package

## Emergency Contact

If all recovery methods fail:

1. **Post in Discord** with details:
   - What you did (exact steps)
   - What BIOS file you used
   - Current symptoms
   - Photos of board (if helpful)

2. **Professional Repair:**
   - Some users offer paid BIOS recovery services
   - Check community for recommendations

3. **Replace BIOS Chip:**
   - Last resort: desolder and replace SPI flash chip
   - Requires soldering skills
   - Pre-programmed chips available

## Success Recovery Stories

**Common recovery scenarios that worked:**

1. **CMOS clear after USB flash:** 90% success rate
2. **Hardware reprogram with CH347T:** 95% success rate
3. **Reflash same file twice:** Sometimes needed
4. **Try different programmer:** Some programmers work better

## See Also

- [BIOS Flashing Guide](flashing.md)
- [BIOS VRAM Configuration](vram.md)
- [Troubleshooting Display Issues](../troubleshooting/display.md)
- [Hardware Specifications](../hardware/specifications.md)
