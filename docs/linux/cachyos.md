# CachyOS Setup Guide

CachyOS is an Arch-based Linux distribution optimized for performance, featuring the BORE scheduler and CPU-optimized packages. It offers the best performance among tested distributions for the BC-250.

**Status:** Works out-of-box (as of late 2025)
**Difficulty:** Intermediate (Arch-based, but has installer)
**Performance:** Best overall (BORE scheduler, optimized packages)
**Kernel:** Ships with compatible kernels by default

!!! success "Updated November 2025"
    CachyOS now ships with compatible kernels by default. The complex custom ISO build procedures from earlier guides are **no longer needed**. Simply download the standard ISO and install normally.

---

## Why Choose CachyOS?

### Advantages

- **Best gaming performance** - BORE scheduler improves frame times and latency
- **Optimized packages** - x86-64-v3/v4 CPU optimizations for faster execution
- **Mesa 25.1+** included by default
- **Kernel Manager GUI** - Easy kernel management and patching
- **Rolling release** - Latest packages and drivers
- **Active development** - Regular updates and optimizations

### Considerations

- **Rolling release** - Requires occasional maintenance
- **Arch-based** - More advanced than Fedora/Bazzite
- **Package availability** - AUR may be needed for some software
- **Not beginner-friendly** - Recommended for users comfortable with Linux

---

## Installation

### Standard Installation (Recommended)

CachyOS can now be installed directly on the BC-250 using the standard ISO.

**Requirements:**
- 8GB+ USB drive
- Internet connection
- 15-30 minutes

**Steps:**

