# Sensors and Monitoring

Guide to monitoring temperatures, fan speeds, voltages, and performance metrics on the BC-250.

---

## Overview

The BC-250 includes multiple hardware monitoring components:

- **Nuvoton NCT6686D SuperIO chip** - Motherboard sensors (temperatures, voltages, fan speeds)
    - For **read-only monitoring**: use the in-kernel `nct6683` driver (with `force=true`)
    - For **read+write PWM fan control**: use the out-of-tree `nct6687` driver ([Fred78290/nct6687d](https://github.com/Fred78290/nct6687d)) with `force=true`
- **AMD GPU sensors** - GPU temperature, voltage, power consumption
- **k10temp** - CPU temperature monitoring
- **NVMe sensors** - M.2 drive temperature

Proper monitoring is essential to ensure your BC-250 stays within safe operating temperatures (70-85°C under load) and to diagnose cooling or power issues.

---

## SuperIO Driver Setup

### About the SuperIO Chip

The BC-250 uses a **Nuvoton NCT6686D** Super I/O chip for hardware monitoring. There are two Linux driver options:

- **`nct6683`** (in-kernel) — Read-only access to sensors (temperatures, voltages, fan speeds). Cannot control fan PWM.
- **`nct6687`** (out-of-tree, [Fred78290/nct6687d](https://github.com/Fred78290/nct6687d)) — Full read+write access including PWM fan control. **Required if you want software fan control.**

Both require `force=true` because the chip isn't auto-detected. Regardless of which module is loaded, sensors will report as `nct6686-isa-0a20`.

The chip provides:

- Multiple temperature sensors (CPU, System, VRM MOS, and more)
- Voltage rails monitoring (+12V, +5V, +3.3V, CPU Soc, CPU Vcore, etc.)
- Fan speed monitoring (up to 8 fan headers)
- PWM fan control (only with `nct6687` module)

### Loading the Sensor Module

By default, Linux may not automatically load the driver for this chip. You need to manually enable it.

#### Option A: Read-Only Sensors (nct6683)

Use this if you only need temperature/voltage/fan speed monitoring without PWM fan control.

**Step 1:** Test if the module loads correctly:

```bash
sudo modprobe nct6683 force=true
```

**Step 2:** Make it permanent:

```bash
echo 'options nct6683 force=true' | sudo tee /etc/modprobe.d/sensors.conf
echo 'nct6683' | sudo tee /etc/modules-load.d/99-sensors.conf
```

#### Option B: Full PWM Fan Control (nct6687 — Recommended)

Use this if you want software fan speed control (CoolerControl, manual PWM, etc.).

**Step 1:** Build and install the nct6687 module:

```bash
git clone https://github.com/Fred78290/nct6687d.git
cd nct6687d
make
sudo make install
```

**Step 2:** Configure modprobe to use nct6687 and blacklist nct6683:

```bash
# Blacklist nct6683 (conflicts with nct6687)
echo 'blacklist nct6683' | sudo tee /etc/modprobe.d/sensors.conf
echo 'options nct6687 force=true' | sudo tee -a /etc/modprobe.d/sensors.conf

# Load nct6687 on boot
echo 'nct6687' | sudo tee /etc/modules-load.d/99-sensors.conf
```

!!!warning "Choose One Module"
    Do not load both `nct6683` and `nct6687` simultaneously — they conflict. Blacklist whichever you're not using.

#### Regenerate Initramfs

**On Fedora/Bazzite:**
```bash
sudo dracut --force
```

**On Arch/Manjaro:**
```bash
sudo mkinitcpio -P
```

**On Debian/Ubuntu:**
```bash
sudo update-initramfs -u
```

Reboot for changes to take effect:

```bash
sudo reboot
```

!!!info "PWM Values Reset on Reboot"
    The `nct6687` module does not persist PWM values across reboots. You'll need CoolerControl, a systemd service, or a udev rule to set your desired fan speed at boot.

---

## Using lm-sensors

### Installation

**Fedora/Bazzite:**
```bash
sudo dnf install lm_sensors
```

**Arch/Manjaro:**
```bash
sudo pacman -S lm_sensors
```

**Debian/Ubuntu:**
```bash
sudo apt install lm-sensors
```

### Detecting Sensors

Run the detection utility (answer YES to all prompts):

```bash
sudo sensors-detect
```

This will scan for all available sensors and configure them automatically.

### Reading Sensor Data

View all sensor readings:

```bash
sensors
```

### Expected Output

The output varies depending on which Super I/O module you loaded.

**With `nct6687` module (recommended — enables PWM fan control):**

```
amdgpu-pci-0100
Adapter: PCI adapter
vddgfx:      699.00 mV
vddnb:         1.10 V
edge:         +46.0°C
PPT:          38.01 W  (avg =  40.25 W)

nvme-pci-0300
Adapter: PCI adapter
Composite:    +38.9°C  (low  =  -0.1°C, high = +82.8°C)
                       (crit = +84.8°C)
Sensor 1:     +38.9°C  (low  = -273.1°C, high = +65261.8°C)

k10temp-pci-00c3
Adapter: PCI adapter
Tctl:         +47.9°C

nct6686-isa-0a20
Adapter: ISA adapter
+12V:            0.00 V  (min =  +0.00 V, max =  +0.00 V)
+5V:             0.00 V  (min =  +0.00 V, max =  +0.00 V)
+3.3V:           3.36 V  (min =  +0.00 V, max =  +3.36 V)
CPU Soc:         0.00 V  (min =  +0.00 V, max =  +0.00 V)
CPU Vcore:       0.00 V  (min =  +0.00 V, max =  +0.00 V)
CPU Fan:          0 RPM  (min =    0 RPM, max =    0 RPM)
Pump Fan:      1907 RPM  (min = 1866 RPM, max = 1907 RPM)
System Fan #1:    0 RPM  (min =    0 RPM, max =    0 RPM)
...
CPU:            +47.0°C  (low  = +35.0°C, high = +47.0°C)
System:         +42.5°C  (low  = +20.0°C, high = +42.5°C)
VRM MOS:        +42.0°C  (low  = +20.0°C, high = +42.0°C)
```

**With `nct6683` module (read-only, no fan control):**

```
nct6686-isa-0a20
Adapter: ISA adapter
VIN0:             832.00 mV (min =  +0.00 V, max =  +0.00 V)
...
fan1:                0 RPM  (min =    0 RPM)
fan2:             1372 RPM  (min =    0 RPM)
...
AMD TSI Addr 98h:  +63.0°C  ...  sensor = AMD AMDSI
Thermistor 14:     +57.5°C  ...  sensor = thermistor
Thermistor 15:     +57.0°C  ...  sensor = thermistor
```

!!!info "Sensor Names Differ By Module"
    Both modules report as `nct6686-isa-0a20`, but the `nct6687` module provides named labels (CPU Fan, Pump Fan, CPU Soc, etc.) while `nct6683` shows generic names (VIN0, fan1, Thermistor 14, etc.).

### Understanding the Sensors

**GPU Sensors (amdgpu-pci-0100):**
- `vddgfx` - GPU core voltage
- `vddnb` - Northbridge/memory voltage
- `edge` - GPU edge temperature (primary GPU temp)
- `PPT` - Package Power Tracking (GPU power consumption in watts)

**CPU Sensors (k10temp-pci-00c3):**
- `Tctl` - CPU temperature (Zen 2 control temperature)

**SuperIO Sensors (nct6686-isa-0a20):**

With `nct6687` module:

- `+12V`, `+5V`, `+3.3V`, `CPU Soc`, `CPU Vcore` etc. - Named voltage rails
- `CPU Fan`, `Pump Fan`, `System Fan #1-6` - Named fan speed monitoring (RPM), up to 8 channels
- `CPU`, `System`, `VRM MOS` - Named temperature sensors

With `nct6683` module:

- `VIN0-VIN16` - Voltage rails (generic names)
- `fan1-fan5` - Fan speed monitoring (RPM)
- `AMD TSI Addr 98h` - AMD Temperature Sensor Interface (CPU temp)
- `Thermistor 14/15` - Board temperature sensors

### Watch Sensors in Real-Time

Monitor sensor changes continuously:

```bash
watch -n 1 sensors
```

This updates every second. Press `Ctrl+C` to exit.

---

## GPU Temperature Monitoring

### Using AMDGPU Sysfs

Read GPU temperature directly:

```bash
cat /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input
```

This returns temperature in millidegrees Celsius (e.g., `63000` = 63°C)

Convert to Celsius:

```bash
awk '{print $1/1000 "°C"}' /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input
```

### GPU Power Consumption

Read current GPU power draw:

```bash
cat /sys/class/drm/card1/device/hwmon/hwmon*/power1_average
```

Returns power in microwatts. Convert to watts:

```bash
awk '{print $1/1000000 "W"}' /sys/class/drm/card1/device/hwmon/hwmon*/power1_average
```

### GPU Clock Speeds

Check current GPU frequency:

```bash
cat /sys/class/drm/card1/device/pp_dpm_sclk
```

Example output:
```
0: 350Mhz
1: 1000Mhz *
2: 2000Mhz
```

The asterisk (*) indicates the current active clock speed.

---

## Fan Speed Monitoring and Control

### Viewing Fan Speeds

From `sensors` output, look for the `nct6686-isa-0a20` section:

```bash
sensors | grep -A 5 "fan"
```

### Fan Headers on BC-250

The board has two physical fan headers: **J1** (primary) and **J4003** (secondary).

The Super I/O chip exposes up to 8 fan channels in software (fan1-fan8), but only the connected headers will show RPM readings:

- The main cooling fan typically reads as **Pump Fan** (fan2 in `nct6687` output, or fan2 in `nct6683` output)
- Other channels show 0 RPM if not connected

### BIOS Fan Control Settings

The BC-250 BIOS has three fan control modes:

1. **Default** - Tries to keep board at maximum safe temperature with minimum fan speed (NOT RECOMMENDED - runs too hot)
2. **Full Speed** - Fans run at 100% constantly (recommended for testing and maximum cooling)
3. **Customize** - Set custom temperature/fan speed curves in BIOS

**Recommendation:** Use "Full Speed" mode for initial testing and gaming. Once stable, you can use software fan control (CoolerControl) for quieter operation.

### Manual Fan Control via PWM

!!!warning "Requires nct6687 Module"
    PWM fan control requires the `nct6687` module. The in-kernel `nct6683` module provides read-only access and cannot set PWM values.

Check available PWM controls:

```bash
ls /sys/class/hwmon/hwmon*/pwm*
```

Set fan speed manually (value 0-255, where 255 = 100%):

```bash
# Find the hwmon for nct6686
HWMON=$(grep -l nct6686 /sys/class/hwmon/hwmon*/name | head -1 | xargs dirname)

# Set PWM (0=off, 127=50%, 255=100%)
echo 80 | sudo tee $HWMON/pwm2
```

!!!info "PWM Resets on Reboot"
    PWM values set manually are not persistent across reboots. Use CoolerControl or a systemd service to set fan speed at boot.

---

## CoolerControl - GUI for Sensor Monitoring and Fan Curves

CoolerControl is a GUI application that provides:

- Real-time sensor monitoring
- Custom fan curves based on any temperature sensor
- Historical temperature graphs
- Fan speed control

### Installation on Bazzite

Bazzite has a built-in recipe for CoolerControl:

```bash
ujust install-coolercontrol
```

This will:
1. Enable the Terra repository
2. Install `liquidctl` and `coolercontrol`
3. Require a reboot to apply

After reboot, start CoolerControl:

```bash
coolercontrol
```

### Installation on Fedora

Enable the Terra repository:

```bash
sudo dnf copr enable copr.fedorainfracloud.org/terra
```

Install CoolerControl:

```bash
sudo dnf install liquidctl coolercontrol
```

### Installation on Arch/Manjaro

Install from AUR:

```bash
yay -S coolercontrol
```

Or:

```bash
paru -S coolercontrol
```

### Using CoolerControl

1. Launch CoolerControl from your application menu or terminal
2. You'll see all detected sensors and fans
3. Click on a fan to create a custom curve
4. Select a temperature sensor to base the curve on (e.g., GPU edge temp)
5. Drag curve points to set fan speed at different temperatures

**Example fan curve:**
- 40°C: 30% fan speed
- 60°C: 50% fan speed
- 70°C: 75% fan speed
- 80°C: 100% fan speed

### Checking Frequency and Temperature with Governor

From Discord community recommendations:

**Desktop mode:**
```bash
# Use CoolerControl (installed with ujust on Bazzite)
coolercontrol
```

**Game mode (Steam Deck UI):**
- Enable Steam overlay performance monitoring in Quick Settings

---

## Other Monitoring Tools

### nvtop - GPU Activity Monitor

`nvtop` is like `htop` but for GPUs. It shows GPU utilization, VRAM usage, temperature, and power consumption in a terminal UI.

**Installation:**

Fedora/Bazzite:
```bash
sudo dnf install nvtop
```

Arch/Manjaro:
```bash
sudo pacman -S nvtop
```

Debian/Ubuntu:
```bash
sudo apt install nvtop
```

**Usage:**
```bash
nvtop
```

Press `q` to quit.

### radeontop - AMD GPU Monitor

Alternative GPU monitoring tool specifically for AMD GPUs.

**Installation:**

Fedora/Bazzite:
```bash
sudo dnf install radeontop
```

Arch/Manjaro:
```bash
sudo pacman -S radeontop
```

Debian/Ubuntu:
```bash
sudo apt install radeontop
```

**Usage:**
```bash
radeontop
```

Press `q` to quit.

### MangoHud - In-Game Overlay

MangoHud provides an in-game overlay showing FPS, GPU/CPU temps, usage, and more.

**Installation:**

Fedora/Bazzite:
```bash
sudo dnf install mangohud
```

Arch/Manjaro:
```bash
sudo pacman -S mangohud
```

Debian/Ubuntu:
```bash
sudo apt install mangohud
```

**Usage:**

Launch games with MangoHud:
```bash
mangohud %command%
```

For Steam games, add to launch options:
```
mangohud %command%
```

**Configuration:**

Create `~/.config/MangoHud/MangoHud.conf`:

```ini
fps_limit=60
vsync=0
gpu_temp
cpu_temp
gpu_power
cpu_power
ram
vram
fps
frametime=0
frame_timing=1
position=top-left
font_size=24
```

---

## Temperature Thresholds and Safe Operating Ranges

### Normal Operating Temperatures

**Idle (Desktop/Light Use):**
- GPU: 45-55°C
- CPU: 45-55°C
- Power: 50-70W

**Gaming/Heavy Load:**
- GPU: 70-85°C
- CPU: 65-80°C
- Power: 150-235W (235W max during Cyberpunk with ray tracing)

### Maximum Safe Temperatures

- **GPU edge temp**: 85°C maximum recommended (can briefly spike to 90°C)
- **CPU (Tctl)**: 90°C maximum
- **NVMe SSD**: 80°C maximum (critical at 81.8°C per spec)

### Temperature Warning Signs

If you see these symptoms, your cooling is insufficient:

- GPU consistently above 85°C during gaming
- Sudden FPS drops or stuttering (thermal throttling)
- System crashes under load
- GPU governor reducing frequency to manage heat

### Improving Cooling

If temperatures are too high:

1. **Check fan speeds** - Ensure fans are running at adequate RPM
2. **Verify thermal paste** - Reapply if paste is old or dried
3. **Increase fan speed** - Use BIOS "Full Speed" mode or CoolerControl
4. **Improve airflow** - Add more fans or cut heatsink fins for better air penetration
5. **Check dust** - Clean dust from heatsink fins
6. **Verify thermal pads** - Ensure good contact on GDDR6 memory chips

### Community Cooling Data

From Discord testing:

- **Arctic P12 Max with cut fins**: 70-85°C gaming, 86°C benchmarks
- **Noctua NF-A12x25 with cut fins**: 65-80°C gaming
- **Stock heatsink with fan shroud**: 80-90°C gaming (thermal limits reached)
- **Wraith Stealth coolers with thermal putty**: ~70°C mining/LLM workloads at 180W

---

## Troubleshooting Sensor Issues

### Sensors Command Shows No Output

**Problem:** Running `sensors` shows nothing or very limited data.

**Solutions:**

1. Run sensor detection:
   ```bash
   sudo sensors-detect
   ```

2. Ensure NCT6683 module is loaded:
   ```bash
   sudo modprobe nct6683 force=true
   ```

3. Check if modules are loaded:
   ```bash
   lsmod | grep -E "nct6683|k10temp|amdgpu"
   ```

4. Verify lm-sensors is installed:
   ```bash
   sensors --version
   ```

### Sensor Module Won't Load

**Problem:** `modprobe nct6683 force=true` or `modprobe nct6687 force=true` fails.

**Solutions:**

1. Check kernel version (needs 6.11+):
   ```bash
   uname -r
   ```

2. Check dmesg for errors:
   ```bash
   dmesg | grep -i nct
   ```

3. Ensure the modules are not conflicting — only load one at a time:
   ```bash
   # Check what's loaded
   lsmod | grep nct
   
   # If nct6683 is loaded but you want nct6687:
   sudo rmmod nct6683
   sudo modprobe nct6687 force=true
   ```

4. For nct6687, you may need to build from source if it's not packaged for your distro:
   ```bash
   git clone https://github.com/Fred78290/nct6687d.git
   cd nct6687d && make && sudo make install
   sudo modprobe nct6687 force=true
   ```

!!!info "nct6683 vs nct6687"
    `nct6683` is read-only (temperature, voltage, fan speed monitoring). `nct6687` provides full read+write access including PWM fan control. For fan curves and manual speed control, you need `nct6687`.

### GPU Temperature Not Showing

**Problem:** No GPU temperature in `sensors` output.

**Solutions:**

1. Check if amdgpu driver is loaded:
   ```bash
   lsmod | grep amdgpu
   ```

2. Verify GPU is detected:
   ```bash
   lspci | grep VGA
   ```

3. Check amdgpu sysfs directly:
   ```bash
   cat /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input
   ```

4. Ensure Mesa 25.1+ is installed:
   ```bash
   glxinfo | grep "OpenGL version"
   ```

### Fan Speeds Show 0 RPM

**Problem:** All fans show 0 RPM even though fans are spinning.

**Possible causes:**

1. **Fan not connected to monitored header** - BC-250 usually only uses fan2 header
2. **3-pin fan on PWM header** - Some fans don't report speed
3. **Fan splitter** - May not pass tachometer signal
4. **BIOS fan setting** - Try changing fan mode in BIOS

**Verification:**

If you can hear/feel the fan spinning, it's working even if sensors show 0 RPM. Use GPU temperature as confirmation of cooling effectiveness.

### Power Consumption Seems Wrong

**Problem:** `sensors` shows very low or zero power consumption.

**Solution:**

1. Power readings update slowly - wait 10-30 seconds
2. Use a Kill-A-Watt or smart plug for accurate wall power measurement
3. GPU power from `sensors` only shows GPU chip power, not total system power
4. Total system power = GPU + CPU + RAM + board + PSU inefficiency

**Example:**
- GPU PPT: 150W
- Total system power at wall: 180-200W

---

## Advanced: Monitoring Scripts

### Temperature Logging Script

Save as `~/monitor-temps.sh`:

```bash
#!/bin/bash

# BC-250 Temperature Monitor
# Logs GPU temp, CPU temp, and power to file

LOGFILE="$HOME/bc250-temps.log"

echo "Timestamp,GPU_Temp,CPU_Temp,GPU_Power" > "$LOGFILE"

while true; do
    TIMESTAMP=$(date +%s)
    GPU_TEMP=$(cat /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input 2>/dev/null | awk '{print $1/1000}')
    CPU_TEMP=$(sensors k10temp-pci-00c3 -u 2>/dev/null | grep temp1_input | awk '{print $2}')
    GPU_POWER=$(cat /sys/class/drm/card1/device/hwmon/hwmon*/power1_average 2>/dev/null | awk '{print $1/1000000}')

    echo "$TIMESTAMP,$GPU_TEMP,$CPU_TEMP,$GPU_POWER" >> "$LOGFILE"
    sleep 5
done
```

Make executable:
```bash
chmod +x ~/monitor-temps.sh
```

Run:
```bash
~/monitor-temps.sh
```

### Temperature Alert Script

Save as `~/temp-alert.sh`:

```bash
#!/bin/bash

# Alert if GPU temp exceeds threshold
THRESHOLD=85

while true; do
    GPU_TEMP=$(cat /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input 2>/dev/null | awk '{print $1/1000}')

    if (( $(echo "$GPU_TEMP > $THRESHOLD" | bc -l) )); then
        notify-send -u critical "BC-250 Temperature Alert" "GPU temp: ${GPU_TEMP}°C (threshold: ${THRESHOLD}°C)"
    fi

    sleep 10
done
```

---

## Quick Reference

### Essential Commands

```bash
# View all sensors
sensors

# Watch sensors in real-time
watch -n 1 sensors

# GPU temperature only
cat /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input | awk '{print $1/1000 "°C"}'

# GPU power consumption
cat /sys/class/drm/card1/device/hwmon/hwmon*/power1_average | awk '{print $1/1000000 "W"}'

# GPU clock speed
cat /sys/class/drm/card1/device/pp_dpm_sclk

# Launch nvtop
nvtop

# Launch radeontop
radeontop
```

### Temperature Targets

| Condition | GPU Temp | CPU Temp | Power Draw |
|-----------|----------|----------|------------|
| Idle | 45-55°C | 45-55°C | 50-70W |
| Light Gaming | 60-75°C | 55-70°C | 100-150W |
| Heavy Gaming | 70-85°C | 65-80°C | 150-200W |
| Stress Test | 80-86°C | 75-85°C | 200-235W |

### Cooling Solutions Performance

| Setup | Idle Temp | Load Temp | Notes |
|-------|-----------|-----------|-------|
| Arctic P12 Max (cut fins) | 49°C | 70-86°C | Best performance |
| Noctua NF-A12x25 (cut fins) | 47-52°C | 65-80°C | Quieter, excellent cooling |
| Single 120mm (cut fins) | 55°C | 80-90°C | Adequate for most games |
| Stock with shroud | 60°C | 85-90°C | Borderline, may throttle |

---

## Related Documentation

- [GPU Governor](governor.md) - Configure GPU frequency scaling and power management
- [Performance Tuning](power.md) - Optimize system performance
- [Hardware Overview](../hardware/specifications.md) - BC-250 specifications and cooling requirements
- [Troubleshooting](../troubleshooting/performance.md) - Solve thermal and stability issues

---

**Last Updated:** March 18, 2026
