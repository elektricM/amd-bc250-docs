# BIOS Overclocking Guide

Overclocking the BC-250 GPU beyond the default 1500 MHz lock.

## Quick Start

**Most users only need to install a governor.** The governor unlocks the GPU from its 1500 MHz lock and enables dynamic frequency scaling up to 2000-2230 MHz.

1. **Install a governor** → [Governor Setup](../system/governor.md)
2. **Tune voltage/frequency** in governor config (see below)
3. **Only patch kernel if needed** — Bazzite is pre-patched, SMU governor bypasses patches entirely

### Requirements

- **GPU Governor** — required for frequency scaling (see [Governor Setup](../system/governor.md))
- **Active cooling** — high static pressure fan (Arctic P12 Max or equivalent)
- **PSU** — 250W+ on 12V rail for overclocked workloads

### What Can Be Overclocked

- **GPU Frequency:** 1500 MHz (locked) → 2000-2230 MHz (with governor)
- **GPU Voltage:** 700-1129 mV (stock kernel OD_RANGE)
- **Memory:** Adjustable via community Mem Timing Utility (advanced — incorrect settings will crash the system)

---

## GPU Frequency Range Kernel Patch

The kernel patch extends the GPU frequency range from 1000-2000 MHz to 350-2230 MHz.

**Created by:** ViRazY ([GitHub](https://github.com/Vinjul1704))

!!!success "Most Users Don't Need to Manually Patch"
    - **Bazzite/PikaOS:** Kernel already includes this patch.
    - **Any distro:** The [`cyan-skillfish-governor-smu`](https://github.com/filippor/cyan-skillfish-governor/tree/smu) bypasses the need for kernel patches entirely via SMU firmware calls.
    - Manual patching is only needed for non-Bazzite users who want to use the TT governor or manual sysfs overclocking with extended frequency range.

### What the Patch Does

- **Default range:** 1000-2000 MHz → **Patched range:** 350-2230 MHz
- **Voltage range:** 700-1129 mV (unchanged by standard patch)
- Allows idle downclocking to 350 MHz for lower power draw
- Extended voltage patch (600-1300 mV) exists but is NOT recommended

#### Pre-Patched Distributions (Easiest)

1. **Bazzite** - Kernel pre-patched, no compilation needed
   - Download: [Bazzite Kernel Releases](https://github.com/bazzite-org/kernel-bazzite/releases)

2. **Arch Linux (AUR packages)**
   - `linux-bazzite-bin` - Bazzite kernel for Arch
   - `linux-lts-amd-bc250-headers` - BC-250 LTS kernel

3. **PikaOS** - Includes GPU frequency patch by default

#### Manual Download

The patch file is available in the BC-250 Discord server:
- **Forum thread:** `bc250-resources` → "Increased GPU frequency range kernel patch"
- **File:** `linux-6.12-bc250-freq.mypatch` (639 bytes)

### Manual Application

#### Method 1: Linux-TKG (Recommended)

```bash
# Clone linux-tkg repository
git clone https://github.com/Frogging-Family/linux-tkg.git
cd linux-tkg

# Create userpatches folder
mkdir linux612-tkg-userpatches

# Download and place patch
# (Get linux-6.12-bc250-freq.mypatch from Discord)
mv linux-6.12-bc250-freq.mypatch linux612-tkg-userpatches/

# Compile kernel
# Follow linux-tkg instructions
# Press Y when asked to apply userpatches
```

**Benefits:**
- Automatic patch application
- Gaming-optimized tweaks included
- Well-tested compilation process

#### Method 2: AMDGPU Module Only (Fast Method)

Instead of 3-hour full kernel build, compile just the amdgpu module in 3 minutes:

**Trade-off:** Loads an out-of-tree module (taints kernel)

**Process:**
1. Download kernel source matching your running kernel
2. Apply patch to cyan_skillfish driver files
3. Build only amdgpu module
4. Load patched module

**Note:** The patch modifies only 3 values in the cyan_skillfish file.

#### Method 3: Distribution-Specific

**Fedora:**
```bash
# Follow Fedora's kernel patching guide
# Or install Bazzite kernel RPMs directly
```

**Arch Linux:**
```bash
# Use AUR packages (recommended)
yay -S linux-bazzite-bin
# Or linux-lts-amd-bc250-headers

# Or apply patch to PKGBUILD for custom builds
```

### Verifying the Patch

Check if patch is applied:

```bash
cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
```

Expected output should show frequency range **350-2230 MHz**.

### Troubleshooting: GPU Locked at 1500 MHz

If GPU remains at 1500 MHz after installing patched kernel:

1. **Verify patch applied:**
   - Try setting frequencies in unpatched range (1000-2000 MHz) via governor
   - If works: Patch NOT applied, rebuild kernel
   - If doesn't work: Governor issue, not patch

2. **Update governor configuration:**
   ```bash
   # For cyan-skillfish-governor-tt:
   sudo nano /etc/cyan-skillfish-governor-tt/config.toml
   # Update safe-points to include extended range frequencies

   sudo systemctl restart cyan-skillfish-governor-tt
   ```

### Recommended Settings

**Maximum Performance:**
```yaml
frequency:
  min: 1000
  max: 2230
voltage:
  min: 700
  max: 1060    # Adjust based on stability (1000-1060 typical)
```

**Balanced Profile:**
```yaml
frequency:
  min: 1000
  max: 2000
voltage:
  min: 700
  max: 1000
```

**Low Power Profile:**
```yaml
frequency:
  min: 350     # Deep idle
  max: 2000
voltage:
  min: 700     # Don't go below 700 mV
  max: 1000
```

### Compatibility

**Kernel Versions:**
- **Tested:** 6.12 (original), 6.15, 6.16.x
- **Expected:** Works on newer kernels (driver-level patch)

**Distribution Support:**

| Distribution | Status | Method |
|--------------|--------|--------|
| Bazzite | ✅ Pre-patched | Use stock kernel |
| Arch (AUR) | ✅ Pre-patched | Install from AUR |
| PikaOS | ✅ Pre-patched | Use stock kernel |
| Fedora | ⚠️ Manual | linux-tkg or Bazzite RPM |
| CachyOS | ⚠️ Manual | May be included in future |
| Manjaro | ⚠️ Manual | No pre-built packages |
| Debian | ⚠️ Manual | Patch kernel source |

### Warnings

!!!danger "Cooling Required at 2230 MHz"
    2230 MHz @ 1050 mV generates significantly more heat than stock. Ensure high static pressure cooling (Arctic P12 Max or Noctua NF-A12x25). Temperature monitoring essential.

!!!warning "Silicon Lottery"
    Not all chips stable at 2000 MHz @ 1000 mV. Voltage requirements for 2230 MHz vary (1000-1060 mV typical). Test stability incrementally.

!!!warning "PSU Requirements"
    Maximum power draw can exceed 320W with full GPU load (Furmark). Ensure adequate PSU capacity. Games typically draw 220-250W max.

!!!danger "Minimum Voltage: 700mV"
    Setting minimum voltage below 700mV locks the GPU to 1500MHz. Always keep min voltage ≥700mV.

**Do NOT:**
- Use extended voltage patch (600-1300 mV) - unnecessary and risky
- Go below 700 mV (locks GPU to 1500MHz, defeats the purpose of overclocking)
- Exceed 1129 mV without careful testing (hardware degradation risk)
- Use Smokeless_UMAF — may cause permanent damage to the board

---

## CPU Overclocking & Undervolting (SMU Tool)

A community-developed SMU (System Management Unit) tool enables CPU overclocking and undervolting on the BC-250.

**Repository:** [bc250-collective/bc250_smu_oc](https://github.com/bc250-collective/bc250_smu_oc)
**Created by:** mrfrakes and dantistnfs (via SMU reverse engineering)

### What It Does

- Overclock all 6 CPU cores (up to 4 GHz @ 1275 mV reported)
- Undervolt the CPU for lower power and temperatures
- Detect viable overclock for your specific board
- Apply overclock automatically on startup

### Requirements

- **Cooling:** Good active cooling is essential — CPU overclocking adds significant heat on top of GPU load
- **PSU:** Ensure adequate headroom (300W+ recommended when combined with GPU overclock)
- **Testing:** Use incremental steps and stress test thoroughly

!!!warning "Silicon Lottery"
    Not all BC-250 boards will reach 4 GHz. Use the tool's detection features to find your chip's safe limits. Always test stability under sustained load before committing settings.

---

## Safe Overclocking Limits

Start at 2000 MHz @ 1000 mV and work up. Stability varies by board (silicon lottery). General guidance:

- **2000 MHz @ 1000 mV** — safe starting point for most boards
- **2100-2175 MHz @ 1025-1050 mV** — works on many boards, test thoroughly
- **2230 MHz @ 1035-1060 mV** — maximum hardware limit, requires good cooling

## Manual Overclocking

### Method 1: Via sysfs (Temporary)

**Test frequencies before making permanent:**

```bash
# View current frequency/voltage table
cat /sys/class/drm/card1/device/pp_od_clk_voltage

# Set voltage and frequency
# Format: vc <level> <frequency_MHz> <voltage_mV>
echo "vc 0 2100 1025" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage

# Commit changes
echo "c" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage

# Verify
cat /sys/class/drm/card1/device/pp_od_clk_voltage
```

**Test with benchmark:**
```bash
# Run Unigine Superposition or game for 30+ minutes
# Monitor temperatures and stability
```

### Method 2: Via Governor Config (Permanent)

**Edit config:**
```bash
sudo nano /etc/cyan-skillfish-governor-tt/config.toml
```

**Multi-voltage point configuration:**
```toml
safe-points = [
    [1000, 700],   # 1000 MHz @ 700 mV
    [1500, 900],   # 1500 MHz @ 900 mV
    [2000, 1000],  # 2000 MHz @ 1000 mV
    [2100, 1025],  # 2100 MHz @ 1025 mV
    [2175, 1050],  # 2175 MHz @ 1050 mV (overclock)
]
```

**Restart governor:**
```bash
sudo systemctl restart cyan-skillfish-governor-tt
```

## Testing Stability

### Stress Testing

**1. Unigine Superposition (Recommended):**
```bash
# Download from: https://benchmark.unigine.com/superposition

# Run benchmark
# 1080p Extreme preset
# Loop for 30+ minutes
```

**2. Gaming Stress Test:**
- Run demanding game for 60+ minutes
- Test different scenarios (loading screens, open world, combat)

**3. Monitor During Testing:**
```bash
# Watch sensors
watch -n 1 sensors

# Check for:
# - GPU temp < 85°C (< 80°C better)
# - No crashes or artifacts
# - Stable frame rates
```

### Signs of Instability

**Reduce frequency or increase voltage if:**
- System crashes/freezes
- Visual artifacts (flickering, black textures, corrupted graphics)
- Performance degradation
- GPU temp > 90°C

## Performance Notes

Overclocking from 2000 to 2230 MHz gives diminishing returns — expect single-digit percentage gains in most games, with significantly higher power draw and temperatures. Most users will see better results from stable governor tuning at 2000-2100 MHz than pushing for maximum clocks.

## Voltage Tuning

### Undervolting for Efficiency

**Goal:** Reduce voltage while maintaining frequency

**Process:**
1. Start at stock frequency (2000 MHz @ 1000mV)
2. Reduce voltage by 25mV (to 975mV)
3. Test stability
4. If stable, reduce another 25mV
5. If crashes, increase by 25mV
6. Find minimum stable voltage

Results vary by board. Test stability thoroughly at each step.

### Overvolting for Higher Clocks

**Use Case:** Pushing beyond 2100 MHz

**Caution:**
- Higher voltage = more heat and power
- Diminishing stability gains above 1050mV
- Risk of hardware degradation

**Safe limits:**
- **Maximum recommended:** 1085mV
- **Absolute maximum:** 1100mV (high risk)

## Cooling and Power

Higher clocks mean more heat and power draw. Keep GPU below 85°C under load. See [Cooling Solutions](../hardware/cooling.md) and [Power Requirements](../hardware/power.md) for details.

**Before overclocking:** Ensure you have a governor installed, adequate cooling, and a PSU rated for 250W+ on 12V rail. Increase frequency in 50-100 MHz steps, testing stability for 30+ minutes at each step.

## Troubleshooting Overclocking Issues

### System Crashes Immediately

**Cause:** Voltage too low for frequency

**Solution:**
- Increase voltage by 25-50mV
- OR reduce frequency by 50-100 MHz

### Crashes After 10-30 Minutes

**Cause:** Thermal throttling or marginal stability

**Solutions:**
1. Check temperatures (should be < 85°C)
2. Improve cooling
3. Increase voltage slightly
4. Reduce frequency

### Visual Artifacts

**Symptoms:**
- Black textures
- Flickering
- Corrupted graphics

**Causes:**
- Memory clock too high (not adjustable)
- Unstable frequency/voltage
- Overheating

**Solutions:**
- Reduce GPU frequency
- Add RADV_DEBUG=nohiz to environment
- Check cooling

### Performance Worse After Overclock

**Possible Causes:**
1. Thermal throttling (too hot)
2. Power limit hit (PSU insufficient)
3. Hitting frequency that triggers instability

**Solutions:**
- Check temperatures
- Verify PSU wattage
- Try different frequency

### Governor Not Applying Overclock

**Check:**
```bash
# Verify governor running
systemctl status cyan-skillfish-governor-smu

# Restart governor
sudo systemctl restart cyan-skillfish-governor-smu

# Check applied settings
cat /sys/class/drm/card1/device/pp_od_clk_voltage
```

## Advanced: Multiple Voltage Points

For Cyan-Skillfish Governor or manual tuning:

**Create voltage curve:**

1. Test each frequency point:
```bash
# Test 1500 MHz
echo "vc 0 1500 875" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage
echo "c" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage
# Run benchmark, find minimum stable voltage

# Test 1750 MHz
echo "vc 0 1750 950" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage
echo "c" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage
# Run benchmark, find minimum stable voltage

# Repeat for 2000, 2100, 2175, 2230 MHz
```

2. Record stable voltage for each frequency
3. Add to governor config

**Benefits:**
- More efficient (lower voltage at mid frequencies)
- Better temperature management
- Smoother frequency transitions

**Time Investment:**
- Testing each frequency point thoroughly
- Recording stable voltages
- Validating with benchmarks

## Community Resources

- [GPU Governor Setup](../system/governor.md) — start here
- [cyan-skillfish-governor-smu](https://github.com/filippor/cyan-skillfish-governor/tree/smu) — SMU governor (no kernel patch needed)
- [PS5GPU-BC250](https://github.com/ZEROAESQUERDA/PS5GPU-BC250) — GUI GPU controller
- [NexGen3D SteamMachine Scripts](https://github.com/NexGen-3D-Printing/SteamMachine) — automated setup for Bazzite
- [DeathStalker Grimoire](https://github.com/DeathStalker471/bc250theGrimoire) — community step-by-step guide
- [Cooling Solutions](../hardware/cooling.md)
- [Power Requirements](../hardware/power.md)
- [BIOS Recovery](recovery.md)

---

**Last Updated:** 2026-03-18
