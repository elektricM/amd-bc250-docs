# Environment Variables Guide

This guide covers all environment variables for BC-250 graphics configuration, performance tuning, and debugging.

## Critical Environment Variables

!!!warning "IMPORTANT: RADV_DEBUG is an ENVIRONMENT VARIABLE, NOT a kernel argument"
    Do NOT add it to kernel parameters (grub, /etc/default/grub, etc.).

    Set RADV_DEBUG only via:
    - Steam launch options: `RADV_DEBUG=nohiz %command%`
    - Shell: `export RADV_DEBUG=nohiz` before launching
    - /etc/environment (system-wide)
    - Per-application wrappers

### RADV_DEBUG=nocompute

**Status:** DEPRECATED on Mesa 25.1+ / **REMOVED in Mesa 25.2+**

The compute queue issue is resolved upstream. Only use if running Mesa 25.0 or earlier. **Fedora 43 ships Mesa 25.3.6** — this flag has no effect and is not needed.

For Mesa 25.1 and newer (including 25.2.x), use RADV_DEBUG=nohiz instead for rendering stability.

**What it did:**
The BC-250's compute queue had hardware issues that caused graphical artifacts and rendering problems. This variable forced all workloads through the graphics queue instead.

**When to use:**

- Only needed on Mesa 25.0 or earlier
- **Not needed on Mesa 25.1+** - compute queue issue is fixed

**How to set (if needed for old Mesa):**

```bash
# Steam launch options
RADV_DEBUG=nocompute %command%

# System-wide (add to /etc/environment)
RADV_DEBUG=nocompute

# Per-application
RADV_DEBUG=nocompute ./game
```

### RADV_DEBUG=nohiz

**Purpose:** Fixes additional graphical artifacts

Some games experience rendering issues even with `nocompute`. This variable disables hierarchical Z-buffer optimization.

**When to use:**

- If you see visual artifacts or glitches in games
- Can be combined with other RADV_DEBUG options

**How to set:**

```bash
# Combine multiple RADV_DEBUG flags with commas
RADV_DEBUG=nocompute,nohiz %command%
```

## RADV (Vulkan Driver) Variables

### VK_ICD_FILENAMES

**Purpose:** Explicitly specify Vulkan driver location

Required for hardware acceleration in some applications, particularly Steam Big Picture mode.

**Usage:**

```bash
export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json:/usr/share/vulkan/icd.d/radeon_icd.i686.json
```

**When needed:**

- Steam Big Picture hardware acceleration
- Applications not finding Vulkan driver automatically
- Flatpak applications (different path)

### radv_enable_unified_heap_on_apu

**Purpose:** Workaround for memory allocation issues

Enables unified memory heap on APU systems. Helps with out-of-memory errors on some games.

**Configuration:**

Add to `/etc/drirc`:

```xml
<driconf>
    <device>
        <application name="Default">
            <option name="radv_enable_unified_heap_on_apu" value="true" />
        </application>
    </device>
</driconf>
```

**When needed:**

- Games crashing with memory errors
- Applications not utilizing full VRAM
- Mesa versions before full BC-250 memory support

## AMD GPU Variables

### AMD_LOG_LEVEL

**Purpose:** Debugging AMD driver issues

Controls verbosity of AMD GPU driver logging.

**Values:**

- `0` - Errors only
- `1` - Warnings
- `2` - Info
- `3` - Verbose
- `4` - Debug (very verbose)

**Usage:**

```bash
AMD_LOG_LEVEL=4 ./application
```

**When to use:**

- Troubleshooting driver issues
- Debugging ROCm/compute problems
- Investigating crashes or hangs

### HSA_OVERRIDE_GFX_VERSION

**Purpose:** Override GPU architecture detection

Forces applications to treat the BC-250 (gfx1013) as a different architecture.

**Common values:**

- `10.1.0` - Pretend to be gfx1010 (RDNA 1.0)
- `10.3.0` - Pretend to be gfx1030 (RDNA 2.0)

**Usage:**

```bash
HSA_OVERRIDE_GFX_VERSION=10.1.0 ./rocm-application
```

**When to use:**

- ROCm applications that don't support gfx1013
- Getting limited support in compute workloads
- LLM inference with llama.cpp (limited success)

**Warning:** This is a workaround and may cause instability or incorrect results.

## LLM/Compute Variables

### GGML_VK_FORCE_MAX_ALLOCATION_SIZE

