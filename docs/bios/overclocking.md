# BIOS Overclocking Guide

Advanced guide to overclocking the BC-250 GPU via BIOS settings and manual configuration.

!!!danger "Advanced Users Only"
    Overclocking can cause system instability, crashes, and potential hardware damage. Only proceed if you understand the risks.

## Overclocking Overview

### What Can Be Overclocked

- **GPU Frequency:** 1500 MHz (locked) → 2000-2230 MHz (with governor/kernel patch)
- **GPU Voltage:** 700-1100 mV
- **Memory:** Not user-adjustable (GDDR6 runs at fixed speed)

### Requirements

1. **Kernel Patch:** Extended frequency range (350-2230 MHz)
   - OR use distribution with patch included (Bazzite, PikaOS)
2. **GPU Governor:** For automatic frequency scaling
3. **Adequate Cooling:** Arctic P12 Max or better
4. **Quality PSU:** 250W+ on 12V rail

---

## GPU Frequency Range Kernel Patch

The GPU frequency range kernel patch is a community-developed modification that extends the BC-250's GPU operating frequencies beyond the default software-imposed limits to match the actual hardware capabilities.

**Created by:** ViRazY ([GitHub](https://github.com/Vinjul1704))

### What the Patch Does

**Frequency Range Extension:**
- **Default range:** 1000 MHz - 2000 MHz
- **Patched range:** 350 MHz - 2230 MHz
- **Hardware limit:** 2230 MHz (actual silicon limit)

**Voltage Range:**
- **Default:** 700 mV - 1129 mV (unchanged in standard patch)
- **Experimental extended patch:** 600 mV - 1300 mV (NOT RECOMMENDED)

!!!warning "Use Standard Patch Only"
    The extended voltage patch (600-1300 mV) is NOT recommended. The standard patch is sufficient for reaching 2230 MHz, and extended voltages risk hardware degradation.

### Why It's Needed

Without the patch, the BC-250's GPU performance is artificially limited:

1. **Performance ceiling:** Stock 2000 MHz maximum prevents full potential
2. **Power efficiency:** Cannot downclock below 1000 MHz for idle
3. **Governor compatibility:** Cannot set frequencies outside 1000-2000 MHz range

### Performance Gains

**Gaming Performance:**
- **Star Wars Battlefront 2:** 80-85 FPS → 120-130 FPS (+50%)
- Users report "huge performance boost" across multiple titles
- Maximum performance at 2230 MHz @ 1000-1060 mV

**Power Efficiency:**
- Idle downclocking to 350 MHz saves ~1-9W vs stock
- Total system power can reach ~69W idle (vs ~78W stock)
- Limited savings due to 700 mV floor being safe minimum

### How to Obtain the Patch

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
   sudo nano /etc/oberon-config.yaml

   # Update to use extended range
   frequency:
     min: 350    # Was 1000
     max: 2230   # Was 2000

   sudo systemctl restart oberon-governor
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

**Do NOT:**
- Use extended voltage patch (600-1300 mV) - unnecessary and risky
- Go below 700 mV (unstable, minimal power savings)
- Exceed 1129 mV without careful testing (hardware degradation risk)

---

## Safe Overclocking Limits

### Community-Tested Safe Limits

| Frequency | Voltage | Stability | Power Draw | Cooling Required |
|-----------|---------|-----------|------------|------------------|
| 2000 MHz | 1000 mV | ✅ Safe (all boards) | 190-200W | Stock + fan |
| 2100 MHz | 1025 mV | ✅ Good (most boards) | 200-210W | Arctic P12 |
| 2175 MHz | 1025 mV | ⚠️ Some boards | 210-220W | Arctic P12 Max |
| 2230 MHz | 1035-1050 mV | ⚠️ Best boards only | 220-235W | Dual fans |
| 2230 MHz | 1085 mV | ⚠️ High risk | 250W+ | Excellent cooling |

### Known Unstable Frequencies

!!!warning "980 MHz Instability"
    **980 MHz is unstable** across all boards tested. Avoid this frequency.

## Manual Overclocking

### Method 1: Via sysfs (Temporary)

**Test frequencies before making permanent:**

```bash
# View current frequency/voltage table
cat /sys/class/drm/card0/device/pp_od_clk_voltage

# Set voltage and frequency
# Format: vc <level> <frequency_MHz> <voltage_mV>
echo "vc 0 2100 1025" | sudo tee /sys/class/drm/card0/device/pp_od_clk_voltage

# Commit changes
echo "c" | sudo tee /sys/class/drm/card0/device/pp_od_clk_voltage

# Verify
cat /sys/class/drm/card0/device/pp_od_clk_voltage
```

**Test with benchmark:**
```bash
# Run Unigine Superposition or game for 30+ minutes
# Monitor temperatures and stability
```

### Method 2: Via Governor Config (Permanent)

**Edit governor config:**
```bash
sudo nano /etc/oberon-config.yaml
```

**Safe overclock example:**
```yaml
opps:
  frequency:
    min: 1000    # 1000 MHz idle
    max: 2100    # 2100 MHz load
  voltage:
    min: 700     # 700 mV idle
    max: 1025    # 1025 mV load
```

**Aggressive overclock example:**
```yaml
opps:
  frequency:
    min: 1000
    max: 2175    # 2175 MHz load
  voltage:
    min: 700
    max: 1050    # Higher voltage for stability
```

**Apply changes:**
```bash
sudo systemctl restart oberon-governor
```

### Method 3: Cyan-Skillfish Governor (Multi-Point)

**Edit config:**
```bash
sudo nano /etc/cyan-skillfish-governor/config.toml
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
sudo systemctl restart cyan-skillfish-governor
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

## Performance Gains

### Benchmark Results

**Unigine Superposition (1080p Extreme):**

| Config | Score | FPS | GPU Temp | Power |
|--------|-------|-----|----------|-------|
| Stock 2000 MHz @ 1000mV | 3888 | ~57 | 76°C | 190W |
| Patch 2230 MHz @ 1035mV | 4118 | ~60 | 86°C | 235W |
| Gain | +230 | +3 | +10°C | +45W |

**Cyberpunk 2077 (1080p High):**

| Config | FPS | Notes |
|--------|-----|-------|
| Stock 2000 MHz | 57.66 | Without overlay |
| OC 2220 MHz | 60.82 | +5.5% performance |

### Performance vs Power Trade-Off

**Analysis:**
- 2000 MHz → 2230 MHz: +11.5% frequency
- Performance gain: +5-6% (not linear due to other bottlenecks)
- Power increase: +20-25%
- Temperature increase: +10°C

**Conclusion:** Diminishing returns above 2100 MHz for most users

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

**Example Results:**
- Some boards stable at 2000 MHz @ 950mV (-50mV)
- Reduces power by ~10-15W
- Lowers temperature by 5-7°C

### Overvolting for Higher Clocks

**Use Case:** Pushing beyond 2100 MHz

**Caution:**
- Higher voltage = more heat and power
- Diminishing stability gains above 1050mV
- Risk of hardware degradation

**Safe limits:**
- **Maximum recommended:** 1085mV
- **Absolute maximum:** 1100mV (high risk)

## Cooling Requirements

### Temperature Targets by Overclock

| Overclock Level | Max Temp | Recommended Cooling |
|-----------------|----------|---------------------|
| Stock (2000 MHz) | 76°C | Single Arctic P12 |
| Light OC (2100 MHz) | 80°C | Arctic P12 Max |
| Medium OC (2175 MHz) | 85°C | Dual fans or Arctic P12 Max |
| Heavy OC (2230 MHz) | 90°C | Dual high-performance fans |

!!!danger "Thermal Throttling"
    Above 85°C, performance may be reduced. Above 90°C, system instability is likely.

### Improving Cooling for Overclock

1. **Upgrade fan:** Arctic P12 Max (highest static pressure)
2. **Add second fan:** For VRM/memory cooling
3. **Improve airflow:** Remove case panels for testing
4. **Replace thermal paste:** Quality paste (Arctic MX-6, Kryonaut)
5. **Straighten fins:** Improve heatsink airflow
6. **Add thermal pads:** Cool memory chips on underside

## Power Supply Considerations

### Power Requirements by Overclock

| Overclock | Typical Power | Peak Power | PSU Recommendation |
|-----------|---------------|------------|---------------------|
| Stock 2000 MHz | 190W | 200W | 220W+ |
| 2100 MHz | 200W | 215W | 250W+ |
| 2175 MHz | 210W | 230W | 270W+ |
| 2230 MHz | 220W | 250W | 300W+ |

!!!warning "PSU Overload"
    Inadequate PSU will trigger over-current protection, causing crashes under load.

### Furmark Power Draw

**Unrealistic stress test:**
- Stock: 250W
- OC 2230 MHz @ 1085mV: **320W**

**Note:** No game reaches Furmark power levels. Use game testing.

## Overclocking Checklist

Before overclocking:

- [ ] BIOS flashed and configured (VRAM split set)
- [ ] Kernel 6.12-6.14 LTS (with frequency patch if needed)
- [ ] Mesa 25.1.3+ installed
- [ ] GPU governor installed and running
- [ ] Adequate cooling (Arctic P12 Max minimum)
- [ ] PSU rated for 250W+ on 12V rail
- [ ] Thermal monitoring set up (`sensors`)
- [ ] Backup of current config

During overclocking:

- [ ] Increase frequency in 50-100 MHz steps
- [ ] Test stability for 30+ minutes per step
- [ ] Monitor temperatures (< 85°C target)
- [ ] Monitor power draw
- [ ] Check for visual artifacts
- [ ] Verify no performance degradation

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
- Try different frequency (avoid 980 MHz range)

### Governor Not Applying Overclock

**Check:**
```bash
# Verify governor running
systemctl status oberon-governor

# Check config loaded
cat /etc/oberon-config.yaml

# Restart governor
sudo systemctl restart oberon-governor

# Check applied settings
cat /sys/class/drm/card0/device/pp_od_clk_voltage
```

## Advanced: Multiple Voltage Points

For Cyan-Skillfish Governor or manual tuning:

**Create voltage curve:**

1. Test each frequency point:
```bash
# Test 1500 MHz
echo "vc 0 1500 875" | sudo tee /sys/class/drm/card0/device/pp_od_clk_voltage
echo "c" | sudo tee /sys/class/drm/card0/device/pp_od_clk_voltage
# Run benchmark, find minimum stable voltage

# Test 1750 MHz
echo "vc 0 1750 950" | sudo tee /sys/class/drm/card0/device/pp_od_clk_voltage
echo "c" | sudo tee /sys/class/drm/card0/device/pp_od_clk_voltage
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

## See Also

- [GPU Governor Setup](../system/governor.md)
- [Cooling Solutions](../hardware/cooling.md)
- [Power Requirements](../hardware/power.md)
- [BIOS Recovery](recovery.md)
