# Arch Linux and Manjaro Setup Guide

<img src="https://archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png" alt="Arch Linux" height="60"/>

Both Arch Linux and Manjaro work excellently on the BC-250. Arch provides maximum control and latest packages, while Manjaro offers easier installation with good hardware detection.

**Status:** Both fully working
**Difficulty:** Arch (Advanced), Manjaro (Intermediate)
**Mesa:** 25.1+ in official repos
**Kernel:** 6.12-6.14 LTS recommended

---

## Why Choose Arch/Manjaro?

### Advantages

- **Rolling release** - Latest Mesa and kernel versions
- **Native BC-250 support** - Mesa 25.1+ in official repos (no COPR/PPAs)
- **Low RAM usage** - 1.3GB typical for Arch
- **Full control** - Configure everything
- **Community scripts** - Automated setup available

### Arch vs Manjaro

**Arch Linux:**
- Manual installation following the Arch Wiki
- Latest packages immediately
- Maximum control, minimal bloat
- Requires advanced Linux knowledge

**Manjaro:**
- GUI installer (Calamares)
- Boots out-of-box on BC-250 (no nomodeset)
- Delayed package updates (tested before release)
- Easier for beginners

**Performance:** Nearly identical. Both use ~1.3-1.5GB RAM.

---

## BIOS Requirements

**REQUIRED before installing:**
1. Flash modified BIOS (P3.00 recommended)
2. Set VRAM allocation (512MB dynamic recommended)
3. Configure fan speeds
4. **Disable IOMMU** (IOMMU is broken - MUST disable)

See [BIOS Flashing Guide](../bios/flashing.md).

---

## Arch Linux Installation

### Prerequisites