**Purpose:** Limit Vulkan memory allocation size for llama.cpp

Prevents out-of-memory errors by limiting single allocation size.

**Value:**

```bash
GGML_VK_FORCE_MAX_ALLOCATION_SIZE=2000000000  # 2GB in bytes
```

**When to use:**

- Running llama.cpp with Vulkan backend
- Out-of-memory errors with large models
- Memory fragmentation issues

**Performance:**

- Enables 4-bit quantized 8B models at ~60 tokens/sec
- Vulkan sees only ~10GB of 12GB VRAM split (limitation)

### TTM Module Configuration

**Purpose:** Control GPU shared memory management

The TTM (Translation Table Manager) module manages shared memory between CPU and GPU.

**Configuration:**

Create `/etc/modprobe.d/ttm-mem-limit.conf`:

```bash
options ttm pages_limit=3959290 page_pool_size=3959290
```

**What it does:**

- Sets available memory pages for GPU (value in 4KB pages)
- Default may only allow 8GB even with 12GB VRAM split
- Allows full RAM to be available for VRAM if necessary

**Calculate pages_limit:**

```bash
# For 15GB available to GPU (3959290 pages × 4KB)
# (Total RAM - 1GB) / 4096 bytes per page
```

## Performance Variables

### mitigations=off

**Type:** Kernel boot parameter (not environment variable)

Disables CPU security mitigations for better processor performance.

**How to set:**

Edit `/etc/default/grub`:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="... mitigations=off"
```

Then update GRUB:

```bash
# Fedora/RHEL
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Debian/Ubuntu
sudo update-grub
```

**Performance gain:**

- 5-10% CPU performance improvement
- Particularly helps with emulation and CPU-bound games

**Security trade-off:**

- Disables Spectre, Meltdown, and other vulnerability mitigations
- Only use on trusted/isolated systems

## Flatpak Variables

### FLATPAK_GL_DRIVERS

**Purpose:** Force Flatpak to use specific Mesa drivers

Enables mesa-git drivers in Flatpak applications.

**Setup:**

```bash
sudo mkdir -p /etc/systemd/system/service.d
sudo bash -c 'echo -e "[Service]\nEnvironment=FLATPAK_GL_DRIVERS=mesa-git" > /etc/systemd/system/service.d/99-flatpak-mesa-git.conf'

flatpak remote-add --if-not-exists flathub-beta https://flathub.org/beta-repo/flathub-beta.flatpakrepo
flatpak install --system flathub-beta org.freedesktop.Platform.GL.mesa-git//24.08
flatpak install --system flathub-beta org.freedesktop.Platform.GL32.mesa-git//24.08
```

## ROCm Variables (Limited Support)

### GGML_HIP_UMA

**Purpose:** Enable Unified Memory Architecture for HIP

Used when compiling llama.cpp with ROCm support.

**Usage:**

```bash
HIPCXX="$(hipconfig -l)/clang" HIP_PATH="$(hipconfig -R)" \
cmake -S . -B build \
  -DLLAMA_CURL=ON \
  -DGGML_HIP=ON \
  -DAMDGPU_TARGETS=gfx1013 \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_HIP_COMPILER_ROCM_ROOT=/usr \
  -DGGML_HIP_UMA=ON
