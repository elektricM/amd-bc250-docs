# RADV Driver Guide for BC-250

This guide covers the Radeon Vulkan (RADV) driver, which is the primary graphics driver for BC-250 on Linux.

---

## What is RADV?

**RADV** (Radeon Vulkan) is the open-source Vulkan driver for AMD GPUs, developed as part of Mesa. It provides Vulkan API support for the BC-250's "Cyan Skillfish" GPU (GFX1013 architecture).

### Why RADV for BC-250?

The BC-250 **only works on Linux** for gaming and desktop use. There is no Windows GPU driver support, making RADV your only option for graphics acceleration:

- **Linux-only GPU support** - No Windows drivers exist
- **Open-source** - Part of Mesa, actively maintained
- **BC-250 specific support** - Added in Mesa 25.1.0
- **Better than alternatives** - AMDVLK and proprietary drivers don't support this GPU
- **Vulkan + Zink** - Handles both Vulkan games and OpenGL (via Zink translation)

### BC-250 GPU Architecture

- **Codename:** Cyan Skillfish
- **Architecture:** GFX1013 (RDNA 1.5)
- **Compute Units:** 24 RDNA2 CUs
- **Features:** Hardware ray tracing cores
- **VRAM:** 512MB-15.5GB configurable (shared with CPU)
- **Vulkan ID:** `RADV GFX1013`

!!! note "RDNA 1.5 Hybrid"
    The BC-250 uses a unique hybrid architecture combining Zen 2 CPU cores with cut-down PS5 GPU cores. It's not a standard AMD APU - it uses GDDR6 instead of DDR4/DDR5, and has hardware features not found in consumer APUs.

---

## Mesa Version Requirements

### Minimum: Mesa 25.1.0

BC-250 support was added upstream in Mesa 25.1.0. **Do not use older versions** - they will not work properly or at all.

### Recommended: Mesa 25.1.3+

For stability and performance, use Mesa 25.1.3 or newer:

- **25.1.0** - Initial BC-250 support
- **25.1.3** - Improved stability
- **25.1.5+** - Best tested version, recommended

### What Changed in Mesa 25.1?

Mesa 25.1 includes critical fixes for BC-250:

1. **Compute queue fix** - Disables broken compute-only queue ([MR 33116](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33116))
2. **Unified heap on APU** - Memory management improvements
3. **GFX1013 recognition** - Proper GPU identification
4. **Artifact fixes** - Reduces visual glitches

!!! warning "Mesa 25.0 and Older"
    Mesa versions before 25.1 require custom patches and are no longer supported. All major distributions now have Mesa 25.1+ in their repositories.

---

## Installation by Distribution

### Fedora 42/43

Mesa 25.1 is now in mainline Fedora repositories (as of Fedora 43):

```bash
# Update to latest Mesa
sudo dnf update mesa-*

# Verify version
glxinfo | grep "OpenGL version"
# Should show: Mesa 25.1.X or newer
```

**Fedora 42** may need a manual update if on an older installation:

```bash
# Check current version
dnf list installed | grep mesa

# If < 25.1, update system
sudo dnf upgrade --refresh
```

### Bazzite

Bazzite includes Mesa 25.1+ by default:

```bash
# Check Mesa version
rpm -qa | grep mesa

# Update if needed
rpm-ostree upgrade

# For flatpak apps, install mesa-git (see Flatpak section below)
```

### Arch Linux / CachyOS

Mesa 25.1+ is in the official Arch repositories:

```bash
# Install/update Mesa
sudo pacman -S mesa vulkan-radeon lib32-vulkan-radeon

# Verify installation
pacman -Q mesa vulkan-radeon
```

**CachyOS** uses the same packages but may have additional optimizations:

```bash
# CachyOS may have mesa-git available
sudo pacman -S mesa-git vulkan-radeon-git
```

### Debian / PikaOS

**Debian** requires the experimental repository for Mesa 25.1:

```bash
# Add experimental repo to /etc/apt/sources.list
deb http://deb.debian.org/debian experimental main contrib non-free

# Install Mesa from experimental
sudo apt update
sudo apt install -t experimental mesa-vulkan-drivers libgl1-mesa-dri mesa-utils

# Verify
glxinfo | grep "OpenGL version"
```

**PikaOS** includes Mesa 25.1+ out of the box:

```bash
# Update system
sudo apt update && sudo apt upgrade

# Mesa 25.1 should be installed by default
```

### Manjaro

Manjaro repositories include Mesa 25.1+:

```bash
# Update Mesa
sudo pacman -Syu mesa

# Install Vulkan drivers
sudo pacman -S vulkan-radeon lib32-vulkan-radeon
```

