# Mesa Driver Installation

Mesa provides the OpenGL and Vulkan drivers (RADV) required for BC-250 GPU support.

## Mesa Requirements

### Minimum Version

**Mesa 25.1.0 or newer is required**

- BC-250 (Cyan Skillfish / gfx1013) support added in Mesa 25.1
- No manual patching needed (upstream support)
- Earlier versions will not work

### Recommended Version

**Mesa 25.1.3+ recommended for stability**

- Bug fixes and performance improvements
- Better compatibility
- Mesa 25.1.5+ even better

!!!success "Upstream Support"
    As of Mesa 25.1, BC-250 support is included upstream. No custom patches or compilation needed!

## Checking Mesa Version

```bash
# Check installed Mesa version
glxinfo | grep "OpenGL version"

# Example output:
# OpenGL version string: 4.6 (Compatibility Profile) Mesa 25.1.5

# Check Vulkan driver
vulkaninfo | grep "driverName"

# Should output: driverName = radv
```

## Installation by Distribution

### Fedora 43

**Mesa 25.1+ is in official repositories:**

```bash
# Update system (includes Mesa)
sudo dnf upgrade --refresh

# Verify Mesa version
dnf list mesa-\*
glxinfo | grep "OpenGL version"
```

No additional steps needed!

### Fedora 42

**Check version first:**

```bash
dnf list mesa-\*
```

**If Mesa < 25.1:**

Most Fedora 42 systems after updates should have 25.1+. If not, mesa-git COPR may be needed, but this is increasingly rare.

### Arch Linux / CachyOS

**Mesa 25.1+ in official repos:**

```bash
# Install/update Mesa
sudo pacman -S mesa vulkan-radeon

# Verify
pacman -Q mesa
glxinfo | grep "OpenGL version"
```

### Debian

**Mesa 25.1.3+ available in experimental:**

```bash
# Add experimental repo (if not already added)
echo "deb http://deb.debian.org/debian experimental main" | sudo tee /etc/apt/sources.list.d/experimental.list

# Update package lists
sudo apt update

# Install Mesa from experimental
sudo apt install -t experimental mesa-vulkan-drivers libgl1-mesa-dri mesa-utils

# Verify
glxinfo | grep "OpenGL version"
```

!!!warning "Experimental Repo"
    Debian experimental packages may have dependencies on other experimental packages. Use with caution.

### Ubuntu

**Mesa 25.1.5 available via PPA:**

```bash
# Add PPA (example - check for current BC-250 compatible PPA)
sudo add-apt-repository ppa:kisak/kisak-mesa

# Update and install
sudo apt update
sudo apt upgrade

# Verify
glxinfo | grep "OpenGL version"
```

### Bazzite

**Mesa 25.1+ included by default:**

```bash
# Check version
rpm -qa | grep mesa

# Update if needed
rpm-ostree upgrade
```

### Manjaro

**Mesa in official repos:**

```bash
# Update system
sudo pacman -Syu

# Verify Mesa
pacman -Q mesa
```

## Verifying Installation

### Check OpenGL

```bash
# Install mesa-utils if not present
# Fedora: sudo dnf install mesa-utils
# Arch: sudo pacman -S mesa-utils

# Check OpenGL renderer
glxinfo | grep "OpenGL renderer"

# Should show:
# OpenGL renderer string: AMD Radeon Graphics (radv gfx1013 LLVM 18.1.8 DRM 3.59 6.14.4-104.fc42.x86_64)
```

!!!danger "llvmpipe Means No GPU"
    If you see "llvmpipe" as the renderer, the GPU driver is NOT working. You're using CPU software rendering.

### Check Vulkan

```bash
# Install vulkan-tools
# Fedora: sudo dnf install vulkan-tools
# Arch: sudo pacman -S vulkan-tools

# Check Vulkan device
vulkaninfo | grep deviceName

# Should show:
# deviceName = AMD Radeon Graphics (RADV GFX1013)
```

### Verify RADV Driver

```bash
# Check driver is RADV (not AMDVLK)
vulkaninfo | grep "driverName"

# Should show: driverName = radv
```

## Environment Variables

### Required Variables

```bash
# Add to /etc/environment or ~/.bashrc

# Force RADV driver (not AMDVLK)
AMD_VULKAN_ICD=RADV
```

### Optional Variables

**RADV_DEBUG options:**

```bash
# Fix some graphical glitches
RADV_DEBUG=nohiz

# Disable compute queue (may not be needed on Mesa 25.1+)
# RADV_DEBUG=nocompute
```

**Apply globally:**

```bash
# Edit /etc/environment
sudo nano /etc/environment

# Add:
AMD_VULKAN_ICD=RADV
RADV_DEBUG=nohiz
```

**Apply per-game in Steam:**

```
RADV_DEBUG=nohiz %command%
```

### Mesa Performance Variables

**For OpenGL applications:**

```bash
# Use Zink (OpenGL over Vulkan) for better performance
MESA_LOADER_DRIVER_OVERRIDE=zink
```

!!!info "Zink Overhead"
    Zink adds slight overhead but can improve compatibility and performance for some OpenGL applications. Test per-game.

## Flatpak Mesa Override

**Problem:** Flatpak applications use runtime Mesa, which may be outdated.

**Solution:** Override with mesa-git

