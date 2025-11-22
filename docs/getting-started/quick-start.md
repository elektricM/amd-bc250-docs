# Quick Start Guide

Get your BC-250 up and running with this fast-track checklist.

!!!warning "Requirements First"
    Before starting, ensure you have everything from the [Prerequisites](prerequisites.md) page.

## Setup Checklist

### Step 1: BIOS Flash (Critical)

!!!danger "Must Do First"
    The modded BIOS unlocks essential features like dynamic VRAM allocation and proper fan control. Skip this and you'll have a bad time.

1. Download modded BIOS from [bc250-bios repository](https://gitlab.com/TuxThePenguin0/bc250-bios/)
2. Format USB stick as FAT32
3. Rename BIOS file to `robin5.00`
4. Copy to USB root directory
5. Use included flashing utility
6. **CRITICAL:** Clear CMOS after flashing (remove battery 30 seconds)

**Difficulty:** Easy

[Full BIOS guide →](../bios/flashing.md)

### Step 2: BIOS Configuration

Boot into BIOS (Del key during startup) and configure:

- **VRAM Split:** 512MB (Dynamic)
- **Fan Control:** Full Speed (for testing) or Customize
- **IOMMU:** **Disabled** (MUST disable - IOMMU is broken)
- **Boot Mode:** UEFI

[VRAM configuration guide →](../bios/vram.md)

### Step 3: Install Linux

!!!tip "Recommended: Fedora or Bazzite"
    Fedora 42/43 and Bazzite have the best out-of-box support. Other distros work but need more manual setup.

**Fedora Installation:**

1. Download Fedora 42 or 43 Workstation
2. Boot installer in "Basic Graphics Mode" (enables nomodeset automatically)
3. Complete installation normally
4. Reboot

**Difficulty:** Easy

[Distribution comparison →](../linux/distributions.md)

### Step 4: Install Drivers & Governor

Run the automated setup script:

```bash
# For Fedora 42/43
# Mesa 25.1+ is included in Fedora 43 repos - no additional setup needed
sudo dnf update

# Install governor from COPR
sudo dnf copr enable @exotic-soc/oberon-governor
sudo dnf install oberon-governor
sudo systemctl enable --now oberon-governor.service
```

```bash
# For Bazzite
curl -s https://raw.githubusercontent.com/vietsman/bc250-documentation/refs/heads/main/oberon-setup.sh | sudo sh
```

This installs:

- Mesa 25.1+ drivers (already in Fedora 43+ / Bazzite)
- Oberon GPU governor (required for performance)
- Sensor drivers (lm-sensors package)
- System optimizations

**Difficulty:** Easy

### Step 5: Remove nomodeset

!!!warning "Critical Step"
    After drivers are installed, you MUST remove nomodeset or the GPU won't work properly.

```bash
sudo nano /etc/default/grub

# Find this line:
GRUB_CMDLINE_LINUX_DEFAULT="quiet nomodeset"

# Change to:
GRUB_CMDLINE_LINUX_DEFAULT="quiet"

# Save and update GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Reboot
sudo reboot
```

### Step 6: Verify Installation

Check that everything works:

```bash
# Check Mesa version (should be 25.1+)
glxinfo | grep "OpenGL version"

# Check GPU detected
vulkaninfo | grep deviceName
# Should show: AMD Radeon Graphics (RADV GFX1013)

# Check governor running
systemctl status oberon-governor
# Should show: active (running)

# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk
# Should show multiple frequencies, one marked with *
```

### Step 7: Install Steam & Gaming Tools

```bash
sudo dnf install steam mangohud goverlay

# Enable Steam Proton for Windows games
# In Steam: Settings → Compatibility → Enable Proton for all titles
```

### Step 8: Test a Game!

Launch a game through Steam. Most games work out of the box with Proton.

For launch options in Steam (right-click game → Properties → Launch Options):

```bash
RADV_DEBUG=nohiz %command%
```

This fixes graphical glitches in some games.

---

## Quick Troubleshooting

### No Display

**Problem:** Black screen during/after installation
**Solution:** Boot with nomodeset parameter (added automatically in Fedora Basic Graphics Mode)

### GPU Not Detected

**Problem:** `vulkaninfo` shows llvmpipe instead of AMD GPU
**Solution:**

1. Verify Mesa 25.1+ installed: `dnf list mesa-*`
2. Check kernel version: `uname -r` (should be 6.15.7-6.17.7 or 6.12-6.14 LTS, NOT 6.15.0-6.15.6 or 6.17.8+)
3. Verify nomodeset was removed from GRUB

### Poor Performance / Low FPS

**Problem:** Games running at 15-20 FPS
**Solution:**

1. Check governor is running: `systemctl status oberon-governor`
2. Check GPU frequency: `cat /sys/class/drm/card0/device/pp_dpm_sclk`
3. Should NOT be stuck at 1500MHz

### High Temperatures

**Problem:** GPU hitting 90°C+
**Solution:**

1. Verify fans are spinning at full speed
2. Straighten heatsink fins (they're often bent)
3. Replace thermal paste
4. Use high static pressure fans (Arctic P12 recommended)

[Full troubleshooting guide →](../troubleshooting/display.md)

---

## Performance Targets

You should achieve:

- **Idle:** 40-60°C, 50-80W power draw
- **Gaming:** 70-85°C, 150-200W power draw
- **FPS:** 60+ in most games at 1080p medium-high settings

## Next Steps

Now that your BC-250 is running:

1. **Optimize cooling:** [Cooling guide](../hardware/cooling.md)
2. **Tune performance:** [GPU governor configuration](../system/governor.md)
3. **Test games:** [Game compatibility list](../gaming/compatibility.md)
4. **Join community:** Discord link in GitHub repositories
