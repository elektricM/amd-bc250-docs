# Power Management Guide

Complete power management reference for the BC-250, covering consumption characteristics, PSU requirements, power tuning, and optimization strategies.

---

## Power Consumption Overview

### Measured Power Draw

The BC-250's power consumption varies significantly based on workload and configuration:

| Scenario | Power Draw | Notes |
|----------|------------|-------|
| **Idle (No Governor)** | 105W | Stock configuration, no optimization |
| **Idle (With Governor)** | 85W | 20W savings with oberon-governor |
| **Idle (Optimized)** | 55W | Debian + governor + undervolting |
| **Desktop Use** | 60-85W | Light browsing, system tasks |
| **Gaming (Standard)** | 150-200W | Most games at 1080p |
| **Gaming (RT Enabled)** | 235W | Cyberpunk 2077, high settings + RT |
| **Benchmark (Furmark)** | 250-320W | Stress test, not realistic workload |
| **Full Load (Typical)** | 195W | Sustained heavy workload |

**Component Breakdown** (from hardware testing):
- CPU + GPU at idle: ~31W
- RAM + Memory Controller: ~35W+
- Other board components: ~27W
- Total idle: ~93W (without optimization)

### TDP Specifications

**Official TDP:** 220W

**Realistic Ranges:**
- Normal gaming: 150-200W
- Maximum practical: 235W
- Theoretical maximum: 300W+ (requires extreme cooling, not recommended)

---

## PSU Requirements

### Minimum Requirements

**Power Capacity:**
- Minimum 12V rail: 220W (18.3A @ 12V)
- Recommended: 250W+ on 12V rail for headroom
- Heavy users: 300W+ for overclocking

**Connector:**
- PCIe 8-pin (6+2) connector required
- Some users report 6-pin works but not recommended
- Ground and sense pins important for stability

### Calculating PSU Capacity

```
Watts = Volts × Amps
Required Amps = 220W ÷ 12V = 18.3A minimum
Recommended Amps = 25A for safety margin
```

**Multi-Rail PSU Warning:** If your PSU has multiple 12V rails, ensure a single rail can provide the full wattage. A 300W PSU split across three 100W rails will NOT work.

### Recommended PSU Options

**Budget Options:**

1. **FSP FSP500-30AS** (Flex ATX)
   - 500W total, sufficient 12V rail
   - Compact form factor
   - ~$50-60 new, $10-30 on eBay
   - Coil whine reported on some units

2. **Mean Well LOP-300-12**
   - 300W @ 12V (25A)
   - Open frame design
   - Requires custom cabling
   - Silent operation

3. **Dell 220W-330W Bricks** (repurposed)
   - Available cheap from surplus
   - Requires barrel to 8-pin adapter
   - Some units insufficient for peak loads

**Standard ATX Options:**

- Any quality 450W+ ATX PSU with single 12V rail
- Seasonic, EVGA, Corsair, Be Quiet recommended
- Ensure 12V rail provides 20A+

**Server PSU (Advanced):**
- 1200W+ server PSUs available cheap
- Require fan modification (very loud stock)
- Breakout boards needed
- Best for multi-board setups

### PSU Safety Warnings

**DO NOT use these adapters:**
- SATA to 8-pin (SATA limited to 55W - fire hazard)
- Molex to 8-pin (unless PSU explicitly rated for it)
- Cheap no-name adapters from marketplace sellers

**Signs of insufficient PSU:**
- Random shutdowns under load
- Fan speed fluctuations
- Crashes during demanding games
- OCP (overcurrent protection) trips

---

## Power Limit Configuration

### Using GPU Governor

The governor is essential for power management and efficiency.

**Oberon Governor Configuration:**

Default configuration file: `/etc/oberon-config.yaml`

```yaml
opps:
  - frequency:
    - min: 1000    # MHz
    - max: 2000    # MHz (safe for most boards)
  - voltage:
    - min: 700     # mV
    - max: 1000    # mV
```

**Power Savings Configuration (Lower Consumption):**

```yaml
opps:
  - frequency:
    - min: 1000
    - max: 2000
  - voltage:
    - min: 700
    - max: 950     # Lower max voltage
```