---

## Verification Commands

### Check Mesa Version

```bash
# OpenGL version string includes Mesa version
glxinfo | grep "OpenGL version"
# Expected: OpenGL version string: 4.6 (Compatibility Profile) Mesa 25.1.X

# Alternative method
vulkaninfo | grep "driverVersion"
```

### Check RADV is Active

```bash
# Should show RADV, not AMDVLK or llvmpipe
vulkaninfo | grep "driverName"
# Expected: driverName = radv

# Check GPU name
vulkaninfo | grep "deviceName"
# Expected: deviceName = AMD Radeon Graphics (RADV GFX1013)
```

### Check Vulkan Capabilities

```bash
# Full Vulkan device info
vulkaninfo | grep -A 20 "VkPhysicalDeviceProperties"

# Should show:
# - deviceType = PHYSICAL_DEVICE_TYPE_INTEGRATED_GPU
# - vendorID = 0x1002 (AMD)
# - driverName = radv
```

### Verify GPU is Not Using Software Rendering

```bash
# Check for llvmpipe (software rendering)
vulkaninfo | grep -i llvmpipe
# Should return NOTHING - if llvmpipe appears, GPU is not working

# Alternative check
glxinfo | grep "OpenGL renderer"
# Should show: AMD Radeon Graphics (gfx1013, LLVM...)
# NOT: llvmpipe or software rasterizer
```

---

## Performance Tuning

### Environment Variables

These environment variables control RADV behavior and can fix issues or improve performance.

#### Core RADV Variables

```bash
# Force RADV (not AMDVLK) - usually automatic
export AMD_VULKAN_ICD=RADV

# Disable compute queue (may not be needed on Mesa 25.1+)
# Only use if you have visual artifacts
export RADV_DEBUG=nocompute

# Disable hierarchical Z buffer (fixes some visual glitches)
export RADV_DEBUG=nohiz

# Combine multiple debug flags
export RADV_DEBUG=nocompute,nohiz
```

!!! info "RADV_DEBUG=nocompute on Mesa 25.1+"
    Mesa 25.1 includes [MR 33116](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33116) which automatically disables the broken compute-only queue. You may not need `RADV_DEBUG=nocompute` anymore, but it doesn't hurt to keep it.

#### Memory Management

```bash
# Enable unified heap on APU (recommended)
# Add to /etc/drirc:
<driconf>
    <device>
        <application name="Default">
            <option name="radv_enable_unified_heap_on_apu" value="true" />
        </application>
    </device>
</driconf>
```

This configuration improves memory management for the BC-250's shared CPU/GPU memory architecture.

#### OpenGL via Zink

For OpenGL applications, use Zink (OpenGL on Vulkan) for better performance:

```bash
# Force Zink for OpenGL
export MESA_LOADER_DRIVER_OVERRIDE=zink

# Or per-application
MESA_LOADER_DRIVER_OVERRIDE=zink ./opengl_game
```

### Steam Launch Options

Set environment variables per-game in Steam:

```bash
# Right-click game -> Properties -> Launch Options

# Basic (recommended)
RADV_DEBUG=nohiz %command%

# With compute queue disabled
RADV_DEBUG=nocompute,nohiz %command%

# With Zink for OpenGL games
MESA_LOADER_DRIVER_OVERRIDE=zink %command%

# All combined
AMD_VULKAN_ICD=RADV RADV_DEBUG=nohiz %command%
```

### System-Wide Configuration

For system-wide settings, add to `/etc/environment`:

```bash
# Edit /etc/environment
sudo nano /etc/environment

# Add these lines:
AMD_VULKAN_ICD=RADV
RADV_DEBUG=nohiz
MESA_LOADER_DRIVER_OVERRIDE=zink
```

**Apply changes:**
```bash
# Log out and back in, or reboot
sudo reboot
```

### Vulkan ICD Configuration

Make sure RADV ICD files are used by Vulkan loader:

```bash
# Check ICD files exist
ls /usr/share/vulkan/icd.d/
# Should show: radeon_icd.x86_64.json, radeon_icd.i686.json

# If needed, manually set ICD files
export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json:/usr/share/vulkan/icd.d/radeon_icd.i686.json
```

This is rarely needed on modern distributions but may help with hardware acceleration in Steam Big Picture mode.

---

## Known Issues with RADV on BC-250

### Visual Artifacts and Glitches

**Symptoms:**
- Texture corruption
- Flickering
- Visual glitches in games
- Black textures

**Solutions:**

