# Power Management Guide

Complete power management reference for the BC-250, covering consumption characteristics, PSU requirements, power tuning, and optimization strategies.

---

## Power Consumption Overview

### Measured Power Draw

The BC-250's power consumption varies significantly based on workload and configuration:

| Scenario | Power Draw | Notes |
|----------|------------|-------|
| **Idle (No Governor)** | 105W | Stock configuration, no optimization |
| **Idle (With Governor)** | 85W | 20W savings with GPU governor |
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

**Cyan Skillfish Governor SMU Configuration (recommended):**

Default location: `/etc/cyan-skillfish-governor-smu/config.toml`

```toml
min_frequency = 1000  # MHz
max_frequency = 2000  # MHz (safe for most boards)
min_voltage = 700     # mV
max_voltage = 1000    # mV
```

**Power Savings Configuration (Lower Consumption):**

```toml
min_frequency = 1000
max_frequency = 2000
min_voltage = 700
max_voltage = 950     # Lower max voltage
```

Apply changes:
```bash
sudo systemctl restart cyan-skillfish-governor-smu
```

**SMU Governor Configuration (recommended):**

Default configuration file: `/etc/cyan-skillfish-governor-smu/config.toml` — see [Governor page](governor.md) for details.

**More Granular Control:**

Cyan Skillfish Governor SMU supports multiple voltage/frequency points.

Default location: `/etc/cyan-skillfish-governor-smu/config.toml`

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
sudo systemctl restart cyan-skillfish-governor-smu
```

### Manual Power Limiting (Advanced)

**Set specific frequency and voltage:**

```bash
# Stop governor first
sudo systemctl stop cyan-skillfish-governor-smu

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
# Install cyan-skillfish-governor-smu (Fedora/Bazzite)
sudo dnf copr enable filippor/bazzite
sudo dnf install cyan-skillfish-governor-smu
sudo systemctl enable --now cyan-skillfish-governor-smu
```

**Step 2: Optimize Governor Settings**

Edit `/etc/cyan-skillfish-governor-smu/config.toml` for lower idle power:

```toml
min_frequency = 1000  # Allow GPU to idle lower
max_frequency = 2000
min_voltage = 700     # Minimum safe voltage
max_voltage = 950     # Reduced max voltage
```

Result: 60-70W idle possible

**Step 3: CPU Power Management**

!!!info "CPU Frequency Scaling — Requires ACPI Fix"
    By default, the BC-250 does not expose CPU frequency scaling (no cpufreq interface). However, installing the [bc250-acpi-fix](https://github.com/bc250-collective/bc250-acpi-fix) SSDT-PST table enables standard Linux cpufreq with 8 P-states from 800 MHz to 3200 MHz. With the ACPI fix installed:

    ```bash
    # Set CPU governor (schedutil recommended for balanced power/performance)
    echo schedutil | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

    # Available: conservative, ondemand, userspace, powersave, performance, schedutil
    ```

    Without the ACPI fix, `cpupower frequency-set` will not work.

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

```toml
# Cyan Skillfish Governor SMU
min_frequency = 1000
max_frequency = 2000
min_voltage = 700
max_voltage = 1000
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

**Cyan Skillfish Governor SMU (recommended):**
- Continuous adjustment between multiple frequency steps
- Manages clocks through SMU firmware calls — no kernel patch needed
- Same performance characteristics as TT variant
- CPU usage: 0.9% idle, 1.3% under load

**Cyan Skillfish Governor TT (alternative):**
- Continuous adjustment between multiple frequency steps
- Set point: 70-95% GPU load (configurable)
- Response time: 20-24 ms to burst to max
- CPU usage: 0.9% idle, 1.3% under load
- Granular control, more responsive, thermal throttling aware

**Which to Choose:**

- **Cyan Skillfish SMU**: Recommended default — no kernel patches needed, bypasses kernel frequency/voltage limits via SMU firmware
- **Cyan Skillfish TT**: Alternative — thermal throttling support, requires kernel patch (pre-included in Bazzite)
- **Oberon**: Legacy option — migrate to SMU

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
```toml
# Option 1: Reduce max frequency
# Edit /etc/cyan-skillfish-governor-smu/config.toml
max_frequency = 1800  # Reduced from 2000

# Option 2: Increase voltage
max_voltage = 1025    # Increased from 1000

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
```toml
# Increase voltage in 25 mV steps
# Edit /etc/cyan-skillfish-governor-smu/config.toml
# Increase max voltage in your safe-points until stable
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
systemctl status cyan-skillfish-governor-smu

# If not running
sudo systemctl enable --now cyan-skillfish-governor-smu

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
# Edit /etc/cyan-skillfish-governor-smu/config.toml to set min = max
# min_frequency = 2000
# max_frequency = 2000
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

