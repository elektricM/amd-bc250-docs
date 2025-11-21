# CachyOS Setup Guide

CachyOS is an Arch-based Linux distribution optimized for performance, featuring the BORE scheduler and CPU-optimized packages. While it offers the best performance among tested distributions, installation on the BC-250 requires special steps due to kernel compatibility.

**Status:** Works well with custom installation
**Difficulty:** Advanced (requires custom ISO or migration)
**Performance:** Best overall (BORE scheduler, optimized packages)
**Kernel:** 6.15.7-6.17.7 (recommended) or 6.12-6.14 LTS (stable)

---

## Why Choose CachyOS?

### Advantages

- **Best gaming performance** - BORE scheduler improves frame times
- **Optimized packages** - x86-64-v3/v4 CPU optimizations
- **Mesa 25.1+** included by default
- **Kernel Manager GUI** - Easy kernel management and patching
- **Rolling release** - Latest packages
- **Active development** - Regular updates

### Considerations

- **Cannot install directly** - Standard ISO doesn't work on BC-250
- **Requires custom build** - Must build LTS ISO or migrate from Arch
- **Kernel 6.15.0-6.15.6 and 6.17.8+** cause panics (use 6.15.7-6.17.7 or 6.12-6.14 LTS)
- **Advanced setup** - Not recommended for Linux beginners

---

## Installation Methods

Choose based on your experience level and available resources.

### Method 1: Custom ISO Build (Recommended)

**Updated Status (August 2025):** CachyOS switched to LTS kernel by default in their ISO, so this method may no longer be necessary. Try the standard ISO first.

**Requirements:**
- Another PC or laptop (any Linux distro)
- 8GB+ USB drive
- Internet connection
- 30-60 minutes

**Steps:**

1. **Prepare build environment** (on another PC):
   ```bash
   sudo pacman -S archiso git
   ```

2. **Clone CachyOS Live ISO repository:**
   ```bash
   git clone https://github.com/CachyOS/CachyOS-Live-ISO
   cd CachyOS-Live-ISO
   ```

3. **Replace standard kernel with LTS:**
   ```bash
   grep -rl 'linux-cachyos' ./ | xargs sed -i 's/linux\-cachyos/linux\-cachyos\-lts/g'
   ```

4. **Build ISO:**
   ```bash
   # Follow build instructions from repository
   # Build takes 20-40 minutes
   ```

5. **Flash ISO to USB:**
   ```bash
   sudo dd if=/path/to/cachyos.iso of=/dev/sdX status=progress conv=sync && sync
   # Replace /dev/sdX with your USB drive (check with lsblk)
   ```

6. **Install CachyOS on BC-250:**
   - Boot custom ISO
   - Go through installation normally
   - Choose GRUB as bootloader
   - **DO NOT reboot when installation finishes**

7. **Post-installation LTS kernel setup:**
   ```bash
   # After installation, open terminal
   sudo mount /dev/nvme0n1p2 /mnt
   sudo mount /dev/nvme0n1p1 /mnt/boot/efi

   # Chroot into system
   sudo chroot /mnt

   # Install LTS kernel
   paru -S linux-cachyos-lts linux-cachyos-lts-headers

   # Exit and unmount
   exit
   sudo umount -R /mnt

   # Now reboot
   sudo reboot
   ```

8. **Verify LTS kernel:**
   ```bash
   uname -r
   # Should show: 6.12.x-lts
   ```

---

### Method 2: Install on Another PC (Easiest)

Bypass BC-250 compatibility issues entirely.

**Requirements:**
- BC-250 SSD/NVMe drive
- Another PC with SATA/NVMe slot or USB adapter
- Standard CachyOS ISO

**Steps:**

1. Remove storage drive from BC-250
2. Install in another PC:
   - Connect BC-250's drive to another PC
   - Boot CachyOS ISO
   - Install CachyOS to the BC-250 drive
3. Switch to LTS kernel:
   ```bash
   # Method A: CachyOS Kernel Manager (GUI)
   cachyos-kernel-manager
   # Select "linux-cachyos-lts" and install

   # Method B: Command line
   sudo pacman -S linux-cachyos-lts linux-cachyos-lts-headers
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```
4. Install drive back into BC-250 and boot

---

### Method 3: Arch Migration (Most Control)

Install Arch Linux first, then migrate to CachyOS repositories.