1. **Download CachyOS ISO:**
   - Visit [cachyos.org](https://cachyos.org/)
   - Download latest ISO (KDE or GNOME edition)

2. **Create bootable USB:**
   ```bash
   # Linux
   sudo dd if=cachyos.iso of=/dev/sdX status=progress conv=sync && sync
   # Replace /dev/sdX with your USB drive (check with lsblk)

   # Or use Ventoy, balenaEtcher, Rufus, etc.
   ```

3. **Boot BC-250 from USB:**
   - Insert USB into BC-250
   - Power on and select USB in boot menu

4. **Run installer:**
   - Launch CachyOS installer from live environment
   - Follow installation wizard:
     - **Partitioning:** Auto or manual (GPT, EFI partition)
     - **Desktop:** KDE Plasma or GNOME
     - **Bootloader:** GRUB (recommended)
     - **Kernel:** Default selection (should be compatible)

5. **Verify kernel version:**
   ```bash
   # After installation completes
   uname -r
   # Should show compatible version (6.12-6.14 LTS or 6.15.7-6.17.7)
   ```

6. **Reboot and enjoy**

!!! warning "If Installation ISO Doesn't Boot"
    If you have issues with the standard ISO (black screen, GPU panic), the installer may have reverted to a broken kernel version. See the [Legacy Installation Method](#legacy-installation-method) below or try [Arch Migration](#arch-migration).

---

### Arch Migration (Alternative Method)

Install Arch Linux first, then migrate to CachyOS repositories for optimized packages.

**Steps:**

1. **Install Arch Linux:**

   Follow the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)

   **Key selections:**
   - Kernel: `linux-lts` (6.12.x - 6.14.x) or `linux` (verify version is 6.15.7-6.17.7)
   - Desktop: KDE Plasma or GNOME
   - Bootloader: GRUB

2. **Boot into Arch Linux**

3. **Run CachyOS migration script:**
   ```bash
   wget https://mirror.cachyos.org/cachyos-repo.tar.xz
   tar xvf cachyos-repo.tar.xz
   cd cachyos-repo
   sudo ./cachyos-repo.sh

   # Select x86-64-v3 optimization (best compatibility for Zen 2)
   # Or x86-64-v4 for maximum performance (verify CPU support)
   ```

4. **Install CachyOS kernel (optional):**
   ```bash
   # For LTS stability
   sudo pacman -S linux-cachyos-lts linux-cachyos-lts-headers

   # Or for latest features (verify version is compatible)
   sudo pacman -S linux-cachyos linux-cachyos-headers

   # Update GRUB
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   sudo reboot
   ```

5. **Verify:**
   ```bash
   uname -r  # Check kernel version
   pacman -Sl cachyos  # Should show CachyOS packages
   ```

---

## Post-Installation Setup

### Install GPU Governor

GPU is locked at 1500MHz without governor. CachyOS has COPR packages available.

**Method 1: Install from COPR (Easiest)**
```bash
# Add CachyOS extra repos if not already enabled
# Install oberon-governor
sudo pacman -S oberon-governor

# Enable and start
sudo systemctl enable --now oberon-governor.service

# Verify
systemctl status oberon-governor
```

**Method 2: Build from source**
```bash
# Install dependencies
sudo pacman -S base-devel cmake git

# Clone and build
git clone https://gitlab.com/mothenjoyer69/oberon-governor.git
cd oberon-governor
cmake .
make -j$(nproc)
sudo make install

# Enable and start
sudo systemctl enable --now oberon-governor.service
```

**Verify it's working:**
```bash
cat /sys/class/drm/card0/device/pp_dpm_sclk
# Should show multiple frequencies, * moves based on load
```

---

### Configure Sensors

```bash
# Install lm_sensors
sudo pacman -S lm_sensors

# Load nct6687 for PWM control
echo 'nct6687' | sudo tee /etc/modules-load.d/nct6687.conf

# Rebuild initramfs
sudo mkinitcpio -P
sudo reboot

# Verify
sensors
```

---

### Kernel Parameters

```bash
# Edit GRUB
sudo nano /etc/default/grub

# For kernels 6.10+, amdgpu.sg_display=0 not needed
GRUB_CMDLINE_LINUX_DEFAULT="quiet"

# With performance boost (disables security mitigations):
GRUB_CMDLINE_LINUX_DEFAULT="quiet mitigations=off"

# Update GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

---

### Install Gaming Essentials

```bash
# Steam and gaming tools
sudo pacman -S steam mangohud goverlay gamemode gamescope

# Monitoring tools
sudo pacman -S nvtop htop fastfetch

# Proton GE
sudo pacman -S protonup-qt
```

---

### Legacy Installation Method

!!! warning "Only If Standard ISO Fails"
    This method is **only needed if** the standard CachyOS ISO doesn't boot (black screen, GPU panic). Most users should use the [Standard Installation](#standard-installation-recommended) above.

This was the original installation method when CachyOS shipped with broken kernel 6.15.0-6.15.6. It may still be useful if CachyOS reverts to incompatible kernel versions.

**Option A: Install on Another PC**
1. Remove BC-250's storage drive
2. Install drive in another PC (use SATA/NVMe adapter if needed)
3. Boot CachyOS ISO and install to BC-250's drive
4. Install compatible kernel before removing:
   ```bash
   sudo pacman -S linux-cachyos-lts linux-cachyos-lts-headers
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```
5. Install drive back in BC-250

**Option B: Custom ISO Build**
1. On another Linux PC:
   ```bash
   sudo pacman -S archiso git
   git clone https://github.com/CachyOS/CachyOS-Live-ISO
   cd CachyOS-Live-ISO
   # Replace standard kernel with LTS
   grep -rl 'linux-cachyos' ./ | xargs sed -i 's/linux\-cachyos/linux\-cachyos\-lts/g'
   # Build ISO (follow repo instructions, takes 20-40 min)
   ```
2. Flash custom ISO to USB and install normally

---

## Performance Optimizations

### BORE Scheduler

Already enabled in CachyOS kernel - no configuration needed. Provides better gaming performance and lower latency compared to standard CFS scheduler.

**Benefits:**
- Improved frame time consistency
- Lower input latency
- Better multi-tasking during gaming

### Optimized Packages

CachyOS repos include CPU-optimized builds of packages.

**During CachyOS migration script:**
- Select **x86-64-v3** for best compatibility (Zen 2 fully supports v3)
- Select **x86-64-v4** for maximum performance (verify your CPU supports AVX-512)

**Performance gains:**
- ~5-10% better FPS in some games vs stock packages
- Faster compilation, compression, encoding
- More responsive desktop

### GPU Frequency Patch

Increases GPU range from 1000-2000MHz to 350-2230MHz. See [GPU Frequency Patch Guide](../bios/gpu-frequency-patch.md) for details.

**Quick install:**
```bash
# Use CachyOS Kernel Manager GUI
cachyos-kernel-manager
# Download and apply BC-250 GPU frequency patch
# Rebuild kernel with patch (8-45 minutes depending on method)
```

---

### Fan Control

```bash
# Install CoolerControl
yay -S coolercontrol

# Enable service
sudo systemctl enable --now coolercontrold

# Launch GUI to set custom fan curves
coolercontrol
```

---

## Verification Checklist

```bash
# 1. Check kernel
uname -r  # Expected: 6.12.x-lts or 6.15.7-6.17.7

# 2. Check Mesa
glxinfo | grep "OpenGL version"  # Expected: Mesa 25.1.x+

# 3. Check GPU
vulkaninfo | grep deviceName  # Expected: AMD Radeon Graphics (RADV GFX1013)

# 4. Check governor
systemctl status oberon-governor  # Expected: active (running)

# 5. Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk  # Expected: Multiple frequencies

# 6. Check sensors
sensors  # Expected: nct6687, GPU temp, fan speeds
```

---

## Known Issues

### Governor Doesn't Start on Boot

**Symptom:** GPU stuck at 1500MHz on boot, works after launching game

**Known issue** on CachyOS/Arch (works fine on Fedora)

**Workaround:** Launch any game once to activate, works normally afterward

---

### Kernel Compatibility

**Compatible kernels:**
- **6.12.x - 6.14.x LTS** - Most stable, recommended
- **6.15.7 - 6.17.7** - Works well, newer features

**Broken kernels (avoid):**
- **6.15.0 - 6.15.6** - GPU initialization failures, kernel panics
- **6.17.8+** - GPU driver issues

If you accidentally install a broken kernel:
```bash
# Install working kernel
sudo pacman -S linux-cachyos-lts linux-cachyos-lts-headers
# Or
sudo pacman -S linux-lts linux-lts-headers

# Update GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

---

## Why Choose CachyOS for BC-250?

**Performance:**
- **BORE scheduler** improves frame times and reduces latency
- **Optimized packages** (x86-64-v3/v4) provide 5-10% better FPS in some games
- **Latest Mesa** and kernel updates for best GPU performance
- More responsive desktop feel compared to other distros

**Advanced Features:**
- **Kernel Manager GUI** for easy kernel patching (including BC-250 GPU frequency patch)
- **Rolling release** always has latest drivers and features
- **Arch-based** access to AUR packages and Arch Wiki

**Trade-offs:**
- More complex than Fedora/Bazzite (Arch-based)
- Rolling release requires occasional maintenance
- Better suited for users comfortable with Linux

---

## Summary

CachyOS now works well on BC-250 with standard installation. The complex custom ISO builds from earlier guides are **no longer needed** in most cases. Simply download the standard ISO, install normally, and enjoy the performance benefits of BORE scheduler and optimized packages.

**Quick Start:**
1. Download CachyOS ISO from [cachyos.org](https://cachyos.org/)
2. Install normally (follow installer wizard)
3. Verify compatible kernel is installed (6.12-6.14 LTS or 6.15.7-6.17.7)
4. Install oberon-governor for GPU frequency scaling
5. Configure sensors, install gaming tools, enjoy

---

## Community Resources

- **CachyOS Website:** [cachyos.org](https://cachyos.org/)
- **CachyOS Wiki:** [wiki.cachyos.org](https://wiki.cachyos.org/)
- **CachyOS GitHub:** [github.com/CachyOS](https://github.com/CachyOS)
- **Oberon Governor:** [GitLab](https://gitlab.com/mothenjoyer69/oberon-governor)

---

**Related Guides:**
- [Arch Linux Setup](arch.md) - For manual Arch installation
- [Fedora Setup](fedora.md) - Easier alternative for beginners
- [Bazzite Setup](bazzite.md) - Gaming-focused alternative
- [GPU Governor](../system/governor.md) - Essential for BC-250 performance