Apply changes:
```bash
sudo systemctl restart oberon-governor
```

**Cyan Skillfish Governor Configuration:**

More granular control with multiple voltage/frequency points.

Default location: `/etc/cyan-skillfish-governor/config.toml`

```toml
# Multiple safe-points for precise voltage control
[[safe-points]]
frequency = 350
voltage = 700

[[safe-points]]
frequency = 1000
voltage = 700

[[safe-points]]
frequency = 1500
voltage = 850

[[safe-points]]
frequency = 2000
voltage = 1000

# Load targets (70-95% default)
[load_target]
min = 70
max = 95
```

Apply changes:
```bash
sudo systemctl restart cyan-skillfish-governor
```

### Manual Power Limiting (Advanced)

**Set specific frequency and voltage:**

```bash
# Stop governor first
sudo systemctl stop oberon-governor

# Set custom values
echo "vc 0 1800 950" > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
echo "c" > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

# Test for stability before setting governor config
```

---

## Undervolting for Power Savings

### Benefits of Undervolting

- Reduced power consumption (10-30W savings possible)
- Lower temperatures (5-10°C reduction)
- Quieter operation (fans run slower)
- No performance loss if done correctly

### Safe Undervolting Process

**Starting Point (Conservative):**

| Frequency | Stock Voltage | Undervolt Target |
|-----------|---------------|------------------|
| 1000 MHz | 700 mV | 700 mV (no change) |
| 1500 MHz | 900 mV | 850 mV (-50 mV) |
| 2000 MHz | 1000 mV | 950 mV (-50 mV) |

**Testing Procedure:**

1. Configure governor with lower voltage
2. Run stability test (gaming for 30+ minutes)
3. Monitor for crashes, artifacts, black screens
4. If stable, reduce voltage by another 25 mV
5. If unstable, increase voltage by 25 mV
6. Repeat until you find stable minimum

**Stability Test Commands:**

```bash
# Run vkmark for GPU stress
vkmark

# Or run demanding game
steam steam://rungameid/1091500  # Cyberpunk 2077
```

**Example Undervolt Results (User Reported):**

- 2000 MHz @ 950 mV: Stable, ~163W gaming (Cyberpunk)
- 2230 MHz @ 1000 mV: Stable, ~200W gaming
- 2230 MHz @ 1035 mV: Unstable, required 1050+ mV

### Determining Voltage Requirements

**Trial and Error Method:**

1. Start with frequency you want (e.g., 1800 MHz)
2. Set voltage to 1000 mV (safe default)
3. Test stability
4. Lower voltage in 25 mV increments
5. When crashes occur, increase 25 mV and mark as stable

**Interpolation Method (Advanced):**

If you know two stable points, you can interpolate:
- 1000 MHz @ 700 mV = stable
- 2000 MHz @ 1000 mV = stable
- 1500 MHz @ 850 mV = likely safe (linear interpolation)

**Note:** Voltage curves are NOT perfectly linear. Silicon lottery means every chip is different. Always test thoroughly.

---

## Idle Power Optimization

### Reducing Idle Consumption

**Step 1: Install Governor**

Without governor: 105W idle
With governor: 85W idle (20W savings)

```bash
# Install oberon-governor (Fedora/Bazzite)
sudo dnf copr enable @exotic-soc/oberon-governor
sudo dnf install oberon-governor
sudo systemctl enable --now oberon-governor
```

**Step 2: Optimize Governor Settings**

Edit `/etc/oberon-config.yaml` for lower idle power:

```yaml
opps:
  - frequency:
    - min: 1000    # Allow GPU to idle lower
    - max: 2000
  - voltage:
    - min: 700     # Minimum safe voltage
    - max: 950     # Reduced max voltage
```

Result: 60-70W idle possible

**Step 3: CPU Power Management (Advanced)**

Enable CPU idle states:

```bash
# Check current CPU governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Set to powersave or schedutil
echo "powersave" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Check idle states
sudo cpupower idle-info
```

Result: Additional 2-3W savings (65W idle achievable)

**Best Case Scenario:**

User report: 55W idle on Debian with governor, undervolting, and proper kernel configuration.

