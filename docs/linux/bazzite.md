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
- 120mm high static pressure fan (Arctic P12 recommended)
- DisplayPort cable or passive DP-to-HDMI adapter
- USB drive (8GB+) for installation media

!!!info "Backplate VRAM Cooling Recommended"
    The VRAM chips on the backplate have no temperature sensor. Ensure airflow over the backplate for gaming workloads. Many builds work fine with basic case airflow — a dedicated backplate fan is ideal but not strictly required if your case has decent airflow.

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

### Governor Installation

```bash
# Add COPR repository
sudo dnf copr enable filippor/bazzite

# Install governor (use cyan-skillfish-governor-tt)
rpm-ostree install cyan-skillfish-governor-tt

# Reboot to apply
systemctl reboot

# Enable service after reboot
sudo systemctl enable --now cyan-skillfish-governor-tt.service
```

!!!info "SMU Governor Alternative (No Kernel Patch Needed)"
    The `cyan-skillfish-governor-smu` bypasses kernel patches entirely via SMU firmware calls. Available on AUR, COPR (`filippor/bazzite`), .deb, .rpm, and Nix.

!!!warning "GPU Card Naming Issue"
    The governor may target incorrect device (card0 vs card1). Verify correct device assignment in governor configuration if frequency scaling doesn't work.

### Voltage Configuration

Default configuration (`/etc/cyan-skillfish-governor-tt/config.toml`):

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

!!!success "Bazzite Kernel Already Includes GPU Frequency Patch"
    As of early 2026, the standard Bazzite kernel already includes the GPU frequency range patch. **Manual kernel patching is not needed on Bazzite.** The "Bazzite on Steroids" custom images below are only needed if you want additional pre-configured optimizations.

    Alternatively, the **SMU governor** (`cyan-skillfish-governor-smu`) bypasses the need for kernel patches entirely on any distro.

"Bazzite on Steroids" - Custom images with additional pre-configured optimizations.

### Features

- Custom patched kernel (GPU frequency: 350-2230 MHz vs stock 1000-2000 MHz)
- GPU governor pre-configured
- Weekly automated builds (every Monday)
- Three variants: GNOME, KDE, Deck

### Prerequisites

If you already have Bazzite installed with an older governor:

```bash
# Remove existing oberon installation (if applicable)
sudo systemctl stop oberon-governor
sudo systemctl disable oberon-governor
rpm-ostree uninstall oberon-governor

# Remove old config
sudo rm -f /etc/oberon-config.yaml
```

!!!danger "USB WiFi Drivers May Be Removed (Issue #10)"
    Rebasing to patched images may remove USB WiFi/Bluetooth drivers that are not included in the custom kernel build. If you rely on a USB WiFi adapter (the BC-250 has no built-in wireless), **verify your adapter's driver is included in the patched image before rebasing**. If WiFi stops working after rebase:

    1. Connect via Ethernet temporarily
    2. Check if your WiFi adapter's kernel module is available: `lsmod | grep <your_driver>`
    3. Install missing drivers: `rpm-ostree install <driver-package>`
    4. Or rollback: `rpm-ostree rollback && systemctl reboot`

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
systemctl status cyan-skillfish-governor-tt  # Verify running
```

!!!warning "WiFi May Be Killed by Performance Setup (Issue #10)"
    Users have reported that rebasing to the performance/patched image can kill WiFi drivers. This is likely caused by the kernel swap or driver module changes during the rebase. If you lose WiFi after rebasing, you may need to reinstall WiFi driver modules or roll back with `rpm-ostree rollback`.

### Power and Cooling Warnings

Performance patch increases power draw and temperatures:

- **PSU:** Minimum 240W on 12V rail, recommended 320W+
- **Cooling:** High static pressure fans required
- **Temps:** Expect 85-95°C under full load (normal for this board)

To reduce power consumption, edit `/etc/cyan-skillfish-governor-tt/config.toml`:

```yaml
voltage:
  - min: 700
  - max: 950  # Reduced from 1000
frequency:
  - max: 1800  # Reduced from 2230
```

Then restart: `sudo systemctl restart cyan-skillfish-governor-tt`

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

For **read-only monitoring** (temperatures, voltages, fan speeds):

```bash
echo 'nct6683' | sudo tee /etc/modules-load.d/nct6683.conf
echo 'options nct6683 force=true' | sudo tee /etc/modprobe.d/sensors.conf
systemctl reboot
```

For **PWM fan control**, use the `nct6687` module instead — see the [Sensors Guide](../system/sensors.md) for full instructions.

Verify:

```bash
sensors
# Should show nct6686-isa-0a20 with temperatures and fan speeds
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

### Screen Freezes When Loading Levels On Newer Games

**Symptom:** Screen freezes indefinitely when loading in levels on newer games

**Cause:** CMOS clear didn't happen after flashing the BIOS

**Solution:**
- Clear the CMOS
   - Confirm it worked by verifying that the time/clock in the BIOS was reset and shows the wrong value
- Reapply the BIOS settings changes, such as the VRAM allocation

### Governor Voltage Instability

**Symptom:** Graphics artifacts, crashes, black screens

**Cause:** Default voltage too low for some boards

**Solution:**

```bash
sudo nano /etc/cyan-skillfish-governor-tt/config.toml

# Increase voltage if unstable:
# min_voltage = 1000
# max_voltage = 1000

sudo systemctl restart cyan-skillfish-governor-tt
```

### Flatpak Apps Don't See GPU

**Symptom:** Flatpak games use software rendering (llvmpipe)

**Solution:** See Flatpak Mesa Override section above

### GPU Locked at 1500MHz

**Symptom:** GPU frequency stuck, won't scale

**Solution:**

```bash
# Check governor status (use whichever you installed)
systemctl status cyan-skillfish-governor-tt
# Or: systemctl status oberon-governor

# If not running:
sudo systemctl enable --now cyan-skillfish-governor-tt.service

# Restart if running:
sudo systemctl restart cyan-skillfish-governor-tt

# Verify frequency scaling
cat /sys/class/drm/card1/device/pp_dpm_sclk
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
cat /sys/class/drm/card1/device/pp_dpm_sclk

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
watch -n 1 cat /sys/class/drm/card1/device/pp_dpm_sclk
```

---

## Quick Reference

```bash
# Update system
ujust update

# Check governor
systemctl status cyan-skillfish-governor-tt

# Check GPU frequency
cat /sys/class/drm/card1/device/pp_dpm_sclk

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
- **GPU Governor:** [cyan-skillfish-governor-tt](https://github.com/filippor/cyan-skillfish-governor) (recommended) or [oberon-governor](https://gitlab.com/mothenjoyer69/oberon-governor) (legacy)

---

**Related Guides:**
- [Fedora Setup](fedora.md)
- [CachyOS Setup](cachyos.md)
- [Arch Linux Setup](arch.md)
- [GPU Governor Configuration](../system/governor.md)
