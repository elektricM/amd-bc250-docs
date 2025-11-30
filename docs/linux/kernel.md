# Kernel Requirements and Configuration

The Linux kernel version and configuration is critical for BC-250 stability and performance.

## Kernel Version Compatibility

### Recommended Kernels

**Best Compatibility:**
- **6.15.7 - 6.17.7** - Full BC-250 support, best performance
- **6.16.x** - All versions work well

**Stable Fallback:**
- **6.12.x LTS** - Older but reliable
- **6.13.x** - Stable
- **6.14.x LTS** - Well-tested

**Confirmed Working Versions:**
- 6.16.5 (Fedora 42/43)
- 6.15.11-1-lts (Arch Linux)
- 6.17.4 (CachyOS)

!!!success "Current Recommendation"
    Use kernels **6.15.7 through 6.17.7** for the best BC-250 experience. Kernel support was significantly improved starting with 6.15.7.

### Broken Kernels

!!!danger "Avoid These Kernel Versions"
    - **6.15.0 - 6.15.6:** GPU initialization fails
    - **6.17.8 and newer:** GPU driver broken again

    These versions cause:
    - Kernel panics on boot
    - GPU fails to initialize
    - `amdgpu: Failed to get gpu_info firmware` error
    - Black screen after boot

**Known Error Messages:**
```
[drm:amdgpu_discovery_init [amdgpu]] *ERROR* amdgpu_discovery_init failed
amdgpu 0000:01:00.0: amdgpu: Fatal error during GPU init
```

## Required Kernel Parameters

### Basic Parameters (All Installations)

Add these to GRUB configuration:

```bash
# Edit GRUB
sudo nano /etc/default/grub

# Add to GRUB_CMDLINE_LINUX_DEFAULT:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0"
```

**Parameter Explanations:**

- `quiet` - Reduces boot messages (optional)
- `amdgpu.sg_display=0` - **Required for kernels < 6.10** - disables scatter-gather display

!!!info "amdgpu.sg_display"
    This parameter is only needed for kernels older than 6.10. If using 6.11+, it doesn't hurt to leave it, but it's not strictly necessary.

### Performance Parameters (Optional)

Disabling CPU security mitigations provides a noticeable performance boost for gaming:

**Performance Impact of `mitigations=off`:**
- +18 FPS in Cyberpunk 2077 (60 → 78 FPS at 1080p high settings)
- Reduces CPU overhead from Spectre/Meltdown mitigations
- **Security trade-off:** Disables CPU vulnerability mitigations

!!!warning "Security vs Performance"
    `mitigations=off` improves performance but reduces security. Only use on dedicated gaming systems, not for systems handling sensitive data.

**Fedora/Arch/Debian (GRUB-based):**
```bash
# Edit GRUB config
sudo nano /etc/default/grub

# Add mitigations=off to GRUB_CMDLINE_LINUX_DEFAULT:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0 mitigations=off"

# Update GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # Fedora
sudo update-grub                              # Debian/Ubuntu
sudo grub-mkconfig -o /boot/grub/grub.cfg    # Arch

# Reboot
sudo reboot
```

**Bazzite/Fedora Atomic (rpm-ostree):**
```bash
rpm-ostree kargs --append-if-missing="mitigations=off"
systemctl reboot
```

### Memory Allocation Parameters (Advanced)

For maximum GPU memory access (14.5-14.75GB):

```bash
# Add to kernel parameters
amdgpu.gttsize=14750 ttm.pages_limit=3776000 ttm.page_pool_size=3776000
```

!!! danger "Do NOT Enable IOMMU"
    **NEVER use `amd_iommu=on`** - IOMMU is broken on BC-250 and causes crashes and display failures. The memory parameters above work WITHOUT enabling IOMMU.

**Purpose:**
- Allows GPU to access more system RAM
- Useful for VRAM-heavy workloads
- Alternative to increasing BIOS VRAM allocation

**Via modprobe (alternative method):**
```bash
# Create /etc/modprobe.d/increase_amd_memory.conf
sudo nano /etc/modprobe.d/increase_amd_memory.conf

# Add:
options ttm pages_limit=3776000 page_pool_size=3776000
options amdgpu gttsize=14750

# Rebuild initramfs
sudo dracut --regenerate-all --force  # Fedora
sudo mkinitcpio -P  # Arch
```

### Installation Parameter (Temporary)

**nomodeset - Use during installation only:**

```bash
# Add temporarily at GRUB menu (press 'e' to edit)
nomodeset
```

**Purpose:** Disables GPU driver, allows booting with basic graphics

!!!danger "Remove After Installation"
    After installing Mesa drivers, **remove nomodeset** from GRUB. Leaving it prevents GPU acceleration.

## Applying Kernel Parameters

### Fedora/RHEL

```bash
# Edit GRUB config
sudo nano /etc/default/grub

# Update GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Reboot
sudo reboot
```

### Debian/Ubuntu

```bash
# Edit GRUB config
sudo nano /etc/default/grub

# Update GRUB
sudo update-grub

# Reboot
sudo reboot
```

### Arch Linux

```bash
# Edit GRUB config
sudo nano /etc/default/grub

# Update GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Reboot
sudo reboot
```

### Temporary Boot Parameters

To test parameters without permanently changing configuration:

1. Reboot system
2. At GRUB menu, press **e** to edit boot entry
3. Find line starting with `linux` or `linuxefi`
4. Add parameters to end of line
5. Press **Ctrl+X** to boot with modified parameters

## Kernel Management

### Checking Current Kernel

