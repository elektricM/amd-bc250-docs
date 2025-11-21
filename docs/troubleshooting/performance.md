# Performance Troubleshooting

This guide covers common performance issues and their solutions for the BC-250.

---

## Quick Diagnostics

Before troubleshooting specific issues, run these commands to check your system status:

```bash
# Check GPU frequency and temperature
sensors

# Check GPU utilization and frequency
watch -n 1 cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/gpu_busy_percent
watch -n 1 cat /sys/class/drm/card0/device/pp_dmu_clock

# Check if GPU driver is loaded
lspci -k | grep -A 3 VGA

# Verify Mesa version
glxinfo | grep "OpenGL version"

# Check governor status (if installed)
systemctl status oberon-governor
# or
systemctl status cyan-skillfish-governor
```

---

## GPU Locked at 1500MHz

**Symptoms:**
- GPU frequency stuck at 1500MHz regardless of load
- Low FPS in games despite acceptable temperatures
- `radeontop` or monitoring tools show constant 1500MHz

**Cause:** The default GPU governor is locked by BIOS. Without a user-space governor, the GPU cannot scale frequency dynamically.

**Solution: Install GPU Governor**

The BC-250 requires a custom GPU governor to enable dynamic frequency scaling between 350-2300MHz (patched kernel) or 1000-2000MHz (unpatched kernel).

### Option 1: Oberon Governor (Recommended for most users)

**Features:**
- Multi-step frequency scaling
- Maintains GPU usage between 45-70%
- Lower CPU overhead (0.4% CPU usage)
- 100ms burst-to-max time

**Installation:**

**Fedora/Bazzite:**
```bash
dnf copr enable @exotic-soc/oberon-governor
dnf install oberon-governor
systemctl enable --now oberon-governor
```

**Arch/Manjaro:**
```bash
yay -S oberon-governor
systemctl enable --now oberon-governor
```

**Configuration:**

Edit `/etc/oberon-config.yaml`:

```yaml
opps:
  - frequency:
    - min: 1000
    - max: 2000
  - voltage:
    - min: 700
    - max: 1000
```

Restart the service:
```bash
sudo systemctl restart oberon-governor
```

**Verify it's working:**
```bash
oberon-governor --help  # Should show v0.1.4 or higher
systemctl status oberon-governor
```

### Option 2: Cyan Skillfish Governor (Advanced users)

**Features:**
- Continuous frequency adjustment (no steps)
- Maintains GPU utilization 70-95% (configurable)
- Higher CPU overhead (0.9-1.3% CPU usage)
- 24ms burst-to-max time
- More responsive to burst loads

**Installation:**

**Fedora/RPM:**
```bash
dnf copr enable filippor/bazzite
dnf install cyan-skillfish-governor
```

**Arch/AUR:**
```bash
yay -S cyan-skillfish-governor
```