### Swap and ZRAM Optimization

The BC-250 has only 8GB usable system RAM (with 512MB dynamic VRAM), so swap configuration matters for gaming stability.

**Recommended: Disable ZRAM, enable zswap with lz4, create swap file**

ZRAM compressed swap conflicts with 512MB dynamic VRAM allocation and causes crashes in memory-hungry games (RDR2, Company of Heroes 3). The recommended approach is to replace ZRAM with zswap and a disk-backed swap file.

**Bazzite (rpm-ostree):**

```bash
# 1. Disable zram
echo "" | sudo tee /etc/systemd/zram-generator.conf

# 2. Create 16-32GB swap file (see https://docs.bazzite.gg/Advanced/swapfile/)
sudo btrfs filesystem mkswapfile --size 16G /swap/swapfile
sudo swapon /swap/swapfile
echo '/swap/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab

# 3. Enable lz4 compression in initramfs
rpm-ostree initramfs --enable \
  --arg=--add-drivers \
  --arg=lz4 \
  --arg=--add-drivers \
  --arg=lz4_compress

# 4. Enable zswap and set swappiness
rpm-ostree kargs --append-if-missing="zswap.enabled=1 zswap.max_pool_percent=25 zswap.compressor=lz4"

# 5. Reboot, then set swappiness
systemctl reboot
echo 180 | sudo tee /proc/sys/vm/swappiness
# Make permanent:
echo 'vm.swappiness=180' | sudo tee -a /etc/sysctl.d/99-swap.conf
```

**Fedora (dnf):**

```bash
# 1. Disable zram
sudo systemctl stop zram-swap
sudo systemctl disable zram-swap

# 2. Create swap file
sudo dd if=/dev/zero of=/swapfile bs=1G count=16
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab

# 3. Enable zswap
echo 'zswap.enabled=1 zswap.max_pool_percent=25 zswap.compressor=lz4' >> /etc/default/grub
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# 4. Set swappiness
echo 'vm.swappiness=180' | sudo tee /etc/sysctl.d/99-swap.conf
sudo sysctl -p /etc/sysctl.d/99-swap.conf
```

**Why vm.swappiness=180:**
With zswap enabled, a swappiness of 180 (above the default 60) tells the kernel to prefer compressing and swapping pages over dropping file caches. This improves memory management on low-RAM systems like the BC-250 by keeping more application data resident while swapping out less-used pages efficiently.

**Verify:**
```bash
# Check zswap is active
grep -r . /sys/module/zswap/parameters/

# Check swap is enabled
swapon --show

# Check swappiness
cat /proc/sys/vm/swappiness
```

!!!info "Source"
    Swap optimization approach based on [NexGen3D's SteamMachine script](https://github.com/NexGen-3D-Printing/SteamMachine).

---

### Memory Power Management

**GDDR6 Memory Characteristics:**

- Memory draws significant power (~35W+ at idle)
- No dynamic memory clocking in current drivers
- PS5 uses dynamic memory clocking (not available on BC-250)

**Potential Future Improvements:**
- Driver-level memory clock scaling
- BIOS update to enable dynamic VRAM clocking
- Could save 10-20W at idle if implemented

---

## Power Management Best Practices

### Recommended Configuration

**For Gaming (Balance Performance/Power):**

```toml
# /etc/cyan-skillfish-governor-smu/config.toml
min_frequency = 1000
max_frequency = 2000
min_voltage = 700
max_voltage = 950
```

Expected results:
- Idle: 70-85W
- Gaming: 150-180W
- Temperatures: 65-75°C

**For Power Efficiency (Low Consumption):**

```toml
# /etc/cyan-skillfish-governor-smu/config.toml
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

```toml
# /etc/cyan-skillfish-governor-smu/config.toml
safe-points = [
    [2000, 1000],
    [2230, 1050],
]
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
systemctl status cyan-skillfish-governor-smu
# or if using TT governor:
# systemctl status cyan-skillfish-governor-smu
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
| Governor not working | Service not running | `systemctl enable --now cyan-skillfish-governor-smu` |

---

## Additional Resources

**Governor Projects:**
- [Cyan Skillfish Governor SMU](https://github.com/filippor/cyan-skillfish-governor/tree/smu) (recommended)
- [Oberon Governor](https://gitlab.com/mothenjoyer69/oberon-governor) (legacy)

**Power Monitoring:**
- [CoolerControl](https://gitlab.com/coolercontrol/coolercontrol)
- [MangoHud](https://github.com/flightlessmango/MangoHud)

**Community Resources:**
- Discord Server: BC-250 Community
- GitHub Documentation: [BC-250 Docs](https://github.com/mothenjoyer69/bc250-documentation)

---

**Last Updated:** 2026-03-18
**Contributors:** Community testing and reporting
