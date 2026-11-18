# Bazzite Setup Guide

<img src="https://raw.githubusercontent.com/ublue-os/bazzite/main/repo_content/logo.svg" alt="Bazzite" width="48"/>

Bazzite is a gaming-focused Linux distribution based on Fedora that provides an excellent out-of-the-box experience for the BC-250. Built on Fedora Atomic (immutable system), it offers SteamDeck-like functionality with enhanced stability.

**Status:** Recommended for gaming, works out-of-the-box
**Base:** Fedora Atomic (OSTree-based)
**Mesa Version:** 25.1+ included (stable `44.20260921` ships Mesa 26.2.2)
**Desktop Options:** GNOME, KDE, or Deck UI

---

!!!tip "Prefer a script?"
    Community toolkits like [NexGen-3D-Printing/SteamMachine](https://github.com/NexGen-3D-Printing/SteamMachine), [samedayhurt/bc250-buddy](https://github.com/samedayhurt/bc250-buddy) and [NeOdYmS/bazzite-bc250-toolkit](https://github.com/NeOdYmS/bazzite-bc250-toolkit) automate the steps below; see [Community Setup Toolkits](distributions.md#community-setup-toolkits).

## Why Choose Bazzite?

**Advantages:**
- Works out-of-the-box (no nomodeset needed)
- Mesa 25.1+ included with GPU drivers pre-installed
- Immutable system (harder to break, easy rollback)
- Gaming optimizations (GameMode, Gamescope, Proton GE ready)
- Steam Deck UI option for couch gaming
- Automated governor script available
- Community images with the governor already built in (see [Prebuilt BC-250 Images](#prebuilt-bc-250-images-optional))

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

# Install governor (SMU variant — recommended, no kernel patch needed)
rpm-ostree install cyan-skillfish-governor-smu

# Reboot to apply
systemctl reboot

# Enable service after reboot
sudo systemctl enable --now cyan-skillfish-governor-smu.service
```

!!!info "TT Governor Alternative"
    The `cyan-skillfish-governor-tt` is also available via the same COPR, but it relies on the kernel frequency range patch, and current Bazzite kernels do not carry it (see [below](#prebuilt-bc-250-images-optional)). On Bazzite, use the SMU variant.

!!!warning "GPU Card Naming Issue"
    The governor may target incorrect device (card0 vs card1). Verify correct device assignment in governor configuration if frequency scaling doesn't work.

### Voltage Configuration

The SMU governor reads `/etc/cyan-skillfish-governor-smu/config.toml`. The format, the frequency range and the voltage curve are explained on the [GPU Governor](../system/governor.md#cyan-skillfish-governor-smu-config-recommended) page; restart the service after editing it.

---

## Prebuilt BC-250 Images (Optional)

!!!warning "Current Bazzite kernels do not include the GPU frequency patch"
    Earlier versions of this page said the standard Bazzite kernel carries the 350-2230 MHz frequency range patch. That is not true of current Bazzite. Since stable `44.20260429` Bazzite ships the Open Gaming Collective kernel, whose `cyan_skillfish_ppt.c` keeps the stock 1000-2000 MHz limits (checked at `v7.2.4-ogc3`, the kernel in stable `44.20260921`, and at `v7.2.7-ogc1` in testing). The SMU governor above does not rely on that patch, so the standard setup is unaffected.

If you would rather not layer the governor yourself, [62fixolab/Latest-Bazzite-AMD-BC-250-Patched-Images](https://github.com/62fixolab/Latest-Bazzite-AMD-BC-250-Patched-Images) publishes Deck, GNOME and KDE images built from the official Bazzite `stable` image plus `cyan-skillfish-governor-smu` from the `filippor/bazzite` COPR, with the service enabled. They carry no custom kernel, so you end up with the same system as the standard setup, with the governor already in the image. They are rebuilt when the upstream Bazzite image changes and are signed with cosign. The same repository publishes `testing` and `unstable` channel images and experimental `-40cu` variants (see [40 CU Unlock](../system/40cu-unlock.md)). It is a one-person community project, not part of Bazzite, so read its README before rebasing.

Rebase to the variant matching the desktop you already run:

```bash
# Deck
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/62fixolab/bazzite-bc250-patched-deck:latest
# GNOME
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/62fixolab/bazzite-bc250-patched-gnome:latest
# KDE
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/62fixolab/bazzite-bc250-patched-kde:latest

systemctl reboot
systemctl status cyan-skillfish-governor-smu  # Verify running
```

`rpm-ostree rollback` takes you back to the previous deployment if the new one misbehaves.

!!!note "Coming from the vietsman images?"
    The `ghcr.io/vietsman/bazzite-*-patched` images this page used to recommend are no longer maintained. Their build workflows have been disabled since the last builds on 2025-11-24, they are pinned to Bazzite 42 (Fedora 42, end of life), and they ship `oberon-governor` rather than the SMU governor. They were also where the USB WiFi breakage in [#10](https://github.com/elektricM/amd-bc250-docs/issues/10) was reported. If you are on one, rebase to stock Bazzite or to one of the images above. 62fixolab's README says to move the old `vietsman` patched-kernel COPR repo file out of `/etc/yum.repos.d` first, otherwise the rebase fails with a 404.

If you installed `oberon-governor` yourself on top of Bazzite, remove it before switching to the SMU governor:

```bash
sudo systemctl stop oberon-governor
sudo systemctl disable oberon-governor
rpm-ostree uninstall oberon-governor

# Remove old config
sudo rm -f /etc/oberon-config.yaml
```

### Power and Cooling

Raising the governor's frequency ceiling raises power draw and temperatures. Check your PSU against [Power Supply Requirements](../hardware/power.md) and your cooling against [Cooling Solutions](../hardware/cooling.md) before you do, and lower the ceiling in the governor config if the board runs hot.

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

### ACPI Fix (CPU C-States and P-States)

The [ACPI fix](../system/governor.md#acpi-fix-installation) enables CPU idle states and frequency scaling. On Bazzite, follow the **rpm-ostree variant** of Step 2 on that page: the plain-Fedora BLS instructions do not work here, because Bazzite's boot entries are managed by ostree.

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

### Box Dead After Sitting Idle

**Symptom:** After a period of inactivity the display is black, no input wakes the box, and it is not reachable over the network. Only a power cut recovers it.

**Cause:** The box auto-suspended, and resume from s2idle does not work reliably on this hardware. See [Black Screen After Idle](../troubleshooting/display.md#problem-black-screen-after-idle-nothing-wakes-it) for the diagnosis and fix. Note that Steam Game Mode has its own idle suspend, so masking the suspend targets matters even if the desktop's power settings already say never suspend.

### Screen Freezes When Loading Levels On Newer Games

**Symptom:** Screen freezes indefinitely when loading in levels on newer games

**Cause:** CMOS clear didn't happen after flashing the BIOS

**Solution:**
- Clear the CMOS
   - Confirm it worked by verifying that the time/clock in the BIOS was reset and shows the wrong value
- Reapply the BIOS settings changes, such as the VRAM allocation

### Consistent Micro-stuttering during Gameplay

**Symptom:** Consistent micro-stutters every couple of seconds even for the least demanding 2D games.

**Cause:** The built-in Bazzite Handheld Daemon fails to load required functionality (not present on BC-250), and inevitably fails and restarts continually.

**Solution:** Disable the Handheld Daemon (HHD).

```bash
sudo systemctl disable --now hhd

# Prevent from being re-enabled in future update
sudo systemctl mask hhd
```

### Governor Voltage Instability

**Symptom:** Graphics artifacts, crashes, black screens

**Cause:** Default voltage too low for some boards

**Solution:**

```bash
sudo nano /etc/cyan-skillfish-governor-smu/config.toml

# Increase voltage if unstable:
# min_voltage = 1000
# max_voltage = 1000

sudo systemctl restart cyan-skillfish-governor-smu
```

### Flatpak Apps Don't See GPU

**Symptom:** Flatpak games use software rendering (llvmpipe)

**Solution:** See Flatpak Mesa Override section above

### GPU Locked at 1500MHz

**Symptom:** GPU frequency stuck, won't scale

**Solution:**

```bash
# Check governor status (use whichever you installed)
systemctl status cyan-skillfish-governor-smu
# Or: systemctl status cyan-skillfish-governor-tt

# If not running:
sudo systemctl enable --now cyan-skillfish-governor-smu.service

# Restart if running:
sudo systemctl restart cyan-skillfish-governor-smu

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

If you rebase from Deck to Desktop image, "Return to Game Mode" won't work. Stay on a Deck image (stock `bazzite-deck` or the Deck variant of the [prebuilt images](#prebuilt-bc-250-images-optional)) if you want Game Mode.

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
systemctl status cyan-skillfish-governor-smu

# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Check temps
sensors

# Rollback update
rpm-ostree rollback && systemctl reboot
```

---

## Community Resources

- **Bazzite Official:** [bazzite.gg](https://bazzite.gg)
- **Prebuilt images:** [62fixolab/Latest-Bazzite-AMD-BC-250-Patched-Images](https://github.com/62fixolab/Latest-Bazzite-AMD-BC-250-Patched-Images)
- **GPU Governor:** [cyan-skillfish-governor-smu](https://github.com/filippor/cyan-skillfish-governor/tree/smu) (recommended) or [cyan-skillfish-governor-tt](https://github.com/filippor/cyan-skillfish-governor) (alternative)

---

**Related Guides:**
- [Fedora Setup](fedora.md)
- [CachyOS Setup](cachyos.md)
- [Arch Linux Setup](arch.md)
- [GPU Governor Configuration](../system/governor.md)