1. **Update to Mesa 25.1.3+** - Many artifacts fixed in newer versions
2. **Use `RADV_DEBUG=nohiz`** - Disables hierarchical Z buffer
3. **Apply [MR 33962](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33962)** - Additional artifact fixes (may need to compile Mesa yourself)
4. **Enable unified heap** - Add drirc configuration (see Memory Management section)

```bash
# Test with nohiz first
RADV_DEBUG=nohiz ./game

# If that doesn't help, try nocompute as well
RADV_DEBUG=nocompute,nohiz ./game
```

### Compute Queue Issues

**Symptoms:**
- Games crash on launch
- Applications hang when using compute shaders
- "Out of memory" errors despite available VRAM

**Solution:**

The compute-only queue on BC-250 is broken. Mesa 25.1+ disables it automatically, but you can force it:

```bash
export RADV_DEBUG=nocompute
```

**Background:** The BC-250's compute queue has hardware issues. Mesa 25.1 includes a workaround that detects GFX1013 and disables the compute-only queue by default.

### Limited VRAM Visibility in Vulkan

**Symptoms:**
- Vulkan sees only ~10GB of 12GB VRAM split
- Applications report less memory than configured in BIOS

**Explanation:**

This is a known limitation with Vulkan on BC-250. ROCm can see the full VRAM, but RADV may report less:

- **BIOS configured:** 4GB CPU / 12GB GPU
- **Vulkan reports:** ~10GB available
- **ROCm reports:** Full 12GB

**Workaround:**

Increase TTM memory limits to expose more system memory to GPU:

```bash
# Add to /etc/modprobe.d/ttm-mem-limit.conf
options ttm pages_limit=3959290 page_pool_size=3959290

# Rebuild initramfs
sudo dracut --regenerate-all --force  # Fedora
sudo mkinitcpio -P                     # Arch
sudo update-initramfs -u               # Debian/Ubuntu

# Reboot
```

This allows the GPU to use system RAM as VRAM overflow, partially compensating for the limited visibility.

### No Video Encoding/Decoding Hardware Acceleration

**Symptoms:**
- VA-API doesn't work
- `vainfo` shows errors or no devices
- Hardware video encoding fails

**Explanation:**

The BC-250's video encode/decode hardware (VCN) is disabled or non-functional. This is a hardware limitation, not a driver issue:

```bash
# vainfo will fail
vainfo
# Error: vaInitialize failed with error code -1 (unknown libva error)
```

**Workaround:**

Use software encoding/decoding (CPU). Performance is acceptable for most use cases:

```bash
# For video playback, use software decoding
# mpv, VLC, etc. will fall back automatically

# For OBS recording, use software encoder (x264)
```

### Ray Tracing Performance

**Symptoms:**
- Ray traced games run slowly
- RT cores not being utilized effectively

**Explanation:**

The BC-250 has hardware RT cores, but they're RDNA 1.5 generation (not as efficient as RDNA 2/3). Mesa's RT implementation is improving but not fully optimized for GFX1013:

- Portal RTX: ~40 FPS at 720p (with RT)
- Quake 2 RTX: Playable but slow
- Modern RT games: Generally too demanding

**Recommendations:**
- Disable ray tracing in demanding games
- Use lower resolutions (720p-900p) for RT games
- Wait for Mesa improvements

---

## Comparison with Other AMD Drivers

### RADV vs AMDVLK

| Feature | RADV | AMDVLK |
|---------|------|--------|
| **Open Source** | Yes (Mesa) | Yes (AMD) |
| **BC-250 Support** | ✅ Yes (Mesa 25.1+) | ❌ No |
| **Performance** | Better for most games | Good for some specific games |
| **Updates** | Frequent (Mesa releases) | Less frequent |
| **Compatibility** | Excellent | Limited for BC-250 |

**Verdict:** RADV is the only choice for BC-250. AMDVLK does not support GFX1013.

### RADV vs Proprietary AMD Drivers

| Feature | RADV | AMD PRO |
|---------|------|---------|
| **BC-250 Support** | ✅ Yes | ❌ No |
| **Gaming** | Excellent | Not applicable |
| **OpenGL** | Via Zink | Native |
| **Vulkan** | Native | Native |
| **Linux Kernel** | Upstream AMDGPU | Same (amdgpu) |

**Verdict:** AMD's proprietary driver (AMDGPU-PRO) does not support BC-250. Even the open-source amdgpu kernel module + proprietary userspace won't work. RADV is required.

### RADV vs ROCm

| Feature | RADV | ROCm |
|---------|------|------|
| **Purpose** | Gaming (Vulkan) | Compute (HIP/OpenCL) |
| **BC-250 Support** | ✅ Full | ⚠️ Partial |
| **Graphics** | Excellent | No |
| **Compute** | Limited (Vulkan compute) | Full (when working) |
| **AI/ML** | Via Vulkan backends | Native |