```

**Current status:**

- ROCm support is very limited on BC-250
- rocBLAS missing gfx1013 binaries (`TensileLibrary_lazy_gfx1013.dat`)
- Workarounds produce gibberish output
- Vulkan backend recommended instead

## Setting Variables

### Per-Game (Steam)

Right-click game > Properties > Launch Options:

```bash
RADV_DEBUG=nocompute %command%
RADV_DEBUG=nocompute,nohiz DXVK_HUD=fps %command%
```

### System-Wide (All Applications)

Edit `/etc/environment`:

```bash
RADV_DEBUG=nocompute
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json:/usr/share/vulkan/icd.d/radeon_icd.i686.json
```

### Per-Session (Terminal)

```bash
export RADV_DEBUG=nocompute
export AMD_LOG_LEVEL=2
./your-application
```

### systemd Service Files

For services like game servers:

```ini
[Service]
Environment="RADV_DEBUG=nocompute"
Environment="VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json"
```

## Proton/Wine Variables

### DXVK_HUD

**Purpose:** Display DXVK performance overlay

```bash
DXVK_HUD=fps,gpu,cpu %command%
```

### PROTON_LOG

**Purpose:** Enable Proton debug logging

```bash
PROTON_LOG=1 %command%
```

### PROTON_USE_WINED3D

**Purpose:** Use OpenGL instead of Vulkan

Fallback if Vulkan has issues:

```bash
PROTON_USE_WINED3D=1 %command%
```

## Debugging Variables

### RADV_PERFTEST

**Purpose:** Enable experimental RADV features

```bash
RADV_PERFTEST=nggc,sam  # Example: NGG culling, Smart Access Memory
```

**Warning:** Experimental features may cause instability.

### MESA_DEBUG

**Purpose:** Mesa driver debugging

```bash
MESA_DEBUG=1  # Enable debug output
```

## OpenCL Variables

### RUSTICL_ENABLE

**Purpose:** Enable Rusticl OpenCL implementation

```bash
RUSTICL_ENABLE=radeonsi
```

**Note:** OpenCL support on BC-250 is limited. The GPU may not be recognized as a GPU device.

## Complete Gaming Setup Example

Recommended launch options for Steam games (Mesa 25.1+):

```bash
RADV_DEBUG=nohiz mangohud %command%
```

With additional debugging (Mesa < 25.1 only — nocompute not needed on 25.1+):

```bash
RADV_DEBUG=nohiz DXVK_HUD=fps,gpu MANGOHUD=1 %command%
```

## Environment Variable Precedence

Variables are applied in this order (later overrides earlier):

1. System-wide `/etc/environment`
2. User profile `~/.profile` or `~/.bashrc`
3. systemd service files
4. Steam launch options
5. Direct command line `VARIABLE=value ./app`

## Verification

Check if variables are set:

```bash
# Show all environment variables
env | grep -i radv
env | grep -i amd

# Test Vulkan
vulkaninfo | grep -i "device name"

# Check if driver found
glxinfo | grep -i "opengl renderer"
```

## Mesa 25.1+ / 25.2.x Changes

With Mesa 25.1 and later (Fedora 43 ships **Mesa 25.3.6**):

- `RADV_DEBUG=nocompute` is deprecated on 25.1+ and effectively removed/ignored on 25.2.x — do not use
- Test with `RADV_DEBUG=nohiz` only (no nocompute needed)
- If you are on Mesa 25.0 or older, `nocompute` is still valid
- [Mesa MR #33116](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33116) — upstream fix reference

## Troubleshooting

### Games not using GPU

Check:

```bash
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json vulkaninfo
```

### Still getting artifacts

Try:

```bash
RADV_DEBUG=nocompute,nohiz %command%
```

### Memory errors

Add to `/etc/drirc`:

```xml
<option name="radv_enable_unified_heap_on_apu" value="true" />
```

### ROCm not working

ROCm support is experimental and very limited. Use Vulkan instead.

## Related Configuration Files

- `/etc/environment` - System-wide variables
- `/etc/drirc` - Mesa driver configuration
- `/etc/modprobe.d/` - Kernel module options
- `~/.bashrc` - User environment variables
- `~/.config/MangoHud/MangoHud.conf` - MangoHud settings

## Summary Table

| Variable | Purpose | Required | Value |
|----------|---------|----------|-------|
| `RADV_DEBUG=nocompute` | Fix compute queue | DEPRECATED (25.1+) / Removed (25.2+) | Only use on Mesa 25.0 or older |
| `RADV_DEBUG=nohiz` | Fix artifacts | Sometimes | `nohiz` |
| `VK_ICD_FILENAMES` | Vulkan driver path | For Steam Big Picture | See above |
| `AMD_LOG_LEVEL` | Debug logging | Debug only | `0-4` |
| `HSA_OVERRIDE_GFX_VERSION` | ROCm compat | ROCm only | `10.1.0` |
| `GGML_VK_FORCE_MAX_ALLOCATION_SIZE` | llama.cpp memory | LLM inference | `2000000000` |
| `mitigations=off` | CPU performance | Optional | kernel param |

## Best Practices

1. On Mesa 25.1+ (including 25.2.x), use `RADV_DEBUG=nohiz` only — `nocompute` is deprecated and ignored on 25.2.x
2. On Mesa 25.0 or older: `nocompute` is still needed, but upgrade to 25.1+ if at all possible
3. Fedora 43 ships Mesa 25.3.6 — `nocompute` is not needed there
4. Add variables only when needed for specific issues
5. Test changes one variable at a time
6. Use per-game settings in Steam rather than system-wide when possible
