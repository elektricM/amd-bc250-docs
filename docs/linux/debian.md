# Debian and PikaOS Setup Guide

Debian and PikaOS offer stable, low-power options for the BC-250. While requiring more setup than other distributions, they provide excellent stability and lower idle power consumption.

**Status:** Works well with some effort
**Difficulty:** Intermediate to Advanced
**Base:** Debian Testing/Sid required (not Stable)
**Power Usage:** Lowest among tested distros

---

## Distribution Options

### Debian Testing/Sid

**Advantages:**
- Rock-solid stability
- Lower power consumption
- Full control over system
- Large package repository

**Considerations:**
- Requires Testing or Sid (Stable too old)
- Mesa 25.1+ only in experimental repos
- More manual configuration needed
- Kernel selection critical

### PikaOS

**Advantages:**
- Debian-based gaming distro
- Mesa 25.1+ out of box
- GPU frequency patch included by default
- Works well with BC-250
- Gaming optimizations pre-configured

**Considerations:**
- Smaller community than mainstream distros
- Based on Ubuntu/Debian packages
- Update schedule less frequent

---

## Why Choose Debian/PikaOS?

**Best for:**
- Users who prioritize stability over bleeding edge
- Lower idle power consumption (~50-60W vs ~70W on other distros)
- Those familiar with Debian ecosystem
- Gaming on PikaOS with less configuration

**Not ideal for:**
- Users wanting latest packages immediately
- Beginners (Fedora/Bazzite easier)
- Those needing bleeding-edge Mesa updates

---

## BIOS Requirements

Before installing, ensure BIOS is configured:

1. Flash modified BIOS (P3.00 recommended)
2. Set VRAM allocation (512MB dynamic recommended)
3. Configure fan speeds
4. Disable IOMMU

See [BIOS Flashing Guide](../bios/flashing.md).

---

## Debian Installation

### Prerequisites

- Debian Testing or Sid ISO (not Stable)
- USB drive (4GB+)
- Ethernet connection recommended
- Passive DP-to-HDMI adapter

### Installation Steps