**Steps:**

1. **Install Arch Linux:**
   ```bash
   # Use archinstall for easier setup
   archinstall

   # Select:
   # - Kernel: linux-lts
   # - Desktop: KDE Plasma or GNOME
   # - Bootloader: GRUB
   ```

2. **Boot Arch** (may need nomodeset on first boot)

3. **Run CachyOS migration script:**
   ```bash
   wget https://mirror.cachyos.org/cachyos-repo.tar.xz
   tar xvf cachyos-repo.tar.xz
   cd cachyos-repo
   sudo ./cachyos-repo.sh

   # Select x86-64-v3 optimization
   ```

4. **Install CachyOS LTS kernel:**
   ```bash
   sudo pacman -S linux-cachyos-lts linux-cachyos-lts-headers
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   sudo reboot
   ```

5. **Verify:**
   ```bash
   uname -r  # Should show 6.12.x-1-lts
   ```

---

## Post-Installation Setup

### Install Oberon Governor

GPU is locked at 1500MHz without governor.

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

# Verify
systemctl status oberon-governor
```

**Check it's working:**

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

# Basic configuration:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0"

# With performance boost:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0 mitigations=off"

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

## Performance Optimizations

### BORE Scheduler

Already enabled in CachyOS kernel - no configuration needed. Provides better gaming performance and lower latency.

### Optimized Packages

```bash
# During repo setup, select v3 for best compatibility
# v4 is faster but requires newer CPUs (Zen2 supports v3)
```

### GPU Frequency Patch

Increases GPU range from 1000-2000MHz to 350-2230MHz.

**Method 1: CachyOS Kernel Manager**
- Download BC-250 GPU frequency patch
- Use Kernel Manager GUI to apply
- Rename from .mypatch to .patch if needed

**Method 2: Manual compilation**
- Place patch in kernel source
- Compile with CachyOS optimizations
- ~8 minutes with modprobed-db
- ~45 minutes full compile

---

### Fan Control

```bash
# Install CoolerControl
yay -S coolercontrol

# Enable service
sudo systemctl enable --now coolercontrold

# Set custom fan curves
coolercontrol
```

---

## Verification Checklist

```bash
# 1. Check kernel
uname -r  # Expected: 6.12.x-1-lts

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

### Broken Kernel Versions Cause Panics

**Symptom:** Black screen, kernel panic, GPU initialization failure on 6.15.0-6.15.6 or 6.17.8+

**Solution:** Use working kernel versions (6.15.7-6.17.7 or 6.12-6.14 LTS)

```bash
# If on broken version, install working kernel:
sudo pacman -S linux-cachyos  # Check version is 6.15.7-6.17.7
# Or
sudo pacman -S linux-lts  # For 6.12-6.14 LTS stability
```

**Note:** Kernels 6.15.7-6.17.7 work well. Avoid 6.15.0-6.15.6 and 6.17.8+

---

### Standard ISO Boots to Black Screen

**Symptom:** CachyOS standard ISO shows black screen

**Cause:** Non-LTS kernel doesn't support BC-250

**Solution:** Use custom LTS ISO build (Method 1)

---

## Why CachyOS for Performance?

**BORE Scheduler Benefits:**
- Better frame time consistency
- Lower input latency
- Improved multi-tasking during gaming

**Optimized Packages:**
- x86-64-v3 builds use AVX2 instructions
- Faster execution across system
- Particularly noticeable in compilation, compression

**Performance Comparison:**
- ~5-10% better FPS vs Fedora in some games
- More responsive desktop feel
- Faster package operations

**Trade-off:**
- Harder setup vs Fedora/Bazzite
- More maintenance (rolling release)
- Requires kernel knowledge

---

## Community Resources

- **CachyOS Website:** [cachyos.org](https://cachyos.org/)
- **CachyOS Wiki:** [wiki.cachyos.org](https://wiki.cachyos.org/)
- **Live ISO Repo:** [CachyOS-Live-ISO](https://github.com/CachyOS/CachyOS-Live-ISO)
- **Oberon Governor:** [GitLab](https://gitlab.com/mothenjoyer69/oberon-governor)

---

**Related Guides:**
- [Arch Linux Setup](arch.md)
- [Fedora Setup](fedora.md)
- [Bazzite Setup](bazzite.md)
- [GPU Governor](../system/governor.md)
