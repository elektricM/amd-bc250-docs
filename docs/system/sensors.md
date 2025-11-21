# Sensors and Monitoring

A comprehensive guide to monitoring temperatures, fan speeds, voltages, and performance metrics on the BC-250.

---

## Overview

The BC-250 includes multiple hardware monitoring components:

- **NCT6686/NCT6687 SuperIO chip** - Motherboard sensors (temperatures, voltages, fan speeds)
- **AMD GPU sensors** - GPU temperature, voltage, power consumption
- **k10temp** - CPU temperature monitoring
- **NVMe sensors** - M.2 drive temperature

Proper monitoring is essential to ensure your BC-250 stays within safe operating temperatures (70-85°C under load) and to diagnose cooling or power issues.

---

## NCT6686/NCT6687 SuperIO Setup

### What is the NCT6686/NCT6687?

The Nuvoton NCT6686 or NCT6687 is a Super I/O chip on the BC-250 that provides hardware monitoring capabilities including:

- Multiple temperature sensors (thermistors, AMD TSI)
- Voltage rails monitoring
- Fan speed monitoring (up to 5 fan headers)
- Fan control capabilities

### Loading the Sensor Module

By default, Linux may not automatically load the driver for this chip. You need to manually enable it.

#### Step 1: Load the Module Temporarily

Test if the module loads correctly:

```bash
sudo modprobe nct6683 force=true
```

Verify it loaded:

```bash
lsmod | grep nct6683
```

#### Step 2: Make it Permanent

Create a modprobe configuration file:

```bash
sudo nano /etc/modprobe.d/sensors.conf
```

Add the following line:

```
options nct6683 force=true
```

Create a modules load file:

```bash
sudo nano /etc/modules-load.d/99-sensors.conf
```

Add:

```
nct6683
```

#### Step 3: Regenerate Initramfs

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

Here's what you should see on a properly configured BC-250:

```bash
amdgpu-pci-0100
Adapter: PCI adapter
vddgfx:      906.00 mV
vddnb:       824.00 mV
edge:         +63.0°C
PPT:          55.12 W  (avg =   0.00 W)

nvme-pci-0300
Adapter: PCI adapter
Composite:    +51.9°C  (low  =  -0.1°C, high = +79.8°C)
                       (crit = +81.8°C)
Sensor 1:     +51.9°C  (low  = -273.1°C, high = +65261.8°C)

k10temp-pci-00c3
Adapter: PCI adapter
Tctl:         +51.5°C

nct6686-isa-0a20
Adapter: ISA adapter
VIN0:             832.00 mV (min =  +0.00 V, max =  +0.00 V)
VIN1:               1.02 V  (min =  +0.00 V, max =  +0.00 V)
VIN2:             976.00 mV (min =  +0.00 V, max =  +0.00 V)
VIN6:               1.39 V  (min =  +0.00 V, max =  +0.00 V)
VIN7:             928.00 mV (min =  +0.00 V, max =  +0.00 V)
VIN16:            896.00 mV (min =  +0.00 V, max =  +0.00 V)
fan1:                0 RPM  (min =    0 RPM)
fan2:             1372 RPM  (min =    0 RPM)
fan3:                0 RPM  (min =    0 RPM)
fan4:                0 RPM  (min =    0 RPM)
fan5:                0 RPM  (min =    0 RPM)
AMD TSI Addr 98h:  +63.0°C  (low  =  +0.0°C)
                            (high =  +0.0°C, hyst =  +0.0°C)
                            (crit =  +0.0°C)  sensor = AMD AMDSI
Thermistor 14:     +57.5°C  (low  =  +0.0°C)
                            (high =  +0.0°C, hyst =  +0.0°C)
                            (crit =  +0.0°C)  sensor = thermistor
Thermistor 15:     +57.0°C  (low  =  +0.0°C)
                            (high =  +0.0°C, hyst =  +0.0°C)
                            (crit =  +0.0°C)  sensor = thermistor
intrusion0:       OK
beep_enable:      disabled
```

### Understanding the Sensors

