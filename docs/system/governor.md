# GPU Governor Setup

The GPU governor is essential for BC-250 performance, enabling dynamic frequency and voltage scaling.

## Why You Need a Governor

### Without Governor

- GPU frequency **locked at 1500 MHz**
- No dynamic scaling
- Higher power consumption at idle (85-105W)
- Lower performance (can't reach 2000+ MHz)

### With Governor

- GPU scales from 1000 MHz (idle) to 2000-2230 MHz (load)
- Dynamic voltage scaling (700-1000 mV)
- Better temperatures
- Lower idle power (65-85W)
- Better gaming performance

**Power Savings:** 20-30W reduction at idle

!!!success "Essential for Performance"
    The governor is not optional for good gaming performance. Without it, you're stuck at 1500 MHz.

## Governor Options

### Oberon Governor (Original - Recommended)

**Developer:** mothenjoyer69 / TuxThePenguin0
**Type:** Two-state governor (min/max frequency)

**Features:**
- Simple configuration
- Proven stability
- Low CPU overhead (0.4%)
- Binary states: 1000 MHz idle, 2000 MHz load

**Repository:** [gitlab.com/mothenjoyer69/oberon-governor](https://gitlab.com/mothenjoyer69/oberon-governor)

### Filip's Enhanced Governor

**Developer:** FilippoR
**Type:** Multi-step governor

**Features:**
- Multiple frequency steps (not just 2)
- Maintains GPU usage 45-70%
- More responsive than original
- Available as COPR/RPM package
- Latest version: v0.1.4+

**Repository:** [github.com/filippor/oberon-governor](https://github.com/filippor/oberon-governor)

**Target:** Keeps GPU load in optimal range for consistent performance

### Cyan-Skillfish Governor

**Developer:** Magnap
**Type:** Continuous scaling governor

**Features:**
- Continuously adjusts frequency (no steps)
- Maintains GPU utilization 70-95%
- More precise voltage control
- Multiple voltage points supported
- Available as .deb, .rpm, AUR, Nix

**Repository:** [github.com/Magnap/cyan-skillfish-governor](https://github.com/Magnap/cyan-skillfish-governor)

**Performance:** Equivalent to max frequency at constant load, with better efficiency

## Installation

### Option 1: COPR (Fedora/Bazzite - Easiest)

COPR repositories are available for easy installation on Fedora and Bazzite. This eliminates the need to compile from source.

**For Oberon Governor (Recommended):**
```bash
# Add mothenjoyer69's COPR repository
sudo dnf copr enable @exotic-soc/oberon-governor

# Install governor
sudo dnf install oberon-governor

# Enable and start service
sudo systemctl enable --now oberon-governor.service

# Check status
systemctl status oberon-governor
```

**For Cyan-Skillfish Governor:**
```bash
# Add filippor's COPR repository
sudo dnf copr enable filippor/bazzite

# Install cyan-skillfish-governor
sudo dnf install cyan-skillfish-governor

# Enable and start service
sudo systemctl enable --now cyan-skillfish-governor.service
```

!!!success "No Compilation Required"
    Using COPR packages means you don't need to manually compile the governor from source. The packages are pre-built and maintained.

!!!warning "COPR Package Issues"
    Some users report core dumps with filippor's oberon-governor package. Use @exotic-soc/oberon-governor for Oberon, or filippor/bazzite for cyan-skillfish-governor instead.

### Option 2: Build from Source (All Distros)

**Install Dependencies:**

```bash
# Fedora
sudo dnf install -y libdrm-devel cmake make gcc-c++ git

# Arch
sudo pacman -S base-devel cmake git

# Debian/Ubuntu
sudo apt install build-essential cmake git libdrm-dev libyaml-cpp-dev
```

**Clone and Build:**

```bash
# Clone repository
git clone https://gitlab.com/mothenjoyer69/oberon-governor.git
cd oberon-governor

# Build
cmake . && make

# Install
sudo make install

# Enable service
sudo systemctl enable --now oberon-governor.service
```

### Option 3: Bazzite Automated Script

```bash
# Download and run setup script
curl -s https://raw.githubusercontent.com/vietsman/bc250-documentation/refs/heads/main/oberon-setup.sh | sudo sh

# Verify installation
systemctl status oberon-governor
```

### Option 4: Cyan-Skillfish Governor

**Fedora:**
```bash
sudo dnf copr enable filippor/bazzite
sudo dnf install cyan-skillfish-governor
```

**Arch:**
```bash
yay -S cyan-skillfish-governor
```

**Debian/Ubuntu:**
```bash
# Download .deb from GitHub releases
wget https://github.com/Magnap/cyan-skillfish-governor/releases/download/v0.1.3/cyan-skillfish-governor_0.1.3_amd64.deb
sudo dpkg -i cyan-skillfish-governor_0.1.3_amd64.deb
```

## Configuration

### Oberon Governor Config

**Config File:** `/etc/oberon-config.yaml`

**Default Configuration:**
```yaml
opps:
  frequency:
    min: 1000    # Minimum GPU frequency (MHz)
    max: 2000    # Maximum GPU frequency (MHz)
  voltage:
    min: 700     # Minimum voltage (mV)
    max: 1000    # Maximum voltage (mV)
```

**Safe Overclocking Config:**
```yaml
opps:
  frequency:
    min: 1000
    max: 2175    # Slight overclock
  voltage:
    min: 700
    max: 1025    # Slightly higher voltage for stability
```

**Restart after changes:**
```bash
sudo systemctl restart oberon-governor
```

### Filip's Multi-Step Config

**Config File:** `/etc/oberon-config.yaml`

**Advanced Multi-Step Configuration:**

!!!warning "Requires Kernel Patch"
    The 350 MHz minimum frequency requires the GPU frequency range kernel patch. Without the patch, the governor will crash with `std::__ios_failure`. Use `min: 1000` on stock kernels.

```yaml
opps:
  frequency:
    min: 350     # Requires kernel patch! Use 1000 on stock kernel
    max: 2175    # Overclocked maximum
  voltage:
    min: 700
    max: 1025
  steps: 24      # Number of frequency steps (creates 25 levels: 0-24)

governor:
  polling_delay_ms: 50        # How often to check GPU load
  up_threshold_high: 85       # Load % to jump to maximum
  up_threshold_low: 70        # Load % to step up one level
  down_threshold_high: 45     # Load % to step down one level
  down_threshold_low: 5       # Load % to drop to minimum
  gfx_temp_soft_lim: 80       # Temp to step down (°C)
  gfx_temp_hard_lim: 90       # Temp to drop to minimum (°C)
  soc_temp_hard_lim: 90       # SoC temp limit (°C)
  overheat_reset_ms: 10000    # Cool-down time after overheat
```

### Cyan-Skillfish Governor Config

**Config File:** `/etc/cyan-skillfish-governor/config.toml`

**Example Configuration:**
```toml
# Define voltage points (frequency MHz, voltage mV)
safe-points = [
    [1000, 700],   # 1000 MHz @ 700 mV (idle)
    [1500, 900],   # 1500 MHz @ 900 mV
    [2000, 1000],  # 2000 MHz @ 1000 mV (gaming)
    [2175, 1025],  # 2175 MHz @ 1025 mV (boost)
]

# GPU load target range (70-95%)
[load_target]
min = 0.70
max = 0.95

# Timing configuration
[timing]
interval_ms = 50       # Sampling interval
burst_samples = 20     # Samples before burst to max
```

**Test voltage/frequency manually:**
```bash
# Stop governor
sudo systemctl stop cyan-skillfish-governor

# Manually set frequency and voltage
echo vc 0 2000 1000 > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

# Test stability with benchmark/game

# If stable, add to config
```

## Verification

### Check Governor is Running

```bash
# Check service status
systemctl status oberon-governor

# Should show: active (running)
```

### Check Frequency Scaling

```bash
# View current GPU frequencies
cat /sys/class/drm/card0/device/pp_dpm_sclk

# Example output:
# 0: 1000Mhz
# 1: 1500Mhz
# 2: 2000Mhz *
#
# The * indicates active frequency
```

**Test dynamic scaling:**
1. Check frequency at idle (should be ~1000 MHz)
2. Start a game or benchmark
3. Check frequency under load (should increase to 2000+ MHz)

### Monitoring Tools

**CoolerControl (GUI):**
```bash
# Fedora
sudo dnf copr enable terra/terra
sudo dnf install coolercontrol

# Bazzite
ujust install-coolercontrol
```

Shows real-time GPU frequency, voltage, and temperature.

**Command Line:**
```bash
# Watch frequency changes
watch -n 1 'cat /sys/class/drm/card0/device/pp_dpm_sclk'
```

**MangoHud (In-Game Overlay):**
```bash
# Install MangoHud
sudo dnf install mangohud  # Fedora
sudo pacman -S mangohud    # Arch

# Run game with overlay
mangohud %command%  # Steam launch option
```

## Troubleshooting

### Governor Not Starting on Boot

**Symptoms:**
- GPU stuck at 1500 MHz
- Service shows as inactive

**Check service:**
```bash
sudo systemctl status oberon-governor

# Check logs
sudo journalctl -u oberon-governor
```

**Solutions:**

**1. Enable service:**
```bash
sudo systemctl enable oberon-governor
```

**2. Check config file exists:**
```bash
ls -l /etc/oberon-config.yaml

# If missing, reinstall governor
```

**3. Manual restart:**
```bash
sudo systemctl restart oberon-governor
```

**Workaround (Arch/CachyOS):**

Some users report governor doesn't activate until GPU is used:
- Run a game or benchmark once after boot
- Governor activates and stays active

### Frequency Stuck at 1500 MHz

**Possible causes:**
1. Governor not running
2. Config file missing/incorrect
3. Governor binary not installed

**Debug:**
```bash
# Check governor binary exists
which oberon-governor

# Check config
cat /etc/oberon-config.yaml

# Try manual start
sudo oberon-governor

# Check for errors
```

### System Crashes with Governor

**Symptoms:**
- System unstable during gaming
- Crashes when GPU frequency changes

**Causes:**
- Voltage too low for frequency
- Overheating
- Unstable overclock

**Solutions:**

**1. Increase voltage:**
```yaml
# Edit /etc/oberon-config.yaml
opps:
  voltage:
    max: 1050  # Increase from 1000 to 1050
```

**2. Reduce max frequency:**
```yaml
opps:
  frequency:
    max: 1900  # Reduce from 2000
```

**3. Check temperatures:**
```bash
sensors
# GPU should be < 85°C
```

### Governor High CPU Usage

**Normal CPU Usage:**
- Oberon original: 0.4% CPU
- Filip's enhanced: 0.4-1.0% CPU
- Cyan-Skillfish: 0.9-1.3% CPU

**If CPU usage > 2%:**

**Check polling interval:**
```yaml
# Reduce polling frequency
governor:
  polling_delay_ms: 100  # Increase from 50
```

**Check for bugs:**
```bash
# View governor logs
sudo journalctl -u oberon-governor -f
```

## Performance Comparison

| Governor | Idle Freq | Max Freq | CPU Usage | Response Time | Performance |
|----------|-----------|----------|-----------|---------------|-------------|
| **None** | 1500 MHz | 1500 MHz | 0% | N/A | ⭐⭐ |
| **Oberon** | 1000 MHz | 2000 MHz | 0.4% | 100ms | ⭐⭐⭐⭐ |
| **Filip's** | 350-1000 MHz | 2175 MHz | 0.4-1.0% | 50-100ms | ⭐⭐⭐⭐⭐ |
| **Cyan-Skillfish** | Variable | 2000+ MHz | 0.9-1.3% | 24ms | ⭐⭐⭐⭐⭐ |

## Governor Comparison

### When to Use Oberon (Original)

- **Simple setup:** Just works out-of-box
- **Proven stability:** Most tested
- **Low overhead:** Minimal CPU usage
- **Best for:** Beginners, stability-focused builds

### When to Use Filip's Enhanced

- **Better performance:** Multi-step scaling
- **Available as package:** Easy to install
- **Good balance:** Performance + stability
- **Best for:** Most users, gaming builds

### When to Use Cyan-Skillfish

- **Maximum control:** Precise frequency control
- **Best efficiency:** Continuous scaling
- **Advanced config:** Multiple voltage points
- **Best for:** Advanced users, overclockers

## Overclocking with Governor

### Safe Overclocking Guide

**Step 1: Test Maximum Stable Frequency**

```bash
# Stop governor
sudo systemctl stop oberon-governor

# Manually set test frequency
echo vc 0 2100 1050 > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

# Run benchmark (30+ minutes)
# If stable, try higher
# If crashes, lower frequency or increase voltage
```

**Step 2: Update Governor Config**

```yaml
# /etc/oberon-config.yaml
opps:
  frequency:
    max: 2100  # Your stable frequency
  voltage:
    max: 1050  # Your stable voltage
```

**Step 3: Restart and Test**

```bash
sudo systemctl restart oberon-governor

# Test with games/benchmarks
# Monitor temperatures
```

**Known Limits:**
- **1000 MHz:** Minimum safe frequency on stock kernel
- **2000 MHz @ 1000mV:** Safe for all boards
- **2175 MHz @ 1025mV:** Safe for most boards
- **2230 MHz:** Maximum with kernel patch, requires cooling

!!!warning "Overclocking Risks"
    Overclocking can cause instability, crashes, and potentially hardware damage. Always monitor temperatures and test thoroughly.

## See Also

- [System Configuration](../system/governor.md)
- [BIOS Overclocking Guide](../bios/overclocking.md)
- [Cooling Solutions](../hardware/cooling.md)
- [Performance Tuning](../gaming/compatibility.md)
