# Fan Control

Software fan control for the BC-250: set the CPU_FAN1 fan speed by hand or run a temperature-based fan curve from Linux. This is useful when the BIOS fan modes are unsuitable — the default mode runs the board too hot, "Full Speed" runs the fan constantly at 100%.

PWM **write** access requires the out-of-tree `nct6687` driver. If it is not set up yet, follow [Sensors & Monitoring — Full PWM Fan Control](sensors.md#option-b-full-pwm-fan-control-nct6687-recommended) first. The in-kernel `nct6683` driver is read-only. Ready-made scripts for the setups described on this page are available in the [Naster17/bc250fc](https://github.com/Naster17/bc250fc) repository.

---

## How the fan header maps to Linux

The NCT6686D Super I/O chip exposes up to 8 fan channels, but only one header is populated on the board:

| Header | Linux (nct6687) | hwmon node | Notes |
|---|---|---|---|
| CPU_FAN1 (J1) | **Pump Fan** (fan2) | `pwm2` / `fan2_input` | The only usable header; 4-pin PWM |
| J4003 | — | — | Rack fan distribution header, see [Pinouts](../hardware/pinouts.md) |

The hwmon device shows up as `nct6686` (driver module `nct6687`):

```bash
HWMON=$(grep -l nct6686 /sys/class/hwmon/hwmon*/name | head -1 | xargs dirname)
echo "$HWMON"          # e.g. /sys/class/hwmon/hwmon3
cat "$HWMON/fan2_input"   # RPM
cat "$HWMON/pwm2"         # duty cycle 0-255
```

`pwm2_enable` selects the mode: `2` = BIOS-controlled, `1` = manual software control. Writing to `pwm2` only has an effect in manual mode.

!!!info "BIOS numbering is different"
    The BIOS calls the same header "CPU Fan" and numbers it differently from Linux. See the fan-numbering table in [Pinouts](../hardware/pinouts.md#fan-headers).

## Fan channels: CPU_FAN1 and the J4003 header

The NCT6686D exposes 8 PWM channels in hwmon, and all five wired fan outputs of the board are controllable under Linux. Besides the main CPU_FAN1 header there is a second connector next to it, silkscreened **`J4003`** — a 2×8-pin, 2.54 mm multi-fan header with individually labeled pins (`F1T`…`F5T` tachometers, `F1P`…`F5P` PWM outputs, `DET`, `GND`). It was designed for five 80 mm rack fans, and all five PWM pins are usable for chassis fans.

| Channel | Connected to | BIOS fan |
|---|---|---|
| `pwm2` | **CPU_FAN1** (4-pin header) | fan 1 |
| `pwm1` | J4003 fan pin | fan 5 |
| `pwm3` | J4003 fan pin | fan 2 |
| `pwm4` | J4003 fan pin | fan 3 |
| `pwm5` | J4003 fan pin | fan 4 |
| `pwm6`–`pwm8` | not wired on this board | — |

Channels `pwm1`–`pwm5` accept and hold manual duty cycles; `pwm6`–`pwm8` are present in the chip but not connected on this board and do not hold a value. RPM feedback appears only on channels with a fan physically connected (`fanX_input`). The full pin arrangement of J4003 is in [Pinouts](../hardware/pinouts.md#fan-headers).

To control any channel by hand:

```bash
HWMON=$(grep -l nct6686 /sys/class/hwmon/hwmon*/name | head -1 | xargs dirname)
echo 1 | sudo tee "$HWMON/pwm4_enable"    # manual mode
echo 100 | sudo tee "$HWMON/pwm4"         # ~40% duty
```

## Manual PWM control

Quick check that the whole chain works — set the CPU_FAN1 fan to ~50%:

```bash
HWMON=$(grep -l nct6686 /sys/class/hwmon/hwmon*/name | head -1 | xargs dirname)

echo 1 | sudo tee "$HWMON/pwm2_enable"    # manual mode
echo 128 | sudo tee "$HWMON/pwm2"         # 128/255 = 50%
sleep 2
cat "$HWMON/fan2_input"                   # RPM should drop noticeably
```

Reference values for a typical 4-wire fan:

| PWM | RPM |
|---|---|
| 255 | ~4300 |
| 128 | ~2600 |
| 64 | ~1640 |

!!!warning "PWM does not survive a reboot"
    Neither the manual duty cycle nor manual mode persists across reboots (the BIOS regains control every boot). For a permanent setup, use a fan curve daemon — see below.

## The bc250fc toolkit

[Naster17/bc250fc](https://github.com/Naster17/bc250fc) packages the whole setup into an installer and two scripts:

```bash
git clone https://github.com/Naster17/bc250fc.git
cd bc250fc
sudo sh install.sh
```

`install.sh` detects the package manager (apk/apt/dnf/pacman), installs the build dependencies and `lm_sensors`, builds and loads `nct6687` if the kernel does not ship it, blacklists the conflicting `nct6683`, and makes the module load persistent. It then installs:

- **`fan`** — manual control CLI for every wired channel:
  ```bash
  sudo fan 95      # CPU_FAN1 to 95% (stops the curve daemon automatically)
  sudo fan status  # all channels: PWM / RPM / current mode
  sudo fan auto    # hand CPU_FAN1 back to the curve daemon
  sudo fan 4 60    # J4003 fan on channel 4 to 60%
  ```
- **`bc250-fan-curve`** — a daemon that reads the GPU edge temperature every 1 s and applies a fan curve to CPU_FAN1. It installs the matching init unit — OpenRC (Alpine, Gentoo) or systemd (Fedora, Bazzite, Arch, Debian) — and starts at boot. J4003 channels are left at their manual values.

On immutable Bazzite images the module install needs the [atomic-distro workarounds](sensors.md#immutable-atomic-distros-bazzite-silverblue-fedora-coreos).

### The default curve

The stock curve follows the **GPU edge temperature** — on this board it is the most load-sensitive sensor and tracks silicon heat under GPU-heavy workloads (gaming, LLM inference) better than the CPU socket sensor:

| GPU edge temp | PWM | ≈ Fan speed |
|---|---|---|
| < 40 °C | 90 | ~35 % |
| 40–49 °C | 110 | ~43 % |
| 50–59 °C | 140 | ~55 % |
| 60–69 °C | 175 | ~69 % |
| 70–79 °C | 215 | ~84 % |
| ≥ 80 °C | 255 | 100 % |

A hard safety net forces 100 % above 95 °C regardless of the curve. To tune the thresholds, edit `curve()` in `/usr/local/bin/bc250-fan-curve` and restart the service (`rc-service bc250-fan-curve restart` / `systemctl restart bc250-fan-curve`). Keep the ≥ 80 °C branch at 255 — it is the safety ramp.

!!!tip "Why GPU edge and not CPU temperature"
    The Zen 2 cores on the BC-250 are rarely the hottest part under GPU-heavy load, and the board's "CPU Socket" sensor reads 0 when unpopulated. The GPU edge sensor (`/sys/class/drm/card0/device/hwmon/hwmon*/temp1_input`) reacts within seconds to GPU load. If a different sensor is preferred, change `TEMP=` at the top of the script.

## CoolerControl (GUI alternative)

If a GUI with drag-and-drop curves is preferred, [CoolerControl](https://coolercontrol.org) works with the `nct6687` hwmon device — see [Sensors & Monitoring](sensors.md#coolercontrol-gui-for-sensor-monitoring-and-fan-curves) for installation. The bc250fc daemon and CoolerControl should not run at the same time; pick one.

## BIOS vs software control

| | BIOS "Full Speed" | BIOS "Customize" | Software curve |
|---|---|---|---|
| Noise | worst | OK | OK |
| Reacts to GPU load | no (board sensor) | partially | yes (GPU edge) |
| Survives reboot | yes | yes | via service |
| Survives OS crash/hang | yes | yes | **no** |

!!!warning "Software fan control stops if the OS hangs"
    The PWM setting stays at its last value when Linux wedges. If the system hangs under load, the fan will not spin up. Keep the curve conservative (the bc250fc defaults ramp to 100% by 80 °C) and consider the BIOS "Full Speed" mode while stress-testing new overclocks.

---

## Troubleshooting

### Fan ignores manual PWM values

**Symptoms:**
- Writing to `pwmX` has no effect, the fan speed does not change
- The duty cycle reads back its old value a second later

**Solution:**
1. Stop the curve daemon — it overwrites the channel every second (`fan <percent>` does this automatically, or stop the service).
2. Make sure the channel is in manual mode: `echo 1 | sudo tee "$HWMON/pwmX_enable"`.
3. If writes are still ignored, check which driver is loaded: `lsmod | grep nct668`. The in-kernel `nct6683` driver is read-only — swap to `nct6687 force=true` (see [Sensors & Monitoring](sensors.md#option-b-full-pwm-fan-control-nct6687-recommended)).

### `nct6683` and `nct6687` loaded together

**Symptoms:**
- Sensor readings flicker or disappear, PWM writes fail
- `dmesg` shows both drivers claiming the same chip

**Solution:**
1. Blacklist one of the two drivers — for PWM control keep `nct6687`:
   ```bash
   echo blacklist nct6683 | sudo tee /etc/modprobe.d/nct6683-blacklist.conf
   ```
2. Unload the other one: `sudo rmmod nct6683`, then reload `nct6687 force=true`.

### PWM does not survive reboot or suspend

**Symptoms:**
- After a reboot the fan runs at BIOS speed again; `pwmX_enable` is back to `2`

**Solution:**
1. This is expected — duty cycle and manual mode are volatile. Run the bc250fc service (it reapplies the curve at boot).
2. For suspend/resume, restart the service from your resume hook.

### `error: nct6686 hwmon not found`

**Symptoms:**
- `fan` exits immediately with `nct6686 hwmon not found`

**Solution:**
1. Check `lsmod | grep nct6687` — the module is probably not loaded.
2. Load it with `sudo modprobe nct6687 force=true`; the `force=true` parameter is required because the chip's customer ID is not auto-detected.
3. Check `dmesg | tail` if the load fails.

### Fedora: kernel headers not found during module build

**Symptoms:**
- `make` for `nct6687d` fails with "No such file or directory" pointing into `/lib/modules/$(uname -r)/build`

**Solution:**
1. Install headers matching the running kernel: `sudo dnf install kernel-devel-$(uname -r)`.
2. After every kernel update, rebuild the module (rerun the bc250fc installer).

### btop shows 0°C CPU temperature after loading nct6687

**Symptoms:**
- btop's CPU box shows `0°C`, sometimes flipping to a real value between launches
- `sensors` shows a correct `Tctl` under `k10temp-pci-00c3`

**Root cause:** the `nct6687` driver adds the board's temperature sensors to hwmon, including an unpopulated **`CPU Socket`** sensor that reads exactly **0**. btop's auto-selection grabs any sensor with "cpu" in its name and can pick that one. Setting `cpu_sensor = "k10temp"` does not help: btop keys sensors by their full `chip/label` name, so the short value matches nothing and the broken auto-pick stays active.

**Solution:**
1. Use the full sensor name in `~/.config/btop/btop.conf`:
   ```toml
   cpu_sensor = "k10temp/Tctl"
   ```
2. Restart btop. The exact names btop knows are listed in its options menu (CPU sensor dropdown).