- USB drive (4GB+)
- Ethernet connection (for initial setup)
- Passive DP-to-HDMI adapter (recommended)
- Arch Linux ISO from [archlinux.org](https://archlinux.org/download/)
- Familiarity with Linux command line

### Installation Steps

!!!info "Follow the Arch Wiki"
    Arch Linux installation should be done following the official [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide). This ensures you understand the system you're building and can troubleshoot issues.

**Key BC-250 Specific Requirements:**

1. **Kernel Selection**
   - Install `linux-lts` package (6.12.x - 6.14.x)
   - **AVOID:** Kernel 6.15.0-6.15.6 and 6.17.8+ (GPU initialization failures)
   - Use 6.15.7-6.17.7 for best performance or 6.12-6.14 LTS for stability

2. **Boot Parameters**
   - If black screen during installation, add `nomodeset` to kernel parameters
   - **IMPORTANT:** Remove `nomodeset` after drivers are installed

3. **Required Packages During Installation**
   ```bash
   # Base system
   pacstrap -K /mnt base linux-lts linux-firmware

   # Desktop environment (example with KDE)
   pacstrap -K /mnt plasma-meta kde-applications-meta

   # Graphics drivers (DO NOT install during archinstall, install after reboot)
   # mesa vulkan-radeon xf86-video-amdgpu

   # Network and basic utilities
   pacstrap -K /mnt networkmanager git base-devel
   ```

4. **Swap Configuration**
   - Traditional swap partition recommended
   - **WARNING:** Do NOT use ZRAM if using 512MB dynamic VRAM (conflicts)

5. **Bootloader**
   - GRUB recommended for BC-250
   - Configure with ability to edit boot parameters

6. **Enable Multilib** (for 32-bit support and Steam)
   - Edit `/etc/pacman.conf` before `pacstrap`
   - Uncomment `[multilib]` section

**After Installation - First Boot:**

1. If black screen, use `nomodeset`:
   - At GRUB, press `e`
   - Add `nomodeset` to kernel line
   - Press Ctrl+X to boot

2. Install Mesa and drivers (see Post-Installation section below)

---

## Manjaro Installation

### Installation Steps

1. **Download Manjaro ISO**
   - Get KDE or GNOME edition from [manjaro.org](https://manjaro.org/download/)
   - Flash to USB with balenaEtcher

2. **Boot and Install**
   - Boot from USB (should work without nomodeset)
   - Run Calamares installer
   - Follow on-screen instructions
   - Complete installation and reboot

3. **First Boot**
   - Update system:
     ```bash
     sudo pacman -Syu
     ```

**Community note:** "Out of the box after the BIOS flash, Manjaro KDE just booted fine"

---

## Post-Installation Optimization

After installation, run the automated setup script to configure BC-250 specifics.

### Automated Setup Script (Recommended)

**For Arch:**
```bash
git clone https://github.com/eabarriosTGC/BC250--ARCH.git
cd BC250--ARCH
sudo chmod +x ./Arch-setup.sh
sudo ./Arch-setup.sh
```

**For Manjaro:**
```bash
git clone https://github.com/eabarriosTGC/BC250--ARCH.git
cd BC250--ARCH
sudo chmod +x ./bc520-manjaro.sh
sudo ./bc520-manjaro.sh
```

### What the Script Does

1. **Package Installation**
   - Installs base-devel, git, cmake, lm_sensors
   - Installs build tools for Oberon governor

2. **RADV Environment Configuration**
   - Creates `/etc/environment.d/99-radv-bc250.conf`:
     ```bash
     RADV_DEBUG=nocompute
     ```
   - Disables broken compute queue (prevents glitches)

3. **AMD GPU Kernel Module**
   - Creates `/etc/modprobe.d/amdgpu-bc250.conf`:
     ```bash
     options amdgpu sg_display=0
     ```
   - Only required for kernels < 6.10, safe to keep

4. **Temperature Sensors**
   - Loads nct6683 module
   - Creates `/etc/modules-load.d/nct6683-bc250.conf`
   - Enables temperature monitoring

5. **Oberon Governor**
   - Clones and compiles governor
   - Enables dynamic frequency scaling (1000MHz-2000MHz+)
   - Without this, GPU is locked at 1500MHz

6. **Initramfs Regeneration**
   - Rebuilds initial RAM filesystem with new modules

---

## Kernel Parameters

After running the script, update GRUB with additional parameters.

```bash
sudo nano /etc/default/grub
```

Find `GRUB_CMDLINE_LINUX_DEFAULT` and update:

**Basic:**
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0"
```

**With performance boost:**
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0 mitigations=off"
```

**Remove nomodeset** (if added during install):
```
# WRONG:
GRUB_CMDLINE_LINUX_DEFAULT="quiet nomodeset amdgpu.sg_display=0"

# CORRECT:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0"
```

Update GRUB:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

---

## Verification

### Check Mesa Version

```bash
glxinfo | grep "OpenGL version"
# Expected: Mesa 25.1.X
```

### Check Vulkan Driver

```bash
vulkaninfo | grep "driverName"
# Expected: driverName = radv
```

### Check GPU

```bash
lspci | grep VGA
# Expected: AMD/ATI Device

vulkaninfo | grep deviceName
# Expected: AMD Radeon Graphics (RADV GFX1013)
```

### Install Utilities

```bash
sudo pacman -S mesa-utils vulkan-tools fastfetch nvtop htop

# Test
fastfetch  # Shows system info with GPU
nvtop      # Real-time GPU monitoring
```

**If you see llvmpipe:**
- Mesa drivers not working
- Check Mesa version (must be 25.1+)
- Check kernel parameters applied
- Check dmesg: `dmesg | grep amdgpu`

---

## GPU Governor

### Verify Governor Running

```bash
systemctl status oberon-governor
# Expected: active (running)
```

### Check Frequency Scaling

```bash
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Example output:
# 0: 1000MHz
# 1: 1500MHz *
# 2: 2000MHz
```

The `*` moves between frequencies based on load.

### Known Issue: Governor Not Working on Boot

**Symptom:** GPU stuck at 1500MHz until you run a game once

**Affected:** Arch, Manjaro, CachyOS (NOT Fedora/Bazzite)

**Workaround:** Launch any game/benchmark once after boot to activate

**Manual start:**
```bash
sudo systemctl restart oberon-governor
```

---

## Additional Configuration

### Temperature Sensors

```bash
sensors

# Expected output:
# nct6683-isa-0a20
# GPU Temp: +45.0°C
# SoC Temp: +42.0°C
# Fan1: 1800 RPM
# Fan2: 1800 RPM
```

**If not showing:**
```bash
lsmod | grep nct6683
sudo modprobe nct6683
dmesg | grep nct6683
```

### Fan Control (Optional)

For fan control (nct6683 is read-only), use nct6687:

```bash
echo 'nct6687' | sudo tee /etc/modules-load.d/nct6687.conf
sudo mkinitcpio -P
sudo reboot

# Install CoolerControl for GUI
yay -S coolercontrol
```

### Gaming Tools

```bash
# Steam and Proton
sudo pacman -S steam

# Performance overlays and optimization
sudo pacman -S mangohud goverlay gamemode gamescope

# Proton GE manager
sudo pacman -S protonup-qt
```

---

## Troubleshooting

### Black Screen on Boot

**Solution 1: Use nomodeset**
- At GRUB, press `e`
- Add `nomodeset` to kernel line
- Boot and reinstall Mesa
- Remove nomodeset and add proper parameters

**Solution 2: Check kernel**
```bash
uname -r
# If 6.15.0-6.15.6 or 6.17.8+, install working kernel (6.15.7-6.17.7 or 6.12-6.14 LTS)
```

### GPU Not Detected / llvmpipe

```bash
# Check Mesa
pacman -Q mesa  # Should be 25.1.X+

# Update if needed
sudo pacman -Syu

# Check amdgpu module
lsmod | grep amdgpu

# Check for errors
dmesg | grep amdgpu
```

### Audio Problems (Pitched Down / Slowed)

**Symptom:** Audio sounds like robot, video playback slowed

**Cause:** BC-250 DisplayPort audio implementation issue

**Solution:** Use passive DP-to-HDMI adapter

**Alternative:** USB audio adapter

### Screen Freezing (Broken Kernel Versions)

**Symptom:** Random freezes, kernel panics on kernels 6.15.0-6.15.6 or 6.17.8+

**Solution:**
```bash
# Option 1: Install working 6.15.7-6.17.7 range
sudo pacman -S linux  # Check version is in working range
# Or
# Option 2: Install LTS for guaranteed stability
sudo pacman -S linux-lts linux-lts-headers
sudo pacman -R linux

# Update GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot

# Hold kernel updates if needed
# Add to /etc/pacman.conf:
IgnorePkg = linux
```

### KDE/SDDM Crashes with nomodeset

**Symptom:** KDE crashes when nomodeset is enabled

**Solution:**
- Don't use nomodeset if Mesa 25.1+ is installed
- If needed for install, remove before installing KDE

---

## Community Scripts

**Primary repository:**
- [eabarriosTGC/BC250--ARCH](https://github.com/eabarriosTGC/BC250--ARCH)
- [eabarriosTGC/Instalacion-de-Arch](https://github.com/eabarriosTGC/Instalacion-de-Arch-para-la-Placa-BC-250-AMD)

**Alternative script:**
- [pnbarbeito/bc250-arch](https://github.com/pnbarbeito/bc250-arch)

**Governor projects:**
- [Oberon Governor](https://gitlab.com/mothenjoyer69/oberon-governor)
- [Cyan Skillfish Governor](https://github.com/Magnap/cyan-skillfish-governor) (AUR)

---

## Quick Reference

```bash
# System info
fastfetch

# GPU monitoring
nvtop

# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Check governor
systemctl status oberon-governor

# Check temps
sensors

# Update system
sudo pacman -Syu

# Check Mesa
glxinfo | grep "OpenGL version"

# Check Vulkan
vulkaninfo | grep deviceName
```

---

**Related Guides:**
- [Fedora Setup](fedora.md)
- [Bazzite Setup](bazzite.md)
- [CachyOS Setup](cachyos.md)
- [GPU Governor](../system/governor.md)
