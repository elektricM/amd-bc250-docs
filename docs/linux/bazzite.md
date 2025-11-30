# Bazzite Setup Guide

<img src="https://raw.githubusercontent.com/ublue-os/bazzite/main/repo_content/logo.svg" alt="Bazzite" width="48"/>

Bazzite is a gaming-focused Linux distribution based on Fedora that provides an excellent out-of-the-box experience for the BC-250. Built on Fedora Atomic (immutable system), it offers SteamDeck-like functionality with enhanced stability.

**Status:** Recommended for gaming, works out-of-the-box
**Base:** Fedora Atomic (OSTree-based)
**Mesa Version:** 25.1+ included
**Desktop Options:** GNOME, KDE, or Deck UI

---

## Why Choose Bazzite?

**Advantages:**
- Works out-of-the-box (no nomodeset needed)
- Mesa 25.1+ included with GPU drivers pre-installed
- Immutable system (harder to break, easy rollback)
- Gaming optimizations (GameMode, Gamescope, Proton GE ready)
- Steam Deck UI option for couch gaming
- Automated governor script available
- Custom kernel with BC-250 patches available

**Considerations:**
- Package management more complex (rpm-ostree vs dnf)
- Some apps require Flatpak
- Updates can occasionally cause issues (but easy to rollback)

---

## Prerequisites

### BIOS Configuration

Before installing, ensure your BIOS is properly configured:

1. Flash modified BIOS (P3.00 recommended)
2. Set VRAM allocation to 512MB dynamic
3. Configure fan speeds
4. **Disable IOMMU** (IOMMU is broken - MUST disable)

See [BIOS Flashing Guide](../bios/flashing.md) for details.

### Hardware Requirements

- 300W+ PSU on 12V rail (250W minimum)
- 2x 120mm high static pressure fans
- DisplayPort cable or passive DP-to-HDMI adapter
- USB drive (8GB+) for installation media

---

## Installation

### Creating Installation Media

