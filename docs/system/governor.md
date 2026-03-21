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

!!!success "ACPI Fix — Recommended"
    The [bc250-collective/bc250-acpi-fix](https://github.com/bc250-collective/bc250-acpi-fix) provides SSDT tables for CPU C-State and P-State support:

    - **SSDT-CST (C-States):** Enables C1/C2/C3 CPU idle states. Without this, CPU cores never enter sleep at idle.
    - **SSDT-PST (P-States):** Enables CPU frequency scaling from 800 MHz to 3200 MHz via standard Linux cpufreq governors (schedutil, powersave, etc.). Confirmed working on kernel 6.19.8.

    Both tables are loaded via initrd override — see the [ACPI fix installation section](#acpi-fix-installation) below. Not required for GPU governor operation, but significantly improves CPU idle power and enables CPU frequency scaling.

!!!danger "Minimum Voltage: 700mV"
    Never set minimum GPU voltage below 700mV. This locks the GPU to 1500MHz and defeats the purpose of the governor.

## Governor Options

### Cyan-Skillfish Governor TT (Recommended)

**Developer:** filippor (based on Magnap's work)
**Type:** Multi-step governor with thermal throttling
**Service name:** `cyan-skillfish-governor-tt`
**Config:** `/etc/cyan-skillfish-governor-tt/config.toml`

**Features:**
- Multiple frequency steps with thermal throttling
- Maintains GPU usage in optimal range
- Available as COPR/RPM, AUR, .deb, Nix
- Community default for Bazzite/Fedora (Jan 2026+)

**COPR:** `filippor/bazzite`
**Repository:** [github.com/Magnap/cyan-skillfish-governor](https://github.com/Magnap/cyan-skillfish-governor)

!!!warning "Service Name Changed (Dec 2025)"
    filippor renamed the service from `cyan-skillfish-governor` to `cyan-skillfish-governor-tt` on Dec 13, 2025. Config folder moved from `/etc/cyan-skillfish-governor/` to `/etc/cyan-skillfish-governor-tt/`. If upgrading, migrate your config:

    ```bash
    sudo cp /etc/cyan-skillfish-governor/config.toml /etc/cyan-skillfish-governor-tt/config.toml
    ```

### Cyan-Skillfish Governor SMU (Kernel-Patch Free)

**Developer:** filippor / Magnap
**Type:** SMU-based governor — bypasses kernel patches entirely
**Service name:** `cyan-skillfish-governor-smu`
**Released:** January 18, 2026

**Features:**
- Manages clock speeds through SMU firmware calls
- **Does NOT require kernel frequency range patch on any distro**
- Available on AUR (`cyan-skillfish-governor-smu`), COPR (`filippor/bazzite`), .deb, .rpm, Nix
- Best option for CachyOS/Arch users who don't want to patch kernels

**Repository:** [github.com/filippor/cyan-skillfish-governor (smu branch)](https://github.com/filippor/cyan-skillfish-governor/tree/smu)

### Oberon Governor (Deprecated)

**Developer:** mothenjoyer69 / TuxThePenguin0
**Type:** Two-state governor (min/max frequency)
**Status:** Deprecated — use cyan-skillfish-governor-smu instead

**Repository:** [gitlab.com/mothenjoyer69/oberon-governor](https://gitlab.com/mothenjoyer69/oberon-governor)

### NexGen3D Setup Script (Bazzite Beginners)

**Repository:** [github.com/NexGen-3D-Printing/SteamMachine](https://github.com/NexGen-3D-Printing/SteamMachine)

Installs cyan-skillfish-governor-tt + zram + CPU mitigations. De facto standard for Bazzite beginners.

### PS5GPU-BC250 (GUI Controller — New)

**Developer:** ZEROAESQUERDA
**Type:** GUI-based GPU controller with automatic and manual modes

**Features:**
- Visual Qt-based interface (works on KDE, GNOME)
- Adjust min/max GPU frequency and voltage
- Set operating temperature limits
- Automatic clock control in 4 boost stages based on load and temperature
- Manual frequency and voltage control mode
- No kernel patches or config file editing required
- Works like MSI Afterburner / Adrenalin on Windows

**Repository:** [github.com/ZEROAESQUERDA/PS5GPU-BC250](https://github.com/ZEROAESQUERDA/PS5GPU-BC250)

!!!warning "Disable Other Governors First"
    You must disable any running GPU governor (cyan-skillfish-governor-smu, cyan-skillfish-governor-tt) before using PS5GPU-BC250. Running multiple frequency controllers simultaneously will cause conflicts.

---

## Installation

### Option 1: COPR (Fedora/Bazzite - Easiest)

COPR repositories are available for easy installation on Fedora and Bazzite. This eliminates the need to compile from source.

**For Cyan-Skillfish Governor TT (Recommended):**
```bash
# Add filippor's COPR repository
sudo dnf copr enable filippor/bazzite

# Install cyan-skillfish-governor-tt (note the -tt suffix)
sudo dnf install cyan-skillfish-governor-tt

# Enable and start service
sudo systemctl enable --now cyan-skillfish-governor-tt.service
```

**For SMU Governor (No Kernel Patch Needed):**
```bash
# Fedora/Bazzite:
sudo dnf copr enable filippor/bazzite
sudo dnf install cyan-skillfish-governor-smu
sudo systemctl enable --now cyan-skillfish-governor-smu.service

# Arch/CachyOS:
yay -S cyan-skillfish-governor-smu
sudo systemctl enable --now cyan-skillfish-governor-smu.service
```

!!!warning "Important: Verify GPU Device Targeting"
    After installation, verify the governor is targeting the correct GPU device:

    - Check which card is BC-250: `ls -la /sys/class/drm/ | grep card`
    - The governor may target card0 or card1 depending on your system
    - If governor settings don't apply, you may need to manually specify the correct card in configuration

!!!success "No Compilation Required"
    Using COPR packages means you don't need to manually compile the governor from source. The packages are pre-built and maintained.

!!!info "COPR Package Status"
    The `filippor/bazzite` COPR provides `cyan-skillfish-governor-smu` and `cyan-skillfish-governor-tt`, confirmed working as of Dec 2025–Mar 2026. The SMU variant is recommended as it requires no kernel patch.

### Option 2: Build from Source (All Distros)

**Install Dependencies:**

```bash
# Fedora
sudo dnf install -y libdrm-devel cmake make gcc-c++ git

# Arch
sudo pacman -S base-devel cmake git

# Debian/Ubuntu
sudo apt install build-essential cmake git libdrm-dev
```

**Clone and Build:**

```bash
# Clone repository
git clone https://github.com/filippor/cyan-skillfish-governor.git
cd cyan-skillfish-governor
git checkout smu

# Build
cmake . && make

# Install
sudo make install

# Enable service
sudo systemctl enable --now cyan-skillfish-governor-smu.service
```

### Option 3: Bazzite (rpm-ostree)

```bash
# Install via rpm-ostree
rpm-ostree install cyan-skillfish-governor-smu
sudo systemctl enable --now cyan-skillfish-governor-smu.service

# Verify installation
systemctl status cyan-skillfish-governor-smu
```

### Option 4: Arch/CachyOS (AUR)

**Arch/CachyOS:**
```bash
# TT version (requires kernel patch or Bazzite):
yay -S cyan-skillfish-governor-tt
# SMU version (no kernel patch needed — recommended for Arch/CachyOS):
yay -S cyan-skillfish-governor-smu
```

**Debian/Ubuntu:**
```bash
# Download .deb from GitHub releases
# Check https://github.com/Magnap/cyan-skillfish-governor/releases for latest
wget https://github.com/Magnap/cyan-skillfish-governor/releases/latest/download/cyan-skillfish-governor-tt_amd64.deb
sudo dpkg -i cyan-skillfish-governor-tt_amd64.deb
```

## Configuration

### Cyan Skillfish Governor SMU Config

**Config File:** `/etc/cyan-skillfish-governor-smu/config.toml`

The SMU governor manages GPU frequency via SMU firmware calls and does not require any kernel patch. See the [GitHub repository](https://github.com/filippor/cyan-skillfish-governor/tree/smu) for configuration options.

**Restart after changes:**
```bash
sudo systemctl restart cyan-skillfish-governor-smu
```

### Cyan-Skillfish Governor TT Config

**Config File:** `/etc/cyan-skillfish-governor-tt/config.toml`

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

!!!danger "Minimum Voltage: 700mV"
    Setting minimum voltage below 700mV locks the GPU to 1500MHz. Always keep min voltage ≥700mV.

**Test voltage/frequency manually:**
```bash
# Stop governor
sudo systemctl stop cyan-skillfish-governor-tt

# Manually set frequency and voltage
echo vc 0 2000 1000 > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

# Test stability with benchmark/game

# If stable, add to config
```

## Verification

### Check Governor is Running

```bash
# Check service status
systemctl status cyan-skillfish-governor-smu

# Should show: active (running)
```

### Check Frequency Scaling

```bash
# View current GPU frequencies
cat /sys/class/drm/card1/device/pp_dpm_sclk

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
watch -n 1 'cat /sys/class/drm/card1/device/pp_dpm_sclk'
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
# Check governor status:
sudo systemctl status cyan-skillfish-governor-smu

# Check logs
sudo journalctl -u cyan-skillfish-governor-smu
```

**Solutions:**

**1. Enable service:**
```bash
sudo systemctl enable cyan-skillfish-governor-smu
```

**2. Check config file exists:**
```bash
ls -l /etc/cyan-skillfish-governor-smu/config.toml

# If missing, reinstall governor
```

**3. Manual restart:**
```bash
sudo systemctl restart cyan-skillfish-governor-smu
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
4. Wrong COPR package installed

**Debug:**
```bash
# Check governor binary exists
which cyan-skillfish-governor-smu

# Check config
cat /etc/cyan-skillfish-governor-smu/config.toml

# Check for errors in logs
sudo journalctl -u cyan-skillfish-governor-smu --no-pager -n 20
```

### Governor Crashes with iostream Error

**Symptoms:**
```
terminate called after throwing an instance of 'std::__ios_failure'
  what():  basic_ios::clear: iostream error
Aborted
```

**Cause:** Legacy oberon-governor package is incompatible with kernel 6.17+.

**Solution:** Migrate to cyan-skillfish-governor-smu:
```bash
# Remove old oberon-governor
sudo systemctl stop oberon-governor
sudo systemctl disable oberon-governor
sudo dnf remove oberon-governor

# Install cyan-skillfish-governor-smu
sudo dnf copr enable filippor/bazzite
sudo dnf install cyan-skillfish-governor-smu

# Enable and start
sudo systemctl enable --now cyan-skillfish-governor-smu.service

# Verify it's running
systemctl status cyan-skillfish-governor-smu
```

**Verify fix:**
```bash
# Should show 1000 MHz at idle (not stuck at 1500 MHz)
cat /sys/class/drm/card1/device/pp_dpm_sclk
```

!!!note "GPU Card Number"
    Your GPU may be `card0` or `card1` depending on system configuration. Check with `ls /sys/class/drm/` to find the correct card.

### Black Screen on GPU Reset (Governor Running)

**Symptoms:**
- GPU crashes during a game, screen goes black and never recovers
- System appears to be running (fans spinning, SSH may work) but no display output
- Hard reboot required

**Cause:** When the GPU crashes while the governor is actively managing frequencies, the GPU reset mechanism can't complete properly. The governor continues trying to write to sysfs during the reset, preventing recovery.

**Workaround:**
- Disable governor before playing crash-prone games: `sudo systemctl stop cyan-skillfish-governor-tt`
- Re-enable after: `sudo systemctl start cyan-skillfish-governor-tt`

**Long-term fix:** Use stable voltage/frequency settings that don't cause GPU crashes in the first place. If a specific game consistently crashes the GPU, increase voltage or reduce max frequency.

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
```bash
# Edit /etc/cyan-skillfish-governor-smu/config.toml
# Increase max voltage from 1000 to 1050 mV
```

**2. Reduce max frequency:**
```bash
# Edit /etc/cyan-skillfish-governor-smu/config.toml
# Reduce max frequency from 2000 to 1900 MHz
```

**3. Check temperatures:**
```bash
sensors
# GPU should be < 85°C
```

### Governor High CPU Usage

**Normal CPU Usage:**
- Cyan-Skillfish SMU: 0.9-1.3% CPU
- Cyan-Skillfish TT: 0.9-1.3% CPU

**If CPU usage > 2%:**

**Check polling interval:**
```toml
# Reduce polling frequency in /etc/cyan-skillfish-governor-smu/config.toml
# Increase polling interval
```

**Check for bugs:**
```bash
# View governor logs
sudo journalctl -u cyan-skillfish-governor-smu -f
```

## Performance Comparison

| Governor | Idle Freq | Max Freq | CPU Usage | Response Time | Kernel Patch | Performance |
|----------|-----------|----------|-----------|---------------|--------------|-------------|
| **None** | 1500 MHz | 1500 MHz | 0% | N/A | No | ⭐⭐ |
| **Cyan-Skillfish SMU** | Variable | 2000+ MHz | 0.9-1.3% | 24ms | No | ⭐⭐⭐⭐⭐ |
| **Cyan-Skillfish TT** | Variable | 2000+ MHz | 0.9-1.3% | 24ms | Yes | ⭐⭐⭐⭐⭐ |

## Governor Comparison

### When to Use Cyan-Skillfish SMU (Recommended)

- **No kernel patch needed:** Works on any distro out-of-box
- **SMU firmware control:** Bypasses kernel frequency/voltage limits
- **Easy to install:** Available on AUR, COPR, .deb, .rpm, Nix
- **Best for:** All users, especially on Arch/CachyOS/Debian where kernel patching is inconvenient

### When to Use Cyan-Skillfish TT

- **Maximum control:** Precise frequency control with multiple voltage points
- **Best efficiency:** Continuous scaling with thermal throttling awareness
- **Advanced config:** Multiple voltage/frequency safe-points
- **Best for:** Advanced users, overclockers (requires kernel frequency range patch)

## Overclocking with Governor

### Safe Overclocking Guide

**Step 1: Test Maximum Stable Frequency**

```bash
# Stop governor
sudo systemctl stop cyan-skillfish-governor-smu

# Manually set test frequency
echo vc 0 2100 1050 > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage

# Run benchmark (30+ minutes)
# If stable, try higher
# If crashes, lower frequency or increase voltage
```

**Step 2: Update Governor Config**

```bash
# Edit /etc/cyan-skillfish-governor-smu/config.toml
# Set your stable max frequency (e.g. 2100 MHz) and voltage (e.g. 1050 mV)
```

**Step 3: Restart and Test**

```bash
sudo systemctl restart cyan-skillfish-governor-smu

# Test with games/benchmarks
# Monitor temperatures
```

**Known GPU Limits:**
- **350 MHz:** Minimum with kernel frequency range patch
- **1000 MHz:** Minimum on stock kernel
- **2000 MHz @ 1000mV:** Safe starting point
- **2100-2175 MHz @ 1025mV:** Works on many boards, test thoroughly
- **2230 MHz:** Maximum confirmed stable with kernel patch, requires good cooling

## CPU Overclocking with bc250_smu_oc

The [bc250_smu_oc](https://github.com/bc250-collective/bc250_smu_oc) tool overclocks the CPU via SMU commands. It raises the boost clock ceiling while keeping dynamic frequency scaling intact.

### Installation

```bash
git clone https://github.com/bc250-collective/bc250_smu_oc.git
cd bc250_smu_oc
pip install --user .
```

### Testing Frequencies

```bash
# Test 3700 MHz (auto-tunes voltage)
sudo bc250-detect -f 3700 -v 1231

# Test with --keep to maintain OC after tool exits
sudo bc250-detect -f 3900 -v 1280 -k
```

### Verified CPU Overclock Results (Fedora 43, kernel 6.19.8)

| Frequency | Auto-Tuned Voltage | 7zip MIPS | Temp (full load) | vs Stock |
|-----------|-------------------|-----------|------------------|----------|
| 3500 (stock) | auto | 26,062 | 60°C | baseline |
| 3600 MHz | 1150 mV | 26,518 | 65°C | +1.7% |
| 3700 MHz | 1199 mV | 27,212 | 68°C | +4.4% |
| 3800 MHz | 1250 mV | 27,919 | 72°C | +7.1% |
| 3900 MHz | 1275 mV | 28,410 | 75°C | +9.0% |
| 4000 MHz | — | throttles | 77°C | ❌ |

!!!note "Cooling Matters"
    These results were obtained with a fan curve service ramping PWM based on temperature. With the stock fan at PWM 80 (~1400 RPM), 3900 MHz throttles. Good cooling extends the OC headroom.

### Making CPU OC Permanent

```bash
# Test and generate config
sudo bc250-detect -f 3900 -v 1280 -k -c /etc/bc250-overclock.conf

# Install as systemd service
sudo bc250-apply -a -i /etc/bc250-overclock.conf
sudo systemctl enable bc250-smu-oc
```

The OC service raises the boost ceiling at boot. Combined with ACPI P-States and the `schedutil` governor, the CPU scales dynamically: **800 MHz at idle → 3900 MHz under load**.

!!!warning "Overclocking Risks"
    Overclocking can cause instability, crashes, and potentially hardware damage. Always monitor temperatures and test thoroughly.

## ACPI Fix Installation

The [bc250-acpi-fix](https://github.com/bc250-collective/bc250-acpi-fix) provides SSDT tables that enable CPU C-States (idle sleep) and P-States (frequency scaling). Both are confirmed working on kernel 6.19.8.

### What It Enables

- **C-States (SSDT-CST):** CPU cores enter C1/C2/C3 sleep states at idle, reducing power consumption
- **P-States (SSDT-PST):** CPU frequency scales from 800 MHz to 3200 MHz using standard Linux cpufreq governors (schedutil, powersave, performance, etc.)

### Installation

**Step 1: Clone and build the initrd override**

```bash
git clone https://github.com/bc250-collective/bc250-acpi-fix.git
cd bc250-acpi-fix

# Create ACPI override cpio archive
mkdir -p kernel/firmware/acpi
cp SSDT-CST.aml SSDT-PST.aml kernel/firmware/acpi/
find kernel | cpio -o -H newc > /tmp/acpi_override.cpio

# Copy to /boot
sudo cp /tmp/acpi_override.cpio /boot/acpi_override.cpio
```

**Step 2: Add to boot loader**

On Fedora (BLS entries):

```bash
# Edit the BLS entry for your current kernel
sudo nano /boot/loader/entries/$(cat /etc/machine-id)-$(uname -r).conf

# Prepend /acpi_override.cpio to the initrd line:
# Before: initrd /initramfs-6.19.8-200.fc43.x86_64.img
# After:  initrd /acpi_override.cpio /initramfs-6.19.8-200.fc43.x86_64.img
```

On Arch/Debian (GRUB):

```bash
# Add to /etc/default/grub:
GRUB_EARLY_INITRD_LINUX_CUSTOM="acpi_override.cpio"

# Regenerate GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # Fedora
sudo grub-mkconfig -o /boot/grub/grub.cfg    # Arch
sudo update-grub                              # Debian
```

**Step 3: Reboot and verify**

```bash
sudo reboot

# After reboot, check C-states:
ls /sys/devices/system/cpu/cpu0/cpuidle/
# Should show state0, state1, state2, state3

# Check P-states:
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
# Should show: 3200000 2550000 2325000 1960000 1820000 1600000 1271000 800000

# Set recommended CPU governor:
echo schedutil | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

!!!warning "Kernel Update Note"
    When you update to a new kernel, the BLS entry for the new kernel won't include the ACPI override. You'll need to edit the new entry or automate this with a kernel-install hook.

## Community Resources

- [NexGen3D SteamMachine Scripts](https://github.com/NexGen-3D-Printing/SteamMachine) — automated Bazzite setup (governor + zram + CPU mitigations)
- [DeathStalker Grimoire](https://github.com/DeathStalker471/bc250theGrimoire) — community step-by-step guide
- [PS5GPU-BC250](https://github.com/ZEROAESQUERDA/PS5GPU-BC250) — GUI GPU controller
- [cyan-skillfish-governor-smu](https://github.com/filippor/cyan-skillfish-governor/tree/smu) — SMU governor (no kernel patch needed)

## See Also

- [BIOS Overclocking Guide](../bios/overclocking.md)
- [Cooling Solutions](../hardware/cooling.md)
- [Performance Tuning](../gaming/compatibility.md)

---

**Last Updated:** 2026-03-21