**Verdict:** Use RADV for gaming, ROCm for compute workloads (if you can get it working - see LLM inference section).

!!! warning "ROCm on BC-250 is Experimental"
    ROCm support for GFX1013 is incomplete. rocBLAS is missing pre-compiled kernels for this architecture, requiring manual compilation or workarounds. For most users, stick with RADV + Vulkan compute backends (e.g., for llama.cpp).

---

## Advanced Topics

### Mesa Compilation from Source

If you need bleeding-edge fixes or custom patches:

#### Install Build Dependencies

**Fedora:**
```bash
sudo dnf install meson ninja-build gcc g++ python3-mako \
    libdrm-devel libxcb-devel libX11-devel libxshmfence-devel \
    libXext-devel libXfixes-devel libXrandr-devel \
    wayland-protocols-devel wayland-devel \
    elfutils-devel llvm-devel cmake flex bison
```

**Arch:**
```bash
sudo pacman -S meson ninja gcc python-mako libdrm libxcb libx11 \
    libxshmfence libxext libxfixes libxrandr \
    wayland wayland-protocols elfutils llvm cmake flex bison
```

#### Compile Mesa

```bash
# Clone Mesa
git clone https://gitlab.freedesktop.org/mesa/mesa.git
cd mesa

# Checkout stable branch or apply custom patches
git checkout mesa-25.1

# Apply custom patches if needed
# git am /path/to/patch.patch

# Configure build
meson setup builddir/ \
    -Dprefix=/usr/local \
    -Dvulkan-drivers=amd \
    -Dgallium-drivers=radeonsi,zink \
    -Dplatforms=x11,wayland \
    -Dbuildtype=release

# Compile (use all CPU cores)
meson compile -C builddir/

# Install
sudo meson install -C builddir/

# Verify
glxinfo | grep "OpenGL version"
vulkaninfo | grep "driverName"
```

!!! danger "Compiling Mesa Can Break Your System"
    Installing Mesa to `/usr/local` may conflict with system packages. Only do this if you know how to revert changes. Consider using a separate prefix like `/opt/mesa-git` and setting `LD_LIBRARY_PATH`.

#### Applying BC-250 Specific Patches

Some community patches may not be upstream yet:

```bash
# Example: Apply MR 33962 (artifact fixes)
cd mesa
git fetch origin merge-requests/33962/head:mr33962
git checkout mr33962

# Compile as above
```

### Using Flatpak Apps with Mesa-git

Flatpak apps use their own runtime Mesa, which may be outdated. Force them to use newer Mesa:

```bash
# Add Flathub Beta repository
flatpak remote-add --if-not-exists flathub-beta \
    https://flathub.org/beta-repo/flathub-beta.flatpakrepo

# Install mesa-git for runtime 24.08 (check your flatpak runtime version)
flatpak install --system flathub-beta org.freedesktop.Platform.GL.mesa-git//24.08
flatpak install --system flathub-beta org.freedesktop.Platform.GL32.mesa-git//24.08

# Configure systemd to use mesa-git
sudo mkdir -p /etc/systemd/system/service.d
sudo bash -c 'echo -e "[Service]\nEnvironment=FLATPAK_GL_DRIVERS=mesa-git" > \
    /etc/systemd/system/service.d/99-flatpak-mesa-git.conf'

# Reboot or restart flatpak services
sudo systemctl daemon-reload
```

**Verify:**
```bash
# Run flatpak app and check Mesa version
flatpak run --command=sh com.valvesoftware.Steam
$ glxinfo | grep "OpenGL version"
```

!!! note "Flatpak Runtime Version"
    Runtimes older than 23.08 may not support Mesa 25.1. Ensure your flatpak apps use runtime 24.08 or newer.

### LLM Inference with Vulkan

For running large language models (LLMs) using Vulkan backend:

**llama.cpp with Vulkan:**

```bash
# Download pre-compiled llama.cpp Vulkan binary
wget https://github.com/ggerganov/llama.cpp/releases/download/b6104/llama-b6104-bin-ubuntu-vulkan-x64.zip
unzip llama-b6104-bin-ubuntu-vulkan-x64.zip
cd build/bin

# Set environment variable to avoid OOM errors
export GGML_VK_FORCE_MAX_ALLOCATION_SIZE=2000000000  # 2GB chunks

# Run inference
./llama-server --model /path/to/model.gguf --gpu-layers 99

# Expected output:
# ggml_vulkan: Found 1 Vulkan devices:
# ggml_vulkan: 0 = AMD Radeon Graphics (RADV GFX1013) (radv) | uma: 1 | fp16: 1
```