1. Download Bazzite ISO from [bazzite.gg](https://bazzite.gg)
2. Choose your variant:
   - **Bazzite GNOME** - Recommended for beginners
   - **Bazzite KDE** - Desktop users (note: fixed as of mid-2025)
   - **Bazzite Deck** - Steam Deck UI experience
3. Flash to USB using Fedora Media Writer or balenaEtcher

### Installation Process

1. Boot from USB (no special parameters needed)
2. Complete on-screen installation
   - Select timezone and language
   - Create user account
   - Choose disk partitioning
3. Installation takes 10-15 minutes
4. Reboot when prompted

**Note:** Unlike Fedora, Bazzite boots directly without needing nomodeset parameter.

---

## Standard Setup

### Automated Installation (Recommended)

One-command installation of Oberon governor:

```bash
curl -s https://raw.githubusercontent.com/vietsman/bc250-documentation/refs/heads/main/oberon-setup.sh | sudo sh
```

**What this script does:**
- Installs Oberon GPU governor
- Enables governor service
- Configures voltage settings (1000mV default for stability)
- No Mesa modifications needed (already included)

**Important:** Run on fresh install for best results. Script auto-reboots when complete.

### Manual Installation

If automated script doesn't work:

```bash
# Add COPR repository
sudo dnf copr enable filippor/bazzite

# Install governor
rpm-ostree install oberon-governor

# Reboot to apply
systemctl reboot

# Enable service after reboot
sudo systemctl enable --now oberon-governor.service
```

### Voltage Configuration

Default configuration (`/etc/oberon-config.yaml`):

```yaml
voltage:
  - min: 1000  # Safe default
  - max: 1000
frequency:
  - min: 1000  # 1000 MHz
  - max: 2000  # 2000 MHz
```

Some boards are unstable at lower voltages. The script defaults to 1000mV to prevent crashes. If system is stable, you can try lowering min voltage to 700-900mV.

---

## Performance Setup (Advanced)

"Bazzite on Steroids" - Custom images with GPU frequency range patch for up to 50% performance boost.

### Features

- Custom patched kernel (GPU frequency: 350-2230 MHz vs stock 1000-2000 MHz)
- Oberon governor pre-configured
- Weekly automated builds (every Monday)
- Three variants: GNOME, KDE, Deck

### Prerequisites

If you already have Bazzite installed with Oberon:

```bash
# Remove existing oberon installation
sudo systemctl stop oberon-governor
sudo systemctl disable oberon-governor
rpm-ostree uninstall oberon-governor

# Remove config
sudo rm -f /etc/oberon-config.yaml
```

### Rebase to Patched Image

Choose your desktop environment:

**GNOME (Recommended):**
```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/vietsman/bazzite-gnome-patched:latest
```

**KDE:**
```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/vietsman/bazzite-kde-patched:latest
```

**Deck:**
```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/vietsman/bazzite-deck-patched:latest
```

After rebase:

```bash
systemctl reboot
systemctl status oberon-governor  # Verify running
```

### Power and Cooling Warnings

Performance patch increases power draw and temperatures:

- **PSU:** Minimum 240W on 12V rail, recommended 320W+
- **Cooling:** High static pressure fans required
- **Temps:** Expect 85-95°C under full load (normal for this board)

To reduce power consumption, edit `/etc/oberon-config.yaml`:

```yaml
voltage:
  - min: 700
  - max: 950  # Reduced from 1000
frequency:
  - max: 1800  # Reduced from 2230
```

Then restart: `sudo systemctl restart oberon-governor`

### Disable CPU Mitigations (Optional)

For additional gaming performance, disable CPU security mitigations:

```bash
rpm-ostree kargs --append-if-missing="mitigations=off"
systemctl reboot
```

!!!warning "Security Trade-off"
    This disables Spectre/Meltdown mitigations for improved performance (+18 FPS in some games). Only recommended for dedicated gaming systems. See [Kernel Configuration](kernel.md#performance-parameters-optional) for details.

---

## Post-Installation Configuration

### Temperature Sensors

Enable NCT6687 module for PWM fan control:

```bash
echo 'nct6687' | sudo tee /etc/modules-load.d/nct6687.conf
systemctl reboot
```

Verify:

```bash
sensors
# Should show nct6687-isa-0a20 with GPU temp, fan speeds
```

### CoolerControl (Optional)

GUI for fan curve management:

```bash
ujust install-coolercontrol
```

### Flatpak Mesa Override (Old Versions Only)

As of August 2025, Bazzite ships with Mesa 25.1+ for Flatpaks. This section only applies to older installations:

```bash
# Add flathub-beta
flatpak remote-add --if-not-exists flathub-beta https://flathub.org/beta-repo/flathub-beta.flatpakrepo

# Install mesa-git for runtime 24.08
flatpak install --system flathub-beta org.freedesktop.Platform.GL.mesa-git//24.08
flatpak install --system flathub-beta org.freedesktop.Platform.GL32.mesa-git//24.08

# Set environment
sudo mkdir -p /etc/systemd/system/service.d
sudo bash -c 'echo -e "[Service]\nEnvironment=FLATPAK_GL_DRIVERS=mesa-git" > /etc/systemd/system/service.d/99-flatpak-mesa-git.conf'

systemctl reboot
```

### System Updates

```bash
# Update everything
ujust update

# Or manually:
rpm-ostree upgrade
flatpak update
```

**Rollback if update breaks:**

```bash
rpm-ostree rollback
systemctl reboot
```

---

## Known Issues & Solutions

### Governor Voltage Instability

**Symptom:** Graphics artifacts, crashes, black screens

**Cause:** Default voltage too low for some boards

**Solution:**

```bash
sudo nano /etc/oberon-config.yaml

# Change:
voltage:
  - min: 1000  # Increase from 700
  - max: 1000

sudo systemctl restart oberon-governor
```

### Flatpak Apps Don't See GPU

**Symptom:** Flatpak games use software rendering (llvmpipe)

**Solution:** See Flatpak Mesa Override section above

### GPU Locked at 1500MHz

**Symptom:** GPU frequency stuck, won't scale

**Solution:**

```bash
# Check governor status
systemctl status oberon-governor

# If not running:
sudo systemctl enable --now oberon-governor.service

# Restart if running:
sudo systemctl restart oberon-governor

# Verify frequency scaling
cat /sys/class/drm/card0/device/pp_dpm_sclk
```

### Boot Slow / Black Screen During Boot

**Symptom:** 30-60 seconds black screen during boot

**Cause:** Normal - display output doesn't initialize until late in boot

**Solution:** Wait - system will boot. Check uptime after boot to confirm it was actually fast.

---

## Desktop Environment Notes

### GNOME (Recommended)

Most tested, fully working with no known issues.

### KDE Plasma

**Historical issue:** Before mid-2025, KDE would crash due to BC-250's faulty RDRAND CPU instruction.

**Current status:** Fixed in recent Qt releases. Works properly now.

### Deck UI

If you rebase from Deck to Desktop image, "Return to Game Mode" won't work. Rebase to Deck-patched image if you want Big Picture mode.

---

## Troubleshooting

### Diagnostic Commands

```bash
# Check Mesa version
rpm -qa | grep mesa

# Check Vulkan device
vulkaninfo | grep deviceName
# Should show: AMD Radeon Graphics (RADV GFX1013)

# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Check temperatures
sensors

# Check OSTree deployment
rpm-ostree status
```

### Performance Issues

```bash
# Verify GPU is being used
vulkaninfo | grep deviceName
# Should NOT show llvmpipe

# Monitor GPU usage
nvtop

# Check governor scaling
watch -n 1 cat /sys/class/drm/card0/device/pp_dpm_sclk
```

---

## Quick Reference

```bash
# Update system
ujust update

# Check governor
systemctl status oberon-governor

# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Check temps
sensors

# Rollback update
rpm-ostree rollback && systemctl reboot

# Rebase to patched GNOME
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/vietsman/bazzite-gnome-patched:latest
```

---

## Community Resources

- **Bazzite Official:** [bazzite.gg](https://bazzite.gg)
- **Setup script:** [vietsman/bc250-documentation](https://github.com/vietsman/bc250-documentation)
- **Patched images:** [vietsman/bazzite-patched](https://github.com/vietsman/bazzite-patched)
- **Oberon governor:** [oberon-governor GitLab](https://gitlab.com/mothenjoyer69/oberon-governor)

---

**Related Guides:**
- [Fedora Setup](fedora.md)
- [CachyOS Setup](cachyos.md)
- [Arch Linux Setup](arch.md)
- [GPU Governor Configuration](../system/governor.md)
