# Fedora Complete Setup Guide

<img src="https://cdn.simpleicons.org/fedora" alt="Fedora Workstation" width="48"/>

Step-by-step guide to installing and configuring Fedora on the BC-250.

## Overview

Fedora is the most recommended distribution for BC-250, offering:
- Easy installation process
- Mesa 25.1+ in official repositories (Fedora 43)
- Extensive BC-250 community support
- Automated setup scripts available

## Prerequisites

- BC-250 board with BIOS flashed and configured
- USB drive (4GB+) for installation media
- Display connected via DisplayPort
- Keyboard and mouse (USB)
- Internet connection recommended

## Download Fedora

**Recommended Version:** Fedora 43 Workstation

**Download from:** [getfedora.org](https://getfedora.org/workstation/)

**Desktop Options:**
- **GNOME** (default) - Modern, clean interface
- **KDE Plasma** (Fedora Spins) - Highly customizable, recommended by many users

## Create Installation Media

**Using Fedora Media Writer (Recommended):**
1. Download [Fedora Media Writer](https://getfedora.org/fmw)
2. Run and select Fedora Workstation
3. Select your USB drive
4. Click "Write" and wait

**Using balenaEtcher:**
1. Download ISO from Fedora website
2. Download [balenaEtcher](https://www.balena.io/etcher/)
3. Select ISO, select USB drive, flash

## Installation

### Step 1: Boot Installation Media

1. Insert USB drive into BC-250
2. Power on the BC-250
3. System should boot to GRUB menu

!!!warning "Black Screen Issue"
    If you get a black screen, the installer is trying to use the GPU before drivers are loaded.

### Step 2: Select Boot Mode

**For Fedora 42/43 with working kernels (6.15.7-6.17.7):**

You can try the standard "Install Fedora" option. If it boots successfully, no need for basic graphics mode.

**If you get a black screen:**

1. At GRUB menu, select **"Troubleshooting"**
2. Choose **"Install Fedora Workstation in basic graphics mode"**
3. This enables `nomodeset` automatically

!!!info "Nomodeset May Not Be Required"
    On Fedora 42/43 with working kernel versions (6.15.7-6.17.7), nomodeset is often no longer needed during installation. However, if you encounter a black screen, use basic graphics mode.

### Step 3: Complete Installation

1. Select language
2. Choose installation destination (your M.2 SSD)
3. Configure network (optional but recommended)
4. Create user account
5. Set root password (optional)
6. Click "Begin Installation"
7. Wait for installation to complete
8. Click "Reboot System"

**Note:** System will reboot with `nomodeset` still active (limited resolution is normal for now).

## Post-Installation Setup

### Step 1: First Boot and Update

```bash
# Update system
sudo dnf upgrade --refresh
```

### Step 2: Install Dependencies

```bash
sudo dnf install -y git cmake make gcc-c++ libdrm-devel lm_sensors
```

### Step 3: Verify Mesa Version

```bash
# Check Mesa version
dnf list mesa-\*

# Should show 25.1+ for Fedora 43
# If Fedora 42 and < 25.1, may need mesa-git (unlikely now)
```

### Step 4: Install GPU Governor

**Option 1: COPR (Easiest - No Compilation Required)**

COPR repositories provide pre-built packages for Fedora, eliminating the need to compile from source.

```bash
sudo dnf copr enable filippor/bazzite
sudo dnf install oberon-governor
sudo systemctl enable --now oberon-governor.service
```

**Option 2: Build from Source**

```bash
git clone https://gitlab.com/mothenjoyer69/oberon-governor.git
cd oberon-governor
cmake . && make && sudo make install
sudo systemctl enable --now oberon-governor.service
```

### Step 5: Configure Sensors

```bash
# Load sensor module
echo 'nct6683' | sudo tee /etc/modules-load.d/99-sensors.conf
echo 'options nct6683 force=true' | sudo tee /etc/modprobe.d/options-sensors.conf

# Regenerate initramfs
sudo dracut --regenerate-all --force
```

### Step 6: Remove nomodeset and Configure GRUB

```bash
# Edit GRUB configuration
sudo nano /etc/default/grub

# Find: GRUB_CMDLINE_LINUX_DEFAULT="nomodeset quiet"
# Change to: GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0"

# Optional: Add mitigations=off for performance boost
# GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0 mitigations=off"

# Save (Ctrl+O, Enter, Ctrl+X)

# Update GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

!!!info "Kernel Parameters Explained"
    - `quiet` - Reduces boot messages
    - `amdgpu.sg_display=0` - Required for BC-250 GPU (kernel < 6.10)
    - `mitigations=off` - Disables CPU security mitigations (+18 FPS in Cyberpunk 2077)

### Step 7: Reboot

```bash
sudo reboot
```

After reboot, you should have full resolution and GPU acceleration.

## Verification

### Check GPU is Working

```bash
# Check Mesa version
glxinfo | grep "OpenGL version"
# Should show: Mesa 25.1.X

# Check GPU detected
vulkaninfo | grep deviceName
# Should show: AMD Radeon Graphics (RADV GFX1013)

# Check governor running
systemctl status oberon-governor
# Should show: active (running)

# Check sensors
sensors
# Should show GPU temp, fan speeds, etc.
```

## Install Gaming Software

### Steam

```bash
sudo dnf install steam
```

**Enable Proton for Windows games:**
1. Open Steam
2. Settings → Compatibility
3. Check "Enable Steam Play for all other titles"
4. Select Proton version (latest is fine)

### Proton GE (Recommended)

```bash
# Install ProtonUp-Qt
sudo dnf install protonup-qt

# Run ProtonUp-Qt and install latest Proton-GE
```

### Gaming Tools

```bash
# Install useful gaming tools
sudo dnf install mangohud goverlay gamemode gamescope

# MangoHud: FPS overlay
# Goverlay: MangoHud configurator
# Gamemode: CPU governor optimization
# Gamescope: Compositor for better frame pacing
```

## Optional: Hold Kernel Version

Since kernel 6.15.0-6.15.6 and 6.17.8+ break BC-250, you may want to prevent automatic kernel updates to broken versions:

```bash
# Install versionlock plugin
sudo dnf install python3-dnf-plugin-versionlock

# Lock current kernel
sudo dnf versionlock add kernel

# To unlock later:
# sudo dnf versionlock delete kernel
```

## Troubleshooting

### Display Still Not Working After Setup

```bash
# Check amdgpu module loaded
lsmod | grep amdgpu

# Check for errors
dmesg | grep amdgpu

# Verify Mesa
glxinfo | grep -i "opengl renderer"
# Should NOT show "llvmpipe"
```

### Governor Not Starting

```bash
# Check governor service
sudo systemctl status oberon-governor

# Check logs
sudo journalctl -u oberon-governor

# Restart service
sudo systemctl restart oberon-governor
```

### Low FPS in Games

```bash
# Verify GPU is being used (not CPU rendering)
# Run game with MangoHud:
mangohud steam

# Check GPU frequency scaling
cat /sys/class/drm/card0/device/pp_dpm_sclk
# Should show frequencies changing under load
```

## Fedora-Specific Issues

### Kernel Auto-Update to Broken Version

**Symptom:** System breaks after update
**Cause:** Kernel 6.15.0-6.15.6 or 6.17.8+ breaks BC-250

**Solution:**
```bash
# Boot into rescue mode or older kernel
# Remove broken kernel (example for 6.15.5)
sudo dnf remove kernel-6.15.5\*
# Or for 6.17.8+
sudo dnf remove kernel-6.17.8\* kernel-6.17.9\*

# Install working kernel
sudo dnf install kernel-6.16.5-104  # Working 6.15.7-6.17.7 range
# Or LTS for stability
sudo dnf install kernel-6.14.4-104

# Lock kernel version (see above)
```

### MTG Arena Crashes on Fedora

**Symptom:** Magic: The Gathering Arena crashes/freezes
**Workaround:** Some users report better stability on Manjaro or Bazzite
**Possible Fix:** Try different Proton version

## Performance Tuning

### Enable Performance Governor

```bash
# For better gaming performance
sudo cpupower frequency-set -g performance
```

### Optimize for Low Latency

```bash
# Edit /etc/sysctl.conf
sudo nano /etc/sysctl.conf

# Add:
vm.swappiness=10
vm.vfs_cache_pressure=50

# Apply
sudo sysctl -p
```

## Desktop Environment Tips

### KDE Plasma

- Wayland works well on Plasma 6
- Configure compositor for lowest latency:
  - System Settings → Display → Compositor
  - Set latency to "Low" or "Lowest"

### GNOME

- Some users report issues (test carefully)
- Wayland generally stable
- Falls back to X11 if issues

## See Also

- [Kernel Requirements](kernel.md)
- [Mesa Installation](mesa.md)
- [GPU Governor Setup](../system/governor.md)
- [Distribution Comparison](distributions.md)