**Performance:**
- 4-bit quantized 8B model: ~60 tokens/sec
- 12GB VRAM split recommended for larger models
- Vulkan backend more stable than ROCm for BC-250

**Known Issues:**
- Vulkan sees ~10GB of 12GB VRAM (see VRAM visibility issue above)
- Large models (70B+) may OOM even with quantization
- Use `GGML_VK_FORCE_MAX_ALLOCATION_SIZE` to prevent allocation errors

---

## Troubleshooting

### RADV Not Being Used (llvmpipe fallback)

**Symptoms:**
```bash
vulkaninfo | grep deviceName
# Shows: llvmpipe (LLVM 18.1.0, 256 bits)
```

**Cause:** Mesa drivers not installed or not working.

**Solution:**
1. Verify Mesa version: `glxinfo | grep Mesa`
2. Install Vulkan RADV driver: `sudo dnf install mesa-vulkan-drivers` (Fedora) or equivalent
3. Check amdgpu kernel module: `lsmod | grep amdgpu`
4. Check dmesg for errors: `dmesg | grep -i amdgpu`

### Vulkan Initialization Fails

**Symptoms:**
```bash
vulkaninfo
# Failed to create Vulkan instance
# ERROR: [Loader Message] Code 0 : No drivers found
```

**Cause:** ICD files missing or incorrect.

**Solution:**
```bash
# Check ICD files
ls /usr/share/vulkan/icd.d/
# Should have: radeon_icd.x86_64.json

# Manually set ICD path
export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json

# Reinstall Mesa Vulkan drivers
sudo dnf reinstall mesa-vulkan-drivers  # Fedora
sudo pacman -S vulkan-radeon            # Arch
```

### Visual Artifacts Persist After Mesa 25.1

**Symptoms:**
- Textures still corrupted
- Flickering in certain games
- Black squares/triangles

**Solution (priority order):**

1. **Update to Mesa 25.1.5+**
2. **Use `RADV_DEBUG=nohiz`**
3. **Apply MR 33962 patch** (requires compiling Mesa)
4. **Enable unified heap in drirc** (see Memory Management section)
5. **Test with different kernel versions** (6.12-6.14 work best)

### Games Crash on Shader Compilation

**Symptoms:**
- Game crashes during loading
- "Compiling shaders" freezes
- Steam shader pre-caching fails

**Solution:**
```bash
# Clear shader cache
rm -rf ~/.cache/mesa_shader_cache
rm -rf ~/.local/share/Steam/steamapps/shadercache

# Disable shader cache temporarily
export MESA_SHADER_CACHE_DISABLE=1

# Run game
./game

# If that works, re-enable cache and pre-compile:
unset MESA_SHADER_CACHE_DISABLE
# Let game rebuild cache
```

### Poor Performance Despite RADV Working

**Symptoms:**
- Low FPS in games
- GPU usage low in `nvtop`
- GPU frequency stuck at 1500MHz

**Cause:** GPU governor not running (see System Configuration guide).

**Quick Check:**
```bash
# Check GPU frequency
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Should show multiple frequency levels with * at current:
# 0: 1000Mhz
# 1: 1500Mhz *
# 2: 2000Mhz

# If stuck at 1500MHz, governor is not working
systemctl status oberon-governor
```

**Solution:** Install and configure GPU governor (see [System Configuration](../system/governor.md) guide).

---

## Summary

**RADV** is the only working graphics driver for BC-250 on Linux:

- ✅ **Required:** Mesa 25.1.0 minimum (25.1.5+ recommended)
- ✅ **Works with:** All major Linux distributions (Fedora, Arch, Debian, Bazzite, etc.)
- ✅ **Performance:** Good for 720p-1080p gaming, competitive with RX 6600
- ✅ **Environment variables:** `RADV_DEBUG=nohiz` recommended, `nocompute` may not be needed on Mesa 25.1+
- ✅ **Known issues:** Visual artifacts (mostly fixed), limited VRAM visibility, no VA-API
- ❌ **No Windows support:** BC-250 GPU does not work on Windows

For most users, install Mesa 25.1+ from your distribution's repositories, set `RADV_DEBUG=nohiz` in Steam launch options, install the GPU governor, and enjoy gaming.

---

**Last Updated:** 2025-11-21
**Based on:** Discord community discussions (7,000+ messages analyzed)
**See Also:** [Linux Setup Guide](../linux/distributions.md), [System Configuration](../system/governor.md), [Gaming & Performance](../gaming/compatibility.md)