### GPU Sleep States (Experimental)

**Current Limitation:** GPU sleep states are governed by SMU (System Management Unit) firmware, which is locked in current BIOS versions.

**Potential Future Improvement:**
- Linux amdgpu driver supports SMU sleep messages for BIOS 3.00+
- Requires reverse-engineering or BIOS update
- Could potentially reduce idle below 50W

Community members actively investigating this area.

---

## Power Measurement and Monitoring

### Software Monitoring

**1. Built-in Sensors**

View all power and voltage sensors:

```bash
sensors
```

Example output:
```
amdgpu-pci-0100
Adapter: PCI adapter
vddgfx:      906.00 mV    # GPU voltage
vddnb:       824.00 mV    # Northbridge voltage
edge:         +63.0°C     # GPU temperature
PPT:          55.12 W     # Package Power Tracking (GPU power)

nct6686-isa-0a20
Adapter: ISA adapter
VIN0:             832.00 mV
VIN1:               1.02 V
VIN2:             976.00 mV
VIN6:               1.39 V
VIN7:             928.00 mV
VIN16:            896.00 mV
```

**2. Watch Real-Time Power**

```bash
watch -n 1 'sensors | grep -A 4 amdgpu'
```

**3. GPU Frequency and Voltage Monitoring**

Check current GPU state:

```bash
cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
```

**4. GUI Tools**

**CoolerControl** (Desktop):
```bash
# Bazzite/Fedora
ujust install-coolercontrol

# After reboot
coolercontrol
```

**Mangohud** (In-Game Overlay):
```bash
# Install
flatpak install flathub org.freedesktop.Platform.VulkanLayer.MangoHud

# Enable in Steam
Launch Options: mangohud %command%
```

### Hardware Monitoring

**Kill-A-Watt or Smart Plug:**

Best method for total system power measurement:

1. **Sonoff S31 with Tasmota**: Smart plug with power metering
2. **Kill-A-Watt P3**: Standard power meter
3. **TP-Link Kasa Smart Plug**: App-based monitoring

Measures actual wall power including PSU inefficiency.

**Example Measurements:**
- Idle: 85W (governor active)
- Gaming: 150-200W
- Cyberpunk RT: 235W peak
- Furmark: 250-320W (stress test)

---

## Governor Voltage/Frequency Curves

### Understanding Voltage Curves

The relationship between frequency and voltage is NOT linear:

```
Low frequencies (350-1000 MHz): Voltage can stay at 700 mV
Mid frequencies (1000-1500 MHz): Voltage needs to increase slightly
High frequencies (1500-2000 MHz): Voltage scales more steeply
Max frequencies (2000-2300 MHz): Voltage requirements vary by silicon
```

### Example Safe Curves

**Conservative (Maximum Stability):**

```yaml
# Oberon Governor
opps:
  - frequency:
    - min: 1000
    - max: 2000
  - voltage:
    - min: 700
    - max: 1000
```

**Optimized (Good Balance):**

```toml
# Cyan Skillfish Governor
[[safe-points]]
frequency = 350
voltage = 700

[[safe-points]]
frequency = 1000
voltage = 700

[[safe-points]]
frequency = 1500
voltage = 850

[[safe-points]]
frequency = 2000
voltage = 950
```

**Performance (Higher Power):**

```toml
[[safe-points]]
frequency = 350
voltage = 700

[[safe-points]]
frequency = 1500
voltage = 900

[[safe-points]]
frequency = 2000
voltage = 1000

[[safe-points]]
frequency = 2230
voltage = 1035
```

### Governor Behavior Comparison

**Oberon Governor:**
- Binary mode: Switches between min and max frequency
- Set point: 20-40% GPU load (with hysteresis)
- Response time: 100 ms to burst to max
- CPU usage: 0.4% idle, 0.4% under load
- Simple, stable, proven

**Cyan Skillfish Governor:**
- Continuous adjustment between multiple frequency steps
- Set point: 70-95% GPU load (configurable)
- Response time: 20-24 ms to burst to max
- CPU usage: 0.9% idle, 1.3% under load
- Granular control, more responsive

**Which to Choose:**

