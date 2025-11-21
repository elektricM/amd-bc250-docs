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
