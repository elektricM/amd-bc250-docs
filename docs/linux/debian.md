# Debian and PikaOS Setup Guide

<img src="https://cdn.simpleicons.org/debian" alt="Debian" width="48"/>

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
- Mesa 25.1+ only in experimental repos (upstream BC-250 support standard since 25.1)
- More manual configuration needed
- Kernel selection critical

!!!info "Pre-built Debian Image Available"
    A pre-built Debian image exists with kernel 6.18.3, Mesa 26, and GPU patches pre-applied. This significantly reduces setup complexity for new users. Check community resources for access.

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

1. Flash modified BIOS (P3.00 or later recommended. P5.00_clv exists but may cause ReBAR/USB issues; test before relying on it.)
2. Set VRAM allocation (512MB dynamic recommended)
3. Configure fan speeds
4. **Disable IOMMU if experiencing stability issues** (not universal, but can help)

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
# Should show: Mesa 25.1.X or higher (Mesa 26 confirmed working Jan 2026)
```

!!!warning "Mesa 25.1+ Availability on Debian"
    Debian stable/testing & Linux Mint: Mesa 25.1+ may not be available in standard package repositories. Consider using debian-experimental, backports, or compiling from source if your distro is pinned to older versions.

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

**Confirmed working:** 6.18.3+ tested Jan 2026. Current LTS: 6.18.18.

**Important:** Avoid kernel 6.15.0-6.15.6 and 6.17.8–6.17.10 (broken). Use 6.18.18 LTS (recommended), 6.19.x stable, or 6.17.11+.

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

### 5. Install GPU Governor

A GPU governor is required for proper GPU frequency scaling.

**Option 1: Install cyan-skillfish-governor-smu from .deb (recommended)**

The SMU governor is available as a .deb package from [filippor's COPR](https://github.com/filippor/cyan-skillfish-governor). It bypasses kernel patching entirely.

**Option 2: Build oberon-governor from source (legacy)**

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
cat /sys/class/drm/card1/device/pp_dpm_sclk
```

---

### 6. Configure Temperature Sensors

```bash
# Install lm-sensors
sudo apt install lm-sensors
```

For **read-only monitoring** (temperatures, voltages, fan speeds):

```bash
echo 'nct6683' | sudo tee /etc/modules-load.d/nct6683.conf
echo 'options nct6683 force=true' | sudo tee /etc/modprobe.d/sensors.conf
sudo modprobe nct6683 force=true
```

For **PWM fan control**, use the `nct6687` module instead — see the [Sensors Guide](../system/sensors.md) for full instructions.

Verify:

```bash
sensors
# Should show nct6686-isa-0a20 with temperatures and fan speeds
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
# Expected: 6.18.18 LTS (recommended) or 6.17.11+
```

### Check Governor

```bash
# Service status (use whichever you installed)
systemctl status cyan-skillfish-governor-smu  # or oberon-governor

# GPU frequency
cat /sys/class/drm/card1/device/pp_dpm_sclk
# Should show multiple frequencies with * moving
```

### Check Sensors

```bash
sensors

# Expected:
# nct6686-isa-0a20
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

### Kernel Compatibility Issues

**Symptom:** GPU initialization failures, black screens on 6.15.0-6.15.6 or 6.17.8–6.17.10

**Solution:**
- Use 6.18.18 LTS (recommended) or 6.17.11+
- Or use 6.12-6.14 LTS kernels for guaranteed stability
- Avoid 6.15.0-6.15.6 and 6.17.8–6.17.10 (known broken)

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
# Check service (use whichever you installed)
systemctl status oberon-governor          # legacy
systemctl status cyan-skillfish-governor-smu  # SMU variant

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
- **GPU Governor:** [cyan-skillfish-governor](https://github.com/filippor/cyan-skillfish-governor) (recommended) or [oberon-governor](https://gitlab.com/mothenjoyer69/oberon-governor) (legacy)

---

## Quick Reference

```bash
# Update system
sudo apt update && sudo apt upgrade

# Check Mesa
glxinfo | grep "OpenGL version"

# Check GPU
vulkaninfo | grep deviceName

# Check governor (use whichever you installed)
systemctl status cyan-skillfish-governor-smu  # or oberon-governor

# Check GPU frequency
cat /sys/class/drm/card1/device/pp_dpm_sclk

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