```bash
# Display kernel version
uname -r

# Example output: 6.14.4-104.fc42.x86_64
```

### Installing Specific Kernel Version

**Fedora:**
```bash
# List available kernels
dnf list kernel --showduplicates

# Install specific version
sudo dnf install kernel-6.14.4-104

# Set as default in GRUB if needed
sudo grub2-set-default "Fedora Linux (6.14.4-104)"
```

**Arch Linux:**
```bash
# Install LTS kernel
sudo pacman -S linux-lts linux-lts-headers

# Set as default in bootloader
```

### Holding Kernel Version

To prevent automatic updates to problematic kernel versions:

**Fedora:**
```bash
# Install versionlock plugin
sudo dnf install python3-dnf-plugin-versionlock

# Lock kernel at current version
sudo dnf versionlock add kernel

# List locked packages
sudo dnf versionlock list

# Remove lock
sudo dnf versionlock delete kernel
```

**Arch Linux:**
```bash
# Edit pacman.conf
sudo nano /etc/pacman.conf

# Add under [options]:
IgnorePkg = linux

# Save and exit
```

**Debian:**
```bash
# Hold kernel package
sudo apt-mark hold linux-image-6.14.11-amd64

# Unhold
sudo apt-mark unhold linux-image-6.14.11-amd64
```

### Removing Broken Kernel

If kernel 6.15.0-6.15.6 or 6.17.8+ was installed and causes issues:

**Fedora:**
```bash
# Boot into working kernel from GRUB menu
# Remove broken kernel (example: 6.17.8)
sudo dnf remove kernel-6.17.8\*

# Or remove all 6.17.8+ kernels
sudo dnf remove 'kernel-6.17.[8-9]*' 'kernel-6.17.1[0-9]*'

# Verify removal
dnf list installed kernel
```

**Arch Linux:**
```bash
# Boot into working kernel
# Remove problematic kernel
sudo pacman -R linux  # if on broken version

# Install known-good kernel
sudo pacman -S linux-lts  # 6.12 or 6.14 LTS
# or
sudo pacman -S linux  # check version is 6.15.7-6.17.7
```

## Kernel Patches for BC-250

### GPU Frequency Range Patch

**Purpose:** Enables extended frequency range (350 MHz - 2230 MHz) instead of default (1000-2000 MHz)

**Distributions with Patch Included:**
- Bazzite (pre-applied)
- PikaOS (pre-applied)

**Manual Patching:**

Required for:
- Fedora
- Arch Linux
- Debian
- Other distributions

**Patch Application (Advanced):**

1. Download BC-250 frequency patch
2. Apply to kernel source
3. Compile custom kernel
4. Install and boot

[Detailed patching guide available in community resources]

**Alternative:** Use distributions with patch pre-applied (Bazzite, PikaOS)

### TKG Kernel (Arch-based)

For Arch users, linux-tkg provides easy custom kernel building:

```bash
git clone https://github.com/Frogging-Family/linux-tkg
cd linux-tkg

# Create patch directory
mkdir linux612-tkg-userpatches

# Place BC-250 patch in directory

# Run installer
./install.sh install

# Select Linux 6.12 LTS during setup
```

**Benefits:**
- Easy kernel customization
- Includes performance optimizations
- BC-250 patch integration

**Compilation Time:**
- Full compile: ~45 minutes
- With modprobed-db: ~8 minutes

## Kernel Troubleshooting

### GPU Not Detected After Kernel Update

**Symptoms:**
- Black screen after boot
- `lspci | grep VGA` shows device but driver not loading
- `dmesg | grep amdgpu` shows errors

**Solution:**
1. Boot into older kernel from GRUB (hold Shift at boot)
2. Remove newer kernel
3. Hold kernel version
4. Report issue to distribution

### System Boots But No GPU Acceleration

**Check:**
```bash
# Is amdgpu module loaded?
lsmod | grep amdgpu

# Check for errors
dmesg | grep -i amdgpu

# Verify rendering
glxinfo | grep "OpenGL renderer"
# Should NOT show "llvmpipe"
```

**Possible Causes:**
- `nomodeset` still in GRUB config
- Missing kernel parameters
- Mesa not installed

### Kernel Panic on Boot

**Symptoms:**
- System crashes during boot
- Kernel panic message

**Most Common Cause:** Kernel 6.15.0-6.15.6 or 6.17.8+

**Solution:**
1. Boot into working kernel (hold Shift, select previous kernel)
2. Remove problematic kernel version
3. Lock kernel version to prevent auto-update

## Kernel Version Matrix

| Kernel Version | Status | Notes |
|---------------|--------|-------|
| 6.10.x | ⚠️ Works | `amdgpu.sg_display=0` required |
| 6.11.x | ✅ Good | Mesa 25.1+ required |
| 6.12.x LTS | ✅ Good | Stable fallback |
| 6.13.x | ✅ Good | Stable |
| 6.14.x LTS | ✅ Good | Well-tested |
| 6.15.0-6.15.6 | ❌ **Broken** | GPU init fails |
| 6.15.7-6.15.x | ✅ **Recommended** | Kernel support fixed |
| 6.16.x | ✅ **Recommended** | Full compatibility |
| 6.17.0-6.17.7 | ✅ **Recommended** | Best support |
| 6.17.8+ | ❌ **Broken** | GPU driver broken again |

## See Also

- [Mesa Driver Installation](mesa.md)
- [Distribution Comparison](distributions.md)
- [System Configuration](../system/governor.md)
- [Troubleshooting Guide](../troubleshooting/display.md)