- **Oberon**: Better for stability, lower CPU overhead, simpler config
- **Cyan Skillfish**: Better for power efficiency, smoother performance, more tuning

---

## Power-Related Stability Issues

### Common Problems and Solutions

**1. Random Shutdowns Under Load**

**Symptoms:**
- System cuts power during gaming
- Crashes during benchmarks
- No error messages, just power loss

**Causes:**
- Insufficient PSU capacity
- PSU overcurrent protection (OCP) triggered
- Voltage too low for frequency

**Solutions:**
```bash
# Option 1: Reduce max frequency
# Edit /etc/oberon-config.yaml
opps:
  - frequency:
    - max: 1800  # Reduced from 2000

# Option 2: Increase voltage
  - voltage:
    - max: 1025  # Increased from 1000

# Option 3: Upgrade PSU to 300W+ on 12V rail
```

**2. Fan Speed Fluctuations**

**Symptoms:**
- Fans slow down during heavy load
- Fan speed changes unpredictably

**Causes:**
- PSU unable to maintain stable 12V under load
- Voltage droop on 12V rail

**Solutions:**
- Verify PSU 12V rail capacity (should be 20A+)
- Upgrade to higher wattage PSU
- Check 8-pin connector seating
- Reduce power consumption via undervolting

**3. System Instability (Crashes/Artifacts)**

**Symptoms:**
- Random crashes during gaming
- Screen artifacts, glitches
- Black screen flashes

**Causes:**
- Voltage too low for set frequency
- Unstable undervolt

**Solutions:**
```bash
# Increase voltage in 25 mV steps
# Edit /etc/oberon-config.yaml
  - voltage:
    - max: 1025  # Increase until stable
```

**4. High Idle Power (>100W)**

**Symptoms:**
- Board draws excessive power at desktop
- No governor active or not working

**Causes:**
- Governor not installed or not running
- GPU stuck at max frequency

**Solutions:**
```bash
# Check governor status
systemctl status oberon-governor

# If not running
sudo systemctl enable --now oberon-governor

# Verify GPU frequency scaling
cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_dpm_sclk

# Should show active lower states at idle
```

**5. Performance Throttling at Temperature**

**Symptoms:**
- Performance drops after extended gaming
- Framerate becomes unstable after 30+ minutes

**Causes:**
- Thermal throttling (>85°C)
- Governor reducing frequency due to temperature

**Solutions:**
- Improve cooling (see cooling guide)
- Reduce max frequency slightly
- Increase fan speed (BIOS or software)
- Apply better thermal paste

**Temporary workaround (not recommended long-term):**
```yaml
# Force governor to maintain frequency even when hot
# Edit /etc/oberon-config.yaml to set min = max
opps:
  - frequency:
    - min: 2000
    - max: 2000
```

---

## Advanced Power Tuning

### TDP Modification (Experimental)

**Warning:** Not officially supported. May void warranty or damage hardware.

**Theoretical Options:**
- SMU firmware modification (requires reverse-engineering)
- BIOS-level TDP limits (locked in current BIOS)
- Software power capping via kernel modules

**Current Status:** Community investigating, no reliable method yet.

### Memory Power Management

**GDDR6 Memory Characteristics:**

- Memory draws significant power (~35W+ at idle)
- No dynamic memory clocking in current drivers
- PS5 uses dynamic memory clocking (not available on BC-250)

**Potential Future Improvements:**
- Driver-level memory clock scaling
- BIOS update to enable dynamic VRAM clocking
- Could save 10-20W at idle if implemented

### Multi-Board Power Optimization

**For Users Running Multiple BC-250s:**

Power consumption scales linearly:
- 1 board: 85W idle, 195W load
- 12 boards: 1020W idle (85W × 12), 2340W load (195W × 12)

**Optimization Strategies:**

1. **Governor on all boards**: 20W savings per board = 240W total
2. **Aggressive undervolting**: 10-20W savings per board = 120-240W total
3. **Idle optimization**: Target 60W per board = 720W total idle

**Power Budget Planning:**
```
12 boards × 85W idle = 1020W
At $0.12/kWh: 1.02 kW × 24h × 30 days × $0.12 = $88/month idle
With optimization (60W/board): $63/month = $25/month savings
```

