# Quick Reference Guide

Fast answers to common questions. For detailed information, see the full documentation pages.

---

## Quick Start Checklist

### Hardware Setup

- [ ] BC-250 board
- [ ] 300W+ 12V PSU with PCIe 8-pin
- [ ] High static pressure 120mm fan (Arctic P12/P14)
- [ ] DisplayPort cable or passive DP-HDMI adapter
- [ ] USB stick (FAT32) for BIOS flash
- [ ] M.2 NVMe SSD (optional but recommended)
- [ ] USB WiFi adapter (board has no wireless)

### Software Setup

1. **Flash BIOS** - Get modded BIOS from [GitLab](https://gitlab.com/TuxThePenguin0/bc250-bios/)
2. **Set VRAM to 512MB** in BIOS after flashing
3. **Clear CMOS** (critical after USB flash)
4. **Install Fedora/Bazzite** with "Basic Graphics Mode"
5. **Run setup script** to install Mesa 25.1+ and governor
6. **Remove nomodeset** from GRUB after drivers installed
7. **Test a game!**

[Full quick start guide →](../getting-started/quick-start.md)

---

## Critical Settings

### BIOS Configuration

| Setting | Value | Why |
|---------|-------|-----|
| UMA Frame Buffer Size | 512MB | Dynamic VRAM allocation |
| IOMMU | **MUST be Disabled** | IOMMU is broken - causes display failures and crashes |
| Fan Control | Customize | Stock is too aggressive or too quiet |

[BIOS guide →](../bios/flashing.md)

### Kernel Requirements

- **Use:** Kernel 6.18.18 LTS (recommended), 6.17.11+, or 6.12.x-6.14.x LTS (stable)
- **Avoid:** Kernel 6.15.0-6.15.6, 6.17.8-6.17.10 (GPU fails)
- **Boot parameter:** `nomodeset` during install, remove after drivers installed

### Kernel Parameters

```bash
# Standard in /etc/default/grub GRUB_CMDLINE_LINUX_DEFAULT:
quiet

# Optional performance boost (+18 FPS in some games):
mitigations=off

# For older kernels < 6.10 only (not needed on 6.10+):
amdgpu.sg_display=0
```

---

## Power & Cooling

### Power Draw

| State | Consumption |
|-------|-------------|
| Idle | 50-80W |
| Light gaming | 100-150W |
| Heavy gaming | 150-200W |
| Maximum (RT) | Up to 235W |

### Temperature Targets

| State | Temperature | Status |
|-------|-------------|--------|
| Idle | 40-60°C | ✅ Good |
| Gaming | 70-85°C | ✅ Good |
| Stress | 85-90°C | ⚠️ Caution |
| Critical | >90°C | ❌ Too hot |

### Recommended Fans

1. **Arctic P12 Max** - 6.9 mmH2O, best value
2. **Noctua NF-A12x25** - Quieter, premium
3. **Arctic P14 Max** - 140mm option

!!!warning "High Static Pressure Required"
    You NEED fans with 3+ mmH2O static pressure. Regular case fans won't work.

[Cooling guide →](../hardware/cooling.md)

---

## Software Requirements

### Operating System

**Recommended:**
- Fedora 43 Workstation (easiest)
- Bazzite (best for gaming)
- CachyOS (best performance, harder setup)

**Also works:**
- Manjaro, Arch, Debian (with effort)

**Doesn't work:**
- Windows (no GPU drivers)

**Note:** SteamOS now works after Mesa updates

### Driver Versions

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Mesa | 25.1.0 | 25.3.x+ (25.3.6 on Fedora 43) |
| Kernel | 6.12.x | 6.18.18 LTS or 6.19.x stable |
| Governor | Any | cyan-skillfish-governor-tt or -smu |

[Linux setup guide →](../linux/distributions.md)

---

## VRAM Configuration

### 512MB Dynamic (Recommended)

- Automatically allocates between CPU/GPU
- ~10-15GB CPU RAM, up to 12GB VRAM when needed
- Best for most users

### Fixed Allocations

- **10GB/6GB** - Best for AAA gaming, fixes ZRAM conflicts
- **8GB/8GB** - Balanced, good for AI/compute
- **12GB/4GB** - Light gaming, maximum system RAM

!!!info "ZRAM Conflict"
    If RDR2 or Company of Heroes 3 crashes, switch from 512MB dynamic to 10GB/6GB fixed.

[VRAM guide →](../bios/vram.md)

---

## Display Connection

### Working Solutions

| Method | Resolution | Audio | Notes |
|--------|------------|-------|-------|
| Native DP | Up to 4K60 | ✅ | Best option |
| Passive DP-HDMI | Up to 1440p60 | ✅ | Most common |
| Active DP-HDMI | 4K60+ | ❌ | Video only, no audio |
| USB DAC | N/A | ✅ | Workaround for audio |

!!!danger "Active Adapters Break Audio"
    Active (powered) DP-to-HDMI adapters don't pass audio properly. Use passive adapters.

---

## GPU Governor

The governor controls GPU frequency and voltage. **Required for gaming performance.**

### Installation

```bash
# Fedora/Bazzite (recommended):
sudo dnf copr enable filippor/bazzite
sudo dnf install cyan-skillfish-governor-tt
sudo systemctl enable --now cyan-skillfish-governor-tt.service

# Bazzite (automated script):
curl -s https://raw.githubusercontent.com/vietsman/bc250-documentation/refs/heads/main/oberon-setup.sh | sudo sh

# Arch/CachyOS (SMU - no kernel patch needed):
yay -S cyan-skillfish-governor-smu
sudo systemctl enable --now cyan-skillfish-governor-smu.service
```

### Configuration

Edit `/etc/cyan-skillfish-governor-tt/config.toml`:

```toml
# Safe starting point:
min_frequency = 1000  # MHz
max_frequency = 2000  # MHz
min_voltage = 700     # mV (hard minimum, don't go lower)
max_voltage = 1050    # mV
```

Restart governor after changes:

```bash
sudo systemctl restart cyan-skillfish-governor-tt
```

### Check GPU Frequency

!!!note "GPU Card Number"
    The BC-250 GPU is typically `card1` (not `card0`). Verify with: `ls /sys/class/drm/ | grep "^card"`. All sysfs paths in this documentation use `card1`.

```bash
cat /sys/class/drm/card1/device/pp_dpm_sclk
# Should show multiple frequencies, current one marked with *
```

Without governor, GPU is stuck at 1500MHz = poor performance.

[Governor guide →](../system/governor.md)

---

## Common Commands

### System Info

```bash
# Check Mesa version
glxinfo | grep "OpenGL version"

# Check GPU
lspci | grep VGA
vulkaninfo | grep deviceName

# Check RAM/VRAM split
free -h
cat /sys/class/drm/card1/device/mem_info_vram_total

# Check temperatures
sensors

# Monitor GPU
nvtop
```

### GRUB Configuration

```bash
# Edit boot parameters
sudo nano /etc/default/grub

# Update GRUB (Fedora/RHEL):
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Update GRUB (Debian/Ubuntu):
sudo update-grub

# Reboot
sudo reboot
```

### Governor Management

```bash
# Check status (use whichever you installed)
systemctl status cyan-skillfish-governor-tt
# Or: systemctl status oberon-governor

# Start/stop
sudo systemctl start cyan-skillfish-governor-tt
sudo systemctl stop cyan-skillfish-governor-tt

# Restart after config change
sudo systemctl restart cyan-skillfish-governor-tt

# View logs
journalctl -u cyan-skillfish-governor-tt -f
```

---

## Gaming Performance

### Launch Options (Steam)

For most games:
```
RADV_DEBUG=nohiz %command%
```

For games with visual glitches (Mesa < 25.1 only — `nocompute` is automatic on 25.1+):
```
RADV_DEBUG=nohiz %command%
```

### Expected FPS (1080p)

| Game | Settings | FPS Range |
|------|----------|-----------|
| Cyberpunk 2077 | High, FSR | 60-90 |
| Control | High, RT off | 60+ |
| Control | Medium, RT on | 30-40 |
| DMC 5 | High | 100+ |
| CS2 | High | 120+ |
| Elden Ring | High | 60 (capped) |

[Game compatibility list →](../gaming/compatibility.md)

---

## Troubleshooting Quick Fixes

### No Display

**Solution:** Boot with `nomodeset` parameter

```bash
# At GRUB, press 'e'
# Add 'nomodeset' to linux line
# Press Ctrl+X to boot
```

### GPU Not Detected

**Check:**
1. Mesa version ≥ 25.1: `glxinfo | grep Mesa`
2. Kernel ≤ 6.14: `uname -r`
3. nomodeset removed from GRUB
4. Governor running: `systemctl status cyan-skillfish-governor-tt`

### BIOS Settings Don't Stick

**Solution:** Clear CMOS properly

1. Power off, unplug
2. Remove CMOS battery for 60 seconds
3. While battery out, press power button 5 times
4. Replace battery, boot, reconfigure

### Game Crashes

**Check:**
1. VRAM allocation (try 10GB/6GB fixed if using 512MB dynamic with ZRAM)
2. Disable ZRAM: `sudo systemctl disable zram-swap`
3. Update Mesa to latest
4. Check kernel version (use 6.18.18 LTS, 6.17.11+, or 6.12-6.14 LTS)

### High Temperatures

**Fix:**
1. Check fans spinning: `sensors`
2. Straighten heatsink fins
3. Replace thermal paste
4. Verify high static pressure fans (3+ mmH2O)

[Full troubleshooting →](../troubleshooting/display.md)

---

## Important Warnings

!!!danger "Critical Warnings"
    1. **Always clear CMOS after USB BIOS flash**
    2. **Disable IOMMU in BIOS** (IOMMU is broken - MUST disable)
    3. **Use nomodeset during install, remove after drivers installed**
    4. **Avoid kernel 6.15.0-6.15.6, 6.17.8-6.17.10** (GPU driver fails)
    5. **700mV minimum voltage** (GPU locks to 1500MHz below this)
    6. **Active DP-HDMI adapters break audio**
    7. **ACPI fix recommended** — SSDT tables enable CPU C-States (idle power) and P-States (frequency scaling 800-3200 MHz). Confirmed working on kernel 6.19.8. ([bc250-acpi-fix](https://github.com/bc250-collective/bc250-acpi-fix))
    8. **No HW video encode/decode** — VCN firmware blocked by Sony, software decoding only
    9. **Do NOT use Smokeless_UMAF** — may cause permanent damage to the board

---

## Key Community Resources

### Official Documentation
- **GitHub:** https://github.com/mothenjoyer69/bc250-documentation
- **BIOS Repo:** https://gitlab.com/TuxThePenguin0/bc250-bios/
- **Governor:** Multiple forks (Oberon, Cyan Skillfish)

### Community
- **Discord:** Active community, link in GitHub
- **TheRetroWeb:** https://theretroweb.com/motherboards/s/amd-bc-250

### Key Contributors
- mothenjoyer69 - Setup scripts
- Average Data Hoarder - Modded BIOS
- Segfault - Oberon Governor
- FilippoR - COPR packages, Bazzite integration

---

**Need more detail?** See the full documentation sections:
- [Getting Started](../getting-started/introduction.md)
- [BIOS Flashing](../bios/flashing.md)
- [Linux Setup](../linux/distributions.md)
- [Troubleshooting](../troubleshooting/display.md)