```bash
# Add flathub-beta repository
flatpak remote-add --if-not-exists flathub-beta https://flathub.org/beta-repo/flathub-beta.flatpakrepo

# Install mesa-git for runtime 24.08
flatpak install --system flathub-beta org.freedesktop.Platform.GL.mesa-git//24.08
flatpak install --system flathub-beta org.freedesktop.Platform.GL32.mesa-git//24.08

# Set environment for Flatpak
sudo mkdir -p /etc/systemd/system/service.d
sudo bash -c 'echo -e "[Service]\nEnvironment=FLATPAK_GL_DRIVERS=mesa-git" > /etc/systemd/system/service.d/99-flatpak-mesa-git.conf'

# Reboot
sudo reboot
```

!!!warning "Flatpak Runtime Dependency"
    Flatpaks on runtime 23.08 or older cannot use Mesa 25.1. They must be on runtime 24.08+.

## Compilation (Advanced)

### When to Compile Mesa

**Usually not needed**, but compile if:
- Distribution doesn't have Mesa 25.1+ yet
- Testing newest Mesa-git features
- Development/testing purposes

### Mesa Compilation Guide

**Dependencies (Fedora):**

```bash
sudo dnf install git meson ninja-build gcc-c++ \
    libdrm-devel libXrandr-devel libXext-devel \
    libXdamage-devel libX11-devel libxcb-devel \
    libxshmfence-devel libXxf86vm-devel libXfixes-devel \
    wayland-devel wayland-protocols-devel \
    llvm-devel libunwind-devel zlib-devel \
    expat-devel elfutils-libelf-devel python3-mako \
    flex bison
```

**Clone and Build:**

```bash
# Clone Mesa
git clone https://gitlab.freedesktop.org/mesa/mesa.git
cd mesa

# Configure
meson setup build \
    --buildtype=release \
    -Dgallium-drivers=radeonsi \
    -Dvulkan-drivers=amd \
    -Dplatforms=x11,wayland \
    -Dglx=dri

# Compile (use -j for parallel build)
ninja -C build -j$(nproc)

# Install (may need to backup old Mesa first)
sudo ninja -C build install
```

**Compilation Time:** 10-30 minutes depending on CPU

!!!danger "Backup Before Installing"
    Compiling and installing Mesa system-wide can break your system if done incorrectly. Only do this if you know what you're doing.

## Troubleshooting Mesa Issues

### GPU Not Detected

**Symptoms:**
- `vulkaninfo` shows no devices
- `glxinfo` shows llvmpipe

**Causes:**
1. Mesa < 25.1 installed
2. amdgpu kernel module not loaded
3. Kernel parameters incorrect

**Solutions:**

```bash
# Check kernel module
lsmod | grep amdgpu
# If empty, driver not loaded

# Check kernel messages
dmesg | grep -i amdgpu
# Look for errors

# Verify Mesa version
glxinfo | grep "OpenGL version"
# Must be 25.1+

# Check if nomodeset is still active
cat /proc/cmdline
# Should NOT contain "nomodeset"
```

### Software Rendering (llvmpipe)

**Symptom:**
```bash
glxinfo | grep "OpenGL renderer"
# Shows: "llvmpipe (LLVM 18.1.8, 256 bits)"
```

**Cause:** GPU driver not working, falling back to CPU rendering

**Solutions:**
1. Install Mesa 25.1+
2. Remove `nomodeset` from GRUB
3. Ensure amdgpu module loaded
4. Check kernel version (6.12-6.14)

### AMDVLK Instead of RADV

**Symptom:**
```bash
vulkaninfo | grep "driverName"
# Shows: driverName = AMDVLK
```

**Issue:** AMD's open-source Vulkan driver (AMDVLK) loaded instead of RADV

**Solution:**

```bash
# Force RADV
export AMD_VULKAN_ICD=RADV

# Make permanent
echo "AMD_VULKAN_ICD=RADV" | sudo tee -a /etc/environment

# Or uninstall AMDVLK
sudo dnf remove amdvlk  # Fedora
sudo pacman -R amdvlk   # Arch
```

### Graphical Glitches in Games

**Symptoms:**
- Black textures
- Flickering
- Visual artifacts

**Solutions:**

**Try RADV_DEBUG options:**
```bash
# Fix Z-buffer issues
RADV_DEBUG=nohiz

# Disable compute queue (older Mesa versions)
RADV_DEBUG=nocompute

# Combine multiple options
RADV_DEBUG=nohiz,nocompute
```

**Per-game in Steam launch options:**
```
RADV_DEBUG=nohiz %command%
```

### Mesa Update Breaks System

**Symptoms:**
- Black screen after Mesa update
- System doesn't boot

**Recovery:**

1. Boot into recovery mode or older kernel
2. Downgrade Mesa:

```bash
# Fedora
sudo dnf downgrade mesa\*

# Arch
sudo pacman -U /var/cache/pacman/pkg/mesa-25.1.3-1-x86_64.pkg.tar.zst

# Check what version you're downgrading to
```

3. Hold Mesa version temporarily:

```bash
# Fedora
sudo dnf versionlock add mesa\*

# Arch
# Add to /etc/pacman.conf under [options]:
# IgnorePkg = mesa
```

## Mesa Version History

| Mesa Version | BC-250 Support | Notes |
|--------------|----------------|-------|
| < 25.0 | ❌ No | No gfx1013 support |
| 25.0.x | ⚠️ Experimental | Early support, buggy |
| 25.1.0 | ✅ Yes | First official support |
| 25.1.3+ | ✅ Recommended | Stable, bug fixes |
| 25.1.5+ | ✅ Best | Latest improvements |
| 25.2+ | ✅ Good | Ongoing development |

## See Also

- [Kernel Requirements](kernel.md)
- [Distribution Comparison](distributions.md)
- [GPU Governor Setup](../system/governor.md)
- [Troubleshooting Display Issues](../troubleshooting/display.md)