**Debian:**
Download `.deb` from [GitHub releases](https://github.com/Magnap/cyan-skillfish-governor/releases)

**Configuration:**

The governor uses voltage/frequency pairs. Edit the config file and add as many stable points as you've tested:

```toml
safe-points = [
    { freq_mhz = 350,  voltage_mv = 570 },
    { freq_mhz = 860,  voltage_mv = 600 },
    { freq_mhz = 1090, voltage_mv = 650 },
    { freq_mhz = 1280, voltage_mv = 700 },
    { freq_mhz = 1460, voltage_mv = 750 },
    { freq_mhz = 1620, voltage_mv = 800 },
    { freq_mhz = 1760, voltage_mv = 850 },
    { freq_mhz = 1890, voltage_mv = 900 },
    { freq_mhz = 2030, voltage_mv = 950 },
    { freq_mhz = 2090, voltage_mv = 975 },
    { freq_mhz = 2140, voltage_mv = 1000 },
    { freq_mhz = 2230, voltage_mv = 1050 },
]

load_target = { min = 70, max = 95 }
```

!!! warning "Test Your Values"
    Default values are conservative. Test stability for your specific board:

    ```bash
    # Stop the governor
    sudo systemctl stop cyan-skillfish-governor

    # Manually set frequency/voltage
    echo vc 0 <CLOCK> <VOLTAGE> > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

    # Run stress test (benchmark, game, etc.)
    # If stable, add to config. If crashes, increase voltage or lower frequency.
    ```

Enable and start:
```bash
sudo systemctl enable --now cyan-skillfish-governor
```

---

## GPU Frequency Not Scaling Above 2000MHz

**Symptoms:**
- Governor installed but GPU won't go above 2000MHz
- Governor config shows max 2300MHz but actual frequency limited

**Cause:** Kernel doesn't have the frequency range patch.

**Solution: Apply Kernel Patch**

The BC-250 needs a kernel patch to unlock frequency range from 500-2500MHz (default is 1000-2000MHz).

**Pre-patched Kernels:**

**Bazzite:** Uses patched kernel by default (no action needed)

**CachyOS:**
```bash
# CachyOS has pre-patched kernels in their repo
paru -S linux-cachyos-lts-headers
```

**Fedora/Arch - Manual Patch:**

Download the patch:
```bash
wget https://raw.githubusercontent.com/mothenjoyer69/bc250-documentation/main/kernel-patches/amdgpu-frequency-range.patch
```

Apply to kernel source and recompile, or use a tool like `dkms` or CachyOS kernel manager.

**Verification:**
```bash
cat /sys/class/drm/card0/device/pp_od_clk_voltage
# Should show range up to 2300MHz or higher
```

---

## Software Rendering (llvmpipe) Instead of GPU

**Symptoms:**
- `glxinfo | grep "OpenGL renderer"` shows "llvmpipe"
- Games extremely slow (5-10 FPS)
- Steam shows "Software Rendering" in system info

**Diagnostic:**
```bash
glxinfo | grep "OpenGL renderer"
# Bad:  OpenGL renderer string: llvmpipe (LLVM 15.0.0, 256 bits)
# Good: OpenGL renderer string: AMD Radeon Graphics (gfx1013, LLVM 15.0.0, DRM 3.54, 6.12.0)

vulkaninfo --summary
# Should show AMD RADV driver
```

**Cause 1: Mesa Too Old**

Solution: Upgrade to Mesa 25.1.3 or newer

**Fedora 43+:**
```bash
# Mesa 25.1+ is in official repos
sudo dnf update mesa*
```

**Fedora 42:**
```bash
# Use COPR for Mesa 25.1+
sudo dnf copr enable @exotic-soc/bc250-mesa
sudo dnf update mesa*
```

**Arch/Manjaro:**
```bash
# Install from official repos
sudo pacman -S mesa
```

**Verification:**
```bash
glxinfo | grep "OpenGL version"
# Should show Mesa 25.1.3 or higher
```

**Cause 2: Wrong Graphics Adapter Selected**

Some games (especially Red Dead Redemption 2) default to software rendering.

**Solution:**

Check available adapters:
```bash
vulkaninfo --summary
```

In-game, change graphics adapter number to match the GPU (not llvmpipe).

For RDR2 specifically, launch with:
```bash
-useMaximumSettings
```

**Cause 3: Driver Not Loaded**

Check if amdgpu driver is loaded:
```bash
lspci -k | grep -A 3 VGA
# Should show "Kernel driver in use: amdgpu"
```

If not loaded, check:
```bash
dmesg | grep amdgpu
# Look for errors
```

Common fix - ensure these kernel parameters are set (and `nomodeset` is removed):
```bash
# Edit /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.sg_display=0"

# Regenerate grub config
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # Fedora
sudo grub-mkconfig -o /boot/grub/grub.cfg    # Arch
```

---

## Low FPS / Poor Gaming Performance

### Check GPU Utilization

```bash
# Monitor GPU load percentage
watch -n 1 cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/gpu_busy_percent

# If GPU is at 100% constantly = thermal throttling or insufficient performance
# If GPU is at 30-50% = governor not scaling properly, or game/driver issue
# If GPU is at 0-10% = not using GPU (software rendering)
```

### Issue: Governor Not Installed
See "GPU Locked at 1500MHz" section above.

### Issue: Broken Kernel Version

**Symptoms:**
- Previously working setup suddenly has poor performance after kernel update
- Random GPU crashes under load
- System freezes during gaming
- Running kernel 6.15.0-6.15.6 or 6.17.8+

**Solution:**

**Install Working Kernel**

!!! danger "AVOID BROKEN KERNEL VERSIONS"
    Kernel 6.15.0-6.15.6 and 6.17.8+ break GPU driver support. Use 6.15.7-6.17.7 for best performance or 6.12.x-6.14.x LTS for stability.

**Arch/Manjaro:**
```bash
# Option 1: Install working 6.15.7-6.17.7 kernel
sudo pacman -S linux  # Check version is in working range
# Option 2: Install LTS kernel for guaranteed stability
sudo pacman -S linux-lts linux-lts-headers
# Set as default in bootloader
```

**CachyOS:**
```bash
# Check version first
paru -S linux-cachyos  # Ensure it's 6.15.7-6.17.7
# Or for LTS stability:
paru -S linux-cachyos-lts linux-cachyos-lts-headers
```

**Fedora:**
```bash
# Check current kernel
uname -r

# Install older kernel from koji if needed
# Or wait for kernel fixes
```

### Issue: IOMMU Causing Crashes

**Symptoms:**
- Random system crashes under GPU load
- Weird performance issues
- System hangs

**Solution:**

Disable IOMMU in BIOS, or add kernel parameter:
```bash
amd_iommu=off
```

### Issue: RADV_DEBUG Environment Variable

Some older setup guides recommend setting `RADV_DEBUG=nocompute` globally. This may not be needed on Mesa 25.1+.

**Test without it:**
```bash
# Remove from Steam launch options
# Remove from /etc/environment if set there
```

### Issue: Game-Specific Optimizations

**Steam Launch Options:**

Most games work with:
```bash
RADV_DEBUG=nocompute %command%
```

For better performance, try:
```bash
# FSR enabled
WINE_FULLSCREEN_FSR=1 %command%

# Vulkan backend
PROTON_USE_WINED3D=0 %command%

# Gamescope for consistent frametimes
gamescope -W 1920 -H 1080 -f -- %command%
```

---

## Thermal Throttling

**Symptoms:**
- Performance starts good then degrades after 5-15 minutes
- Temperatures above 85-90C
- GPU frequency drops under sustained load

**Check Temperatures:**
```bash
watch -n 1 sensors
```

Safe operating temperatures:
- **70-85C:** Normal under load
- **85-90C:** High but acceptable
- **90C+:** Thermal throttling likely

**Solutions:**

### 1. Improve Cooling

**Arctic P12 Max recommended** (high static pressure: 3.27 mmH2O)

Alternative good options:
- Noctua NF-A12x25 (2.34 mmH2O)
- Arctic P14 PWM (2.4 mmH2O)

Fan configuration:
```bash
# Set fans to full speed in BIOS (for testing)
# Or use fan control software
```

### 2. Repaste / Replace Thermal Pads

- **APU die:** Use quality thermal paste (Arctic MX-4, Noctua NT-H1) or PTM7950 phase-change pad
- **Memory/VRM:** 2mm thermal pads (verify thickness first)
- Ensure heatsink is properly secured

### 3. Undervolt GPU

If thermal throttling persists, try undervolting:

```bash
# Example: 2000MHz at 940mV instead of 1000mV
echo vc 0 2000 940 > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
echo c > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
```

Test stability with benchmarks. If stable, update governor config.

---

## VRAM Allocation Issues

### Checking Current VRAM Split

```bash
# Total system RAM
free -h

# GPU VRAM visible
sudo lshw -C display | grep -i memory
```

### Issue: Game Crashes with "Out of Memory"

**Symptoms:**
- Games crash after loading
- Artifacts then crash (Company of Heroes 3, RDR2)
- "Out of memory" errors

**Common with:** 512MB dynamic allocation + ZRAM enabled

**Solution 1: Use Fixed VRAM Allocation**

Boot into BIOS and change VRAM allocation:
- **For most games:** 4GB VRAM / 12GB RAM
- **For competitive/esports:** 6GB VRAM / 10GB RAM
- **For VRAM-heavy games:** 8GB VRAM / 8GB RAM or 10GB VRAM / 6GB RAM

!!! info "Dynamic Allocation Issues"
    512MB dynamic allocation conflicts with ZRAM on some games. Use fixed allocation instead.

**Solution 2: Disable ZRAM**

```bash
# Disable ZRAM
sudo systemctl stop zram-swap
sudo systemctl disable zram-swap
```

**Solution 3: Increase VRAM Visibility (Advanced)**

For LLM/AI workloads that need more than 12GB VRAM:

Add to kernel command line:
```bash
amdgpu.gttsize=14750 ttm.pages_limit=3776000 ttm.page_pool_size=3776000
```

This allows GPU to allocate up to ~14.75GB VRAM. Limit usage to 14.25-14.5GB in applications to avoid crashes.

---

## Stuttering and Frame Pacing

### Issue: Inconsistent Frame Times

**Symptoms:**
- FPS counter shows 60+ but feels choppy
- Frame time graph shows spikes
- Stuttering during gameplay

**Solution 1: Use Gamescope**

Gamescope provides consistent frame pacing:

```bash
# Install gamescope
sudo dnf install gamescope  # Fedora
sudo pacman -S gamescope    # Arch

# Launch game through gamescope
gamescope -W 1920 -H 1080 -f -- %command%

# With frame limit
gamescope -W 1920 -H 1080 -r 60 -f -- %command%
```

**Solution 2: Disable Compositor (X11)**

For KDE Plasma (X11):
```bash
# Disable compositor in System Settings > Display > Compositor
# Or use keyboard shortcut: Alt+Shift+F12
```

**Solution 3: Use Wayland**

Wayland generally has better frame pacing than X11.

**Solution 4: Audio Configuration**

Some games (especially emulators) have frame pacing tied to audio:

```bash
# Check PulseAudio/PipeWire sample rate
pactl info | grep "Default Sample"

# Lock to 48kHz
# Edit /etc/pulse/daemon.conf (PulseAudio) or PipeWire config
default-sample-rate = 48000
```

For Ryujinx (Switch emulator): Change audio backend in settings can dramatically improve performance.

---

## System Configuration Issues

### Kernel Parameters (Optimal)

Edit `/etc/default/grub`:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.sg_display=0 mitigations=off"
```

**Explanation:**
- `amdgpu.sg_display=0`: Required for kernel < 6.10 (doesn't hurt on newer)
- `mitigations=off`: +18 FPS in Cyberpunk 2077 (60→78 FPS) but reduces security

!!! danger "Security Warning"
    `mitigations=off` disables CPU vulnerability mitigations. Only use if you trust all code running on the system.

Regenerate grub:
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # Fedora
sudo grub-mkconfig -o /boot/grub/grub.cfg    # Arch
```

### Remove nomodeset

!!! danger "Critical: Remove nomodeset After Driver Installation"
    `nomodeset` is ONLY for installation. It disables GPU acceleration. Remove it after Mesa is installed.

Check if present:
```bash
cat /etc/default/grub | grep nomodeset
```

If found, remove it and regenerate grub config.

---

## Performance Monitoring Tools

### Real-time Monitoring

**MangoHud (in-game overlay):**
```bash
# Install
sudo dnf install mangohud  # Fedora
sudo pacman -S mangohud    # Arch

# Use in Steam launch options
mangohud %command%

# Or global config in ~/.config/MangoHud/MangoHud.conf
```

**Radeontop (terminal):**
```bash
sudo dnf install radeontop
radeontop
```

**Sensors (temperatures/voltages):**
```bash
watch -n 1 sensors
```

**CoolerControl (GUI):**
```bash
# Fedora/Bazzite
ujust install-coolercontrol

# Provides fan control and sensor monitoring
```

### Benchmarking

**Unigine Superposition:**
```bash
# Good for thermal/stability testing
# 1080p Extreme preset
# Stock BC-250: ~3888 score
# 2.22GHz OC: ~4118 score
```

**vkmark (Vulkan):**
```bash
sudo dnf install vkmark
vkmark
```

**Cyberpunk 2077 Benchmark:**
Popular community test - consistent results:
- Stock: ~57 FPS (1080p high)
- 2.22GHz: ~60 FPS
- With mitigations=off: +15-20 FPS

---

## Distribution-Specific Issues

### Fedora

**Issue: MTG Arena crashes GUI**
- Confirmed issue on Fedora with Gnome
- Works fine on Manjaro
- Try KDE Plasma instead of Gnome

### Bazzite

**Issue: Freeze on Sleep**
- Known issue: system appears frozen when entering sleep
- Solution: Press power button to wake (don't hold, just press)

**Issue: Bazzite Update Breaks System**
- If an update causes issues, rollback:
```bash
rpm-ostree status
rpm-ostree rollback
```

### CachyOS

**Issue: Installation ISO Won't Boot**
- Use LTS kernel ISO
- Build custom ISO with LTS kernel (see guide in System Configuration section)

---

## Quick Performance Checklist

Use this checklist to verify your system is properly configured:

- [ ] Mesa version 25.1.3 or higher
- [ ] Kernel 6.15.7-6.17.7 or 6.12.x-6.14.x LTS (NOT 6.15.0-6.15.6 or 6.17.8+)
- [ ] GPU governor installed and running (oberon or cyan-skillfish)
- [ ] `nomodeset` removed from kernel parameters
- [ ] BIOS flashed to P3.00 with 512MB dynamic or 4-12GB fixed VRAM
- [ ] `glxinfo` shows RADV driver, not llvmpipe
- [ ] Temperatures under 85C under load
- [ ] Cooling with high static pressure fan (>2.0 mmH2O)
- [ ] IOMMU disabled in BIOS
- [ ] `systemctl status oberon-governor` shows active

**Quick test:**
```bash
# This should show GPU scaling dynamically
watch -n 0.5 'cat /sys/class/drm/card0/device/pp_dmu_clock && cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/gpu_busy_percent'

# Run a game or benchmark
# Frequency should scale from ~1000MHz idle to 2000+MHz under load
# GPU utilization should be 80-100% in demanding scenes
```

---

## Getting Help

If you're still experiencing performance issues after following this guide:

1. **Gather system information:**
```bash
# Create a system report
uname -r                    # Kernel version
glxinfo | grep -i mesa      # Mesa version
sensors                     # Temperatures
systemctl status oberon-governor  # Governor status
cat /etc/oberon-config.yaml       # Governor config
dmesg | grep amdgpu | tail -50    # Recent GPU messages
```

2. **Join the Discord community:**
- BC-250 Discord: Largest community, most active support
- Search for your specific issue first - likely already solved

3. **Check GitHub documentation:**
- https://github.com/mothenjoyer69/bc250-documentation
- https://github.com/AMD-BC-250/documentation

---

## Additional Resources

- [BIOS Flashing Guide](../bios/flashing.md)
- [Linux Distribution Setup](../linux/distributions.md)
- [Overclocking Guide](../bios/overclocking.md)
- [Cooling Solutions](../hardware/cooling.md)