**GPU Sensors (amdgpu-pci-0100):**
- `vddgfx` - GPU core voltage
- `vddnb` - Northbridge/memory voltage
- `edge` - GPU edge temperature (primary GPU temp)
- `PPT` - Package Power Tracking (GPU power consumption in watts)

**CPU Sensors (k10temp-pci-00c3):**
- `Tctl` - CPU temperature (Zen 2 control temperature)

**SuperIO Sensors (nct6686-isa-0a20):**
- `VIN0-VIN16` - Various voltage rails
- `fan1-fan5` - Fan speed monitoring (RPM)
- `AMD TSI Addr 98h` - AMD Temperature Sensor Interface (alternative CPU temp reading)
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
cat /sys/class/drm/card0/device/hwmon/hwmon*/temp1_input
```

This returns temperature in millidegrees Celsius (e.g., `63000` = 63°C)

Convert to Celsius:

```bash
awk '{print $1/1000 "°C"}' /sys/class/drm/card0/device/hwmon/hwmon*/temp1_input
```

### GPU Power Consumption

Read current GPU power draw:

```bash
cat /sys/class/drm/card0/device/hwmon/hwmon*/power1_average
```

Returns power in microwatts. Convert to watts:

```bash
awk '{print $1/1000000 "W"}' /sys/class/drm/card0/device/hwmon/hwmon*/power1_average
```

### GPU Clock Speeds

Check current GPU frequency:

```bash
cat /sys/class/drm/card0/device/pp_dpm_sclk
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

The board has **5 fan headers** (shown as fan1-fan5 in sensors):

- Usually only **fan2** is used for the main cooling fan
- Other headers may show 0 RPM if not connected

### BIOS Fan Control Settings

The BC-250 BIOS has three fan control modes:

1. **Default** - Tries to keep board at maximum safe temperature with minimum fan speed (NOT RECOMMENDED - runs too hot)
2. **Full Speed** - Fans run at 100% constantly (recommended for testing and maximum cooling)
3. **Customize** - Set custom temperature/fan speed curves in BIOS

**Recommendation:** Use "Full Speed" mode for initial testing and gaming. Once stable, you can use software fan control (CoolerControl) for quieter operation.

### Manual Fan Control via PWM

Check available PWM controls:

```bash
ls /sys/class/hwmon/hwmon*/pwm*
```

Set fan speed manually (value 0-255, where 255 = 100%):

```bash
echo 200 | sudo tee /sys/class/hwmon/hwmon*/pwm2
```

Enable manual PWM control:

```bash
echo 1 | sudo tee /sys/class/hwmon/hwmon*/pwm2_enable
```

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

### NCT6683 Module Won't Load

**Problem:** `modprobe nct6683 force=true` fails or doesn't work.

**Solutions:**

1. Check kernel version (needs 6.11+):
   ```bash
   uname -r
   ```

2. Verify chip detection:
   ```bash
   sudo sensors-detect
   ```

3. Check dmesg for errors:
   ```bash
   dmesg | grep nct6683
   ```

4. Some kernels may need `nct6687` instead:
   ```bash
   sudo modprobe nct6687 force=true
   ```

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
   cat /sys/class/drm/card0/device/hwmon/hwmon*/temp1_input
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
    GPU_TEMP=$(cat /sys/class/drm/card0/device/hwmon/hwmon*/temp1_input 2>/dev/null | awk '{print $1/1000}')
    CPU_TEMP=$(sensors k10temp-pci-00c3 -u 2>/dev/null | grep temp1_input | awk '{print $2}')
    GPU_POWER=$(cat /sys/class/drm/card0/device/hwmon/hwmon*/power1_average 2>/dev/null | awk '{print $1/1000000}')

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
    GPU_TEMP=$(cat /sys/class/drm/card0/device/hwmon/hwmon*/temp1_input 2>/dev/null | awk '{print $1/1000}')

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
cat /sys/class/drm/card0/device/hwmon/hwmon*/temp1_input | awk '{print $1/1000 "°C"}'

# GPU power consumption
cat /sys/class/drm/card0/device/hwmon/hwmon*/power1_average | awk '{print $1/1000000 "W"}'

# GPU clock speed
cat /sys/class/drm/card0/device/pp_dpm_sclk

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

**Last Updated:** November 21, 2025