---

## Power Management Best Practices

### Recommended Configuration

**For Gaming (Balance Performance/Power):**

```yaml
# /etc/oberon-config.yaml
opps:
  - frequency:
    - min: 1000
    - max: 2000
  - voltage:
    - min: 700
    - max: 950
```

Expected results:
- Idle: 70-85W
- Gaming: 150-180W
- Temperatures: 65-75°C

**For Power Efficiency (Low Consumption):**

```toml
# /etc/cyan-skillfish-governor/config.toml
[[safe-points]]
frequency = 350
voltage = 700

[[safe-points]]
frequency = 1000
voltage = 700

[[safe-points]]
frequency = 1500
voltage = 850

[[safe-points]]
frequency = 1800
voltage = 950

[load_target]
min = 75
max = 90
```

Expected results:
- Idle: 55-65W
- Gaming: 140-170W
- Temperatures: 60-70°C

**For Performance (Maximum FPS):**

```yaml
opps:
  - frequency:
    - min: 2000
    - max: 2230
  - voltage:
    - min: 1000
    - max: 1050
```

Expected results:
- Idle: 85-95W (GPU idles higher)
- Gaming: 200-235W
- Temperatures: 75-85°C

### Monitoring Checklist

**After any power configuration change, verify:**

1. Idle power draw (should be <85W with governor)
2. Voltage scaling (check with `sensors`)
3. Temperature under load (should be <85°C)
4. System stability (30+ minute gaming test)
5. No PSU issues (fan speed stable, no shutdowns)

### Safety Guidelines

**DO:**
- Start with conservative voltages
- Test stability thoroughly after changes
- Monitor temperatures during stress tests
- Keep max voltage under 1100 mV
- Use quality PSU with sufficient capacity

**DON'T:**
- Set voltage above 1100 mV long-term
- Run stress tests for hours at max power
- Use inadequate cooling with high power configs
- Ignore throttling or instability
- Mix high voltage with high temperature (>85°C)

---

## Troubleshooting Tools

### Power Issue Diagnostics

**Check Governor Status:**
```bash
systemctl status oberon-governor
# or
systemctl status cyan-skillfish-governor
```

**Check Current Power State:**
```bash
# GPU voltage and frequency
cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

# Current GPU power
sensors | grep PPT

# GPU frequency states
cat /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_dpm_sclk
```

**Monitor Real-Time:**
```bash
# Power and temperature
watch -n 1 'sensors | grep -A 5 amdgpu'

# System power (if using smart plug)
# Check plug's web interface or app
```

**Stress Test:**
```bash
# Lightweight GPU load
vkmark

# Heavy stress test (monitor temperatures!)
glmark2

# Check power during test
sensors | grep PPT
```

### Common Issues Quick Reference

| Issue | Likely Cause | Fix |
|-------|--------------|-----|
| >100W idle | No governor | Install and enable governor |
| Random shutdowns | Insufficient PSU | Upgrade PSU or reduce voltage |
| High temps + high power | Undervolt needed | Reduce max voltage by 50 mV |
| Fan speed drops | PSU voltage droop | Upgrade PSU capacity |
| Unstable after undervolt | Voltage too low | Increase voltage by 25 mV |
| Governor not working | Service not running | `systemctl enable --now oberon-governor` |

---

## Additional Resources

**Governor Projects:**
- [Oberon Governor (Original)](https://gitlab.com/TuxThePenguin0/oberon-governor)
- [Oberon Governor (Fork)](https://github.com/filippor/oberon-governor)
- [Cyan Skillfish Governor](https://github.com/Magnap/cyan-skillfish-governor)

**Power Monitoring:**
- [CoolerControl](https://gitlab.com/coolercontrol/coolercontrol)
- [MangoHud](https://github.com/flightlessmango/MangoHud)

**Community Resources:**
- Discord Server: BC-250 Community
- GitHub Documentation: [BC-250 Docs](https://github.com/mothenjoyer69/bc250-documentation)

---

**Last Updated:** 2025-11-21
**Contributors:** Community testing and reporting