1. **Download Debian Testing ISO**
   - Get from [debian.org](https://www.debian.org/devel/debian-installer/)
   - Choose "testing" installer

2. **Create Bootable USB**
   - Use balenaEtcher or dd

3. **Boot and Install**
   - May need `nomodeset` kernel parameter initially
   - Complete standard Debian installation
   - Choose desktop environment (GNOME or KDE)

4. **First Boot**
   - Boot with `nomodeset` if needed
   - Update system before continuing

---

## Post-Installation Setup

### 1. Add Experimental Repository

Mesa 25.1+ is only in experimental repos.

```bash
# Edit sources list
sudo nano /etc/apt/sources.list

# Add experimental repo
deb http://deb.debian.org/debian experimental main contrib non-free non-free-firmware
```

Create pin preferences to prevent unwanted upgrades:

```bash
sudo nano /etc/apt/preferences.d/experimental

# Add:
Package: *
Pin: release a=experimental
Pin-Priority: 1

Package: mesa-vulkan-drivers libgl1-mesa-dri
Pin: release a=experimental
Pin-Priority: 500
```

Update package lists:

```bash
sudo apt update
```

---

### 2. Install Mesa 25.1+

```bash
sudo apt install -t experimental mesa-vulkan-drivers libgl1-mesa-dri
```

Verify installation:

```bash
glxinfo | grep "OpenGL version"
# Should show: Mesa 25.1.X or higher
```

---

### 3. Install Kernel

**Option 1: Debian 6.12 LTS (Recommended)**

```bash
sudo apt install linux-image-6.12
```

**Option 2: Xanmod (Better Performance)**

```bash
# Add Xanmod repository
wget -qO - https://dl.xanmod.org/archive.key | sudo gpg --dearmor -o /usr/share/keyrings/xanmod-archive-keyring.gpg

echo 'deb [signed-by=/usr/share/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-kernel.list

sudo apt update

# Install Xanmod LTS
sudo apt install linux-xanmod-lts-x64v3
```

**Confirmed working:** 6.14.11 Xanmod kernel

**Important:** Avoid kernel 6.15+, stick to 6.12-6.14 range.

---

### 4. Configure Kernel Parameters

```bash
sudo nano /etc/default/grub

# Find GRUB_CMDLINE_LINUX_DEFAULT and update:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0"

# Save and update GRUB
sudo update-grub
```

Remove `nomodeset` if you added it during installation (after Mesa is installed).

---

### 5. Install Oberon Governor

Governor is required for proper GPU frequency scaling.

```bash
# Install dependencies
sudo apt install build-essential cmake git libdrm-dev libyaml-cpp-dev

# Clone and build
git clone https://gitlab.com/mothenjoyer69/oberon-governor.git
cd oberon-governor
cmake .
make -j$(nproc)
sudo make install

# Create systemd service
sudo nano /etc/systemd/system/oberon-governor.service
```

Add the following content:

```ini
[Unit]
Description=Oberon GPU Governor
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/local/bin/oberon-governor
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now oberon-governor.service
```

Verify:

```bash
systemctl status oberon-governor
cat /sys/class/drm/card0/device/pp_dpm_sclk
```

---

### 6. Configure Temperature Sensors

```bash
# Install lm-sensors
sudo apt install lm-sensors

# Load nct6687 module
echo 'nct6687' | sudo tee /etc/modules-load.d/nct6687.conf

# Load module now
sudo modprobe nct6687

# Verify
sensors
```

---

### 7. Install Gaming Tools

```bash
# Steam
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install steam

# MangoHud
sudo apt install mangohud

# GameMode
sudo apt install gamemode
```

---

## PikaOS Installation

PikaOS is a Debian-based gaming distro with BC-250 optimizations included.

### Installation Steps

1. **Download PikaOS ISO**
   - Get from [pikaos.org](https://pikaos.org) or GitHub releases
   - Choose KDE or GNOME edition

2. **Install Normally**
   - Flash ISO to USB
   - Boot and install (should work without nomodeset)
   - Complete installation

3. **Update System**
   ```bash
   sudo apt update && sudo apt upgrade
   ```

4. **Verify GPU Support**
   ```bash
   # Check Mesa version
   glxinfo | grep "OpenGL version"
   # Should show Mesa 25.1+

   # Check Vulkan
   vulkaninfo | grep deviceName
   # Should show: AMD Radeon Graphics (RADV GFX1013)
   ```

### PikaOS Benefits

- Mesa 25.1+ included by default
- GPU frequency patch pre-applied to kernel
- Governor support built-in (may need to enable)
- Gaming tools pre-installed
- Less configuration needed than vanilla Debian

---

## Verification

### Check Installation

```bash
# Mesa version
glxinfo | grep "OpenGL version"
# Expected: Mesa 25.1.X+

# Vulkan driver
vulkaninfo | grep "driverName"
# Expected: driverName = radv

# GPU detection
lspci | grep VGA
# Expected: AMD/ATI device

vulkaninfo | grep deviceName
# Expected: AMD Radeon Graphics (RADV GFX1013)

# Kernel version
uname -r
# Expected: 6.12.x or 6.14.x (avoid 6.15+)
```

### Check Governor

```bash
# Service status
systemctl status oberon-governor

# GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk
# Should show multiple frequencies with * moving
```

### Check Sensors

```bash
sensors

# Expected:
# nct6687-isa-0a20
# GPU Temp: XX°C
# Fan speeds
```

---

## Known Issues

### Mesa Too Old

**Symptom:** vulkaninfo shows llvmpipe instead of radv

**Solution:**
- Ensure you're on Debian Testing/Sid (not Stable)
- Install from experimental repository
- Verify with `apt policy mesa-vulkan-drivers`

### Kernel 6.15+ Issues

**Symptom:** GPU initialization failures, black screens

**Solution:**
- Use 6.12-6.14 LTS kernels
- Avoid 6.15+ until BC-250 support confirmed

### Audio Issues

**Symptom:** Pitched down audio, slowed video playback

**Cause:** BC-250 DisplayPort audio implementation

**Solution:**
- Use passive DP-to-HDMI adapter
- Or use USB audio adapter

---

## Power Consumption Benefits

Debian/PikaOS users report lower idle power consumption:

- **Debian:** ~50-60W idle
- **Other distros:** ~70W idle
- **Under load:** Similar across all distros (~150-235W)

This makes Debian ideal for:
- Always-on servers
- HTPC use cases
- Power-conscious users

---

## Package Management

### Update System

```bash
sudo apt update
sudo apt upgrade
```

### Install Software

```bash
# From standard repos
sudo apt install <package>

# From experimental
sudo apt install -t experimental <package>
```

### Hold Packages

To prevent unwanted upgrades:

```bash
sudo apt-mark hold linux-image-6.12-amd64
```

---

## Troubleshooting

### Black Screen on Boot

**Solution:**
1. Add `nomodeset` to kernel parameters at GRUB
2. Boot, install Mesa 25.1+
3. Remove `nomodeset` from /etc/default/grub
4. Run `sudo update-grub`
5. Reboot

### GPU Not Detected

```bash
# Check Mesa version
apt policy mesa-vulkan-drivers

# Should show installed from experimental
# If not, reinstall:
sudo apt install -t experimental mesa-vulkan-drivers libgl1-mesa-dri --reinstall
```

### Governor Not Working

```bash
# Check service
systemctl status oberon-governor

# Check logs
journalctl -u oberon-governor -f

# Restart service
sudo systemctl restart oberon-governor
```

---

## Performance Tuning

### Disable Mitigations (Optional)

For ~5-10% performance boost:

```bash
sudo nano /etc/default/grub

# Add mitigations=off:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0 mitigations=off"

sudo update-grub
sudo reboot
```

**Warning:** Disables CPU security mitigations. Only use if you understand the implications.

### Install Performance Tools

```bash
# nvtop for GPU monitoring
sudo apt install nvtop

# htop for system monitoring
sudo apt install htop

# CoolerControl for fan management (from GitHub releases)
```

---

## Community Resources

- **Debian:** [debian.org](https://www.debian.org/)
- **PikaOS:** [pikaos.org](https://pikaos.org)
- **Xanmod Kernel:** [xanmod.org](https://xanmod.org/)
- **Oberon Governor:** [GitLab](https://gitlab.com/mothenjoyer69/oberon-governor)

---

## Quick Reference

```bash
# Update system
sudo apt update && sudo apt upgrade

# Check Mesa
glxinfo | grep "OpenGL version"

# Check GPU
vulkaninfo | grep deviceName

# Check governor
systemctl status oberon-governor

# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Check temps
sensors

# Update GRUB
sudo update-grub
```

---

**Related Guides:**
- [Fedora Setup](fedora.md)
- [Arch Linux Setup](arch.md)
- [Bazzite Setup](bazzite.md)
- [GPU Governor](../system/governor.md)
