# Cooling Solutions

The BC-250 requires active cooling for gaming and desktop use. This guide covers tested cooling solutions and best practices.

## Stock Heatsink Limitations

### Stock Configuration

- **Type:** Passive aluminum fin stack heatsink
- **Fin Orientation:** Vertical, front-to-back
- **Design Purpose:** Rack-mounted passive or low-airflow cooling
- **Desktop Use:** Inadequate without active airflow

!!!danger "Stock Cooling Is Insufficient"
    The stock heatsink alone will cause thermal throttling and system instability during gaming. Active cooling is **required**.

## Temperature Targets

### Safe Operating Temperatures

| Component | Idle | Light Load | Gaming | Maximum |
|-----------|------|------------|--------|---------|
| GPU/APU Edge | 40-50°C | 50-60°C | 65-80°C | 90°C |
| CPU (Tctl) | 45-55°C | 55-65°C | 70-85°C | 95°C |
| Memory (underside) | 40-55°C | 50-65°C | 55-70°C | 80°C |

!!!success "Ideal Gaming Temps"
    Aim for GPU temperatures of 70-80°C during gaming for optimal performance and longevity.

!!!warning "Thermal Throttling"
    Above 85°C GPU temperature, the system may throttle performance. Above 90°C, instability and crashes can occur.

## Recommended Cooling Solutions

### Option 1: Arctic P12 Max / P12 Pro (Most Popular)

**Specifications:**
- **Model:** Arctic P12 Max or P12 Pro
- **Size:** 120mm x 25mm
- **Speed:** Up to 3300 RPM (Max), 2100 RPM (Pro)
- **Static Pressure:** 6.9 mm H2O (both models)
- **Airflow:** 73.3 CFM (Max), 68.9 CFM (Pro)
- **Noise:** 52.5 dB(A) at max (Max), 37.8 dB(A) at max (Pro)

**Performance:**
- GPU temps: 65-75°C during gaming
- Excellent static pressure for fin arrays
- Good price/performance ratio

**Setup:**
- Mount directly over heatsink fins
- Use 3D printed shroud or zip ties
- Connect to PWM header for speed control

!!!tip "Community Favorite"
    Both Arctic P12 Max and P12 Pro are highly recommended by the community due to excellent static pressure at a low price. The P12 Pro is more readily available in most regions.

### Option 2: Arctic P14 PWM

**Specifications:**
- **Model:** Arctic P14 PWM
- **Size:** 140mm x 25mm
- **Speed:** Up to 1700 RPM
- **Static Pressure:** 2.40 mm H2O
- **Airflow:** 72.8 CFM
- **Noise:** 38 dB(A) at max

**Performance:**
- GPU temps: 70-80°C during gaming
- Quieter than P12 Max
- Requires larger mounting solution

**Setup:**
- Mount with adapter or custom shroud
- Covers more heatsink area than 120mm
- Better for low-noise builds

### Option 3: Noctua NF-A12x25

**Specifications:**
- **Model:** Noctua NF-A12x25 PWM
- **Size:** 120mm x 25mm
- **Speed:** Up to 2000 RPM
- **Static Pressure:** 2.34 mm H2O
- **Airflow:** 60.1 CFM
- **Noise:** 22.6 dB(A) at max

**Performance:**
- GPU temps: 70-85°C during gaming
- Exceptional build quality
- Very quiet operation
- Lower static pressure than Arctic P12 Max

**Setup:**
- Mount directly over heatsink
- Best for quiet builds
- May need higher fan speed than Arctic

!!!info "Premium Choice"
    Noctua fans are higher quality and quieter but cost 2-3x more than Arctic fans. Performance is similar with Arctic P12 Max.

### Option 4: Dual Fan Setup

**Configuration:**
- **Primary Fan:** 120mm over center of heatsink
- **Secondary Fan:** 120mm or 80mm for RAM/VRM cooling

**Benefits:**
- Lower primary fan speeds = quieter
- Better RAM cooling (memory gets hot!)
- Improved overall system cooling
- Redundancy if one fan fails

**Recommended Combinations:**
- 2x Arctic P12 Max
- Arctic P14 + Arctic P12
- Noctua NF-A12x25 + 80mm fan

**Wiring:**
- Use fan splitter cable for single PWM control
- Or connect second fan to J4003 header

### Option 5: Tower Cooler Conversion

Some users have successfully mounted AM4 tower coolers:

**Compatible Coolers:**
- Thermalright Peerless Assassin
- Various AM4/AM5 coolers with custom mounting

**Pros:**
- Excellent cooling performance
- Quiet operation
- Uses existing hardware

**Cons:**
- Requires custom mounting solution
- May block M.2 slot or other components
- More complex installation

!!!warning "Advanced Modification"
    Tower cooler conversions require fabricating custom mounting brackets. Not recommended for beginners.

## Heatsink Modifications

### Fin Straightening

The stock heatsink often has bent fins that impede airflow. Carefully straightening bent fins can improve airflow.

**Process:**
- Work systematically through fin stack
- Be gentle - aluminum is very soft and bends easily
- Avoid forcing fins apart with tools

**Benefit:** 5-10°C temperature improvement if many fins are bent

### Fin Removal (Optional)

Some users remove center fins to improve fan contact with the heatsink.

!!!danger "High Risk Modification"
    Fin removal is **IRREVERSIBLE** and can damage your board if done incorrectly. Only attempt if absolutely necessary.

**Recommended Method:**
- **Manual removal by pulling/tearing:** Fins can be cleanly pulled apart by hand
- **MUST remove heatsink from board first** - never modify while attached!
- Work slowly and carefully to avoid bending adjacent fins

**NOT Recommended:**
- Dremel cutting (creates dangerous metal shavings)
- Hacksaw cutting (imprecise, messy)
- Any method that creates metal debris near the board

**Alternative:** Use a 3D printed fan shroud instead - no heatsink modification needed.

**Temperature Impact:** 10-15°C improvement, but similar gains possible with proper fan shroud

### Thermal Paste Replacement

The thermal paste on used BC-250 boards is often dried out.

**Recommended Thermal Paste:**
- Arctic MX-4 (good value)
- Arctic MX-6 (newer formula)
- Thermal Grizzly Kryonaut (premium)
- Noctua NT-H1 (reliable)
- Thermalright TFX (budget option)

**Application Method:**
1. Remove heatsink (4 screws)
2. Clean old paste with isopropyl alcohol
3. Apply small dot (pea-sized) of new paste to APU die
4. Remount heatsink with even pressure
5. Tighten screws in X pattern

**Temperature Impact:** 5-10°C improvement if old paste was dried

!!!tip "Use Quality Paste"
    Avoid cheap thermal paste. Quality paste lasts years. PTM7950 phase-change material is also popular.

### Memory Thermal Pad Replacement

GDDR6 memory chips on the underside can get very hot.

**Symptoms of Hot Memory:**
- System crashes during extended gaming
- Instability after 30-60 minutes
- Memory errors

**Solution:**
1. Remove board from case
2. Remove old thermal pads (if present)
3. Apply new thermal pads (1.5mm-2mm thick)
4. Attach aluminum plate or heatsink to underside
5. Optional: Add fan for active cooling

**Thermal Pad Recommendations:**
- Thermalright Odyssey (high performance)
- Arctic Thermal Pad (good value)
- Gelid GP-Ultimate (premium)

## Fan Mounting Options

### Option 1: 3D Printed Shroud

Many community-designed fan shrouds are available on Printables:

**Popular Designs:**
- [BC-250 Fan Shroud by User1](https://www.printables.com)
- [Compact Console Case](https://www.printables.com)
- [Dual Fan Mount](https://www.printables.com)

**Advantages:**
- Custom fit for board
- Integrated mounting for fans
- Can include case design
- No modification to heatsink needed

**Printing Requirements:**
- PLA or PETG filament
- 0.2mm layer height
- 20-30% infill

### Option 2: Direct Fan Mount

Mount fan directly using zip ties.

**Zip Tie Method:**
1. Position fan over heatsink center
2. Thread zip ties through fan mounting holes
3. Loop around heatsink fins or board mounting points
4. Tighten evenly
5. Trim excess zip tie length

!!!warning "Do Not Screw Into Heatsink"
    Do not drill holes in the heatsink fins to screw fans directly. The aluminum is soft and the fins are thin - this can damage the heatsink and reduce cooling efficiency. Use zip ties or a 3D printed shroud instead.

### Option 3: Cardboard/Foam Shroud

Quick DIY solution using cardboard or foam board.

**Materials:**
- Cardboard or foam core board
- Hot glue or duct tape
- Box cutter

**Process:**
1. Cut cardboard to create air duct from fan to heatsink
2. Glue/tape to create shroud around fan and heatsink
3. Ensure no air gaps
4. Mount fan to shroud

**Pros:** Free, fast, adjustable
**Cons:** Not durable, not aesthetically pleasing

## Fan Control

### PWM Control with nct6687

The BC-250 uses the NCT6686/6687 Super I/O chip for fan control.

**Driver Installation:**

```bash
# Load kernel module
echo 'nct6687' | sudo tee /etc/modules-load.d/nct6687.conf

# Rebuild initramfs
sudo dracut --regenerate-all --force  # Fedora
sudo mkinitcpio -P  # Arch
sudo update-initramfs -u  # Debian/Ubuntu

# Reboot
sudo reboot
```

**Verify:**
```bash
sensors
# Should show nct6687-isa-0a20 with fan speeds
```

### CoolerControl (GUI Fan Curves)

CoolerControl provides a GUI for creating custom fan curves.

**Installation:**

```bash
# Bazzite
ujust install-coolercontrol

# Fedora
sudo dnf copr enable terra/terra
sudo dnf install liquidctl coolercontrol

# Arch
yay -S coolercontrol
```

**Configuration:**
1. Launch CoolerControl
2. Select BC-250 fan header
3. Create custom curve (e.g., 30% at 50°C, 100% at 80°C)
4. Apply and test

### BIOS Fan Settings

The BIOS offers three fan modes:

**1. Default Mode:**
- Targets high temperatures
- Fans run at 40% minimum
- Not recommended (inadequate cooling)

**2. Full Speed Mode:**
- Fans at 100% constantly
- Simplest and safest option
- Noisy but effective

**3. Customize Mode:**
- Set custom temperature thresholds
- Define fan speeds at each threshold
- More granular than Default
- Can conflict with OS-level control

!!!warning "BIOS vs OS Control"
    Do not use both BIOS Customize mode and CoolerControl simultaneously. They will fight for control.

### Manual Fan Control

Set fan speed manually (for testing):

```bash
# Set fan 1 to 80% speed
echo 80 | sudo tee /sys/class/hwmon/hwmon*/pwm1

# Set to 100% (255 = full speed)
echo 255 | sudo tee /sys/class/hwmon/hwmon*/pwm1
```

## Cooling Solutions by Budget

| Budget | Solution | Expected Temps |
|--------|----------|----------------|
| **Minimal** | Single Arctic P12, zip tie mount, cardboard shroud | 75-85°C |
| **Budget** | Arctic P12 Max, 3D printed shroud, new thermal paste | 70-80°C |
| **Standard** | Dual Arctic P12, custom shroud, thermal paste + pads | 65-75°C |
| **Premium** | Noctua fans, aluminum case, PTM7950, RAM cooling | 60-70°C |
| **Enthusiast** | Tower cooler conversion, custom water cooling | 55-65°C |

## Cooling for Different Use Cases

### Gaming Build
- **Requirement:** 70-80°C sustained
- **Solution:** Arctic P12 Max or P14, BIOS full speed or custom curve

### Silent Build
- **Requirement:** <30 dB(A) noise
- **Solution:** Noctua NF-A12x25, custom fan curve (max 60%)
- **Trade-off:** Higher temps (75-85°C)

### Compact Build
- **Requirement:** Small form factor
- **Solution:** Single 120mm fan, integrated case design
- **Challenge:** Less cooling headroom

### LLM/Compute Build
- **Requirement:** 24/7 operation, reliability
- **Solution:** Dual 120mm fans, full speed, focus on dust filtering
- **Note:** Longevity over noise

## Troubleshooting Cooling Issues

### High Temps (>85°C) During Gaming

**Causes:**
- Insufficient fan speed
- Poor fan placement
- Dried thermal paste
- Blocked airflow
- High ambient temperature

**Solutions:**
1. Increase fan speed to 80-100%
2. Check fan is positioned over heatsink center
3. Replace thermal paste
4. Remove case panels for testing
5. Ensure room temperature <25°C

### System Crashes After 30 Minutes

**Symptoms:**
- Stable initially, crashes later
- Crashes during demanding games

**Likely Cause:** Memory overheating

**Solutions:**
1. Add thermal pads to memory chips (underside)
2. Add secondary fan for RAM cooling
3. Reduce VRAM allocation (4GB -> 512MB)
4. Improve case airflow

### Fan Not Spinning

**Causes:**
- Fan not connected
- Wrong header (use J1 or J4003)
- Fan header disabled in BIOS
- Faulty fan

**Solutions:**
1. Check fan connector is firmly seated
2. Verify fan works on another system
3. Check BIOS fan settings
4. Test with another fan

### Fan Speed Fluctuations

**Causes:**
- Aggressive fan curve
- Temperature sensor fluctuations
- Insufficient power

**Solutions:**
1. Use smoother fan curve (longer intervals)
2. Enable hysteresis in fan curve
3. Check PSU can deliver power

## See Also

- [Hardware Specifications](specifications.md)
- [Power Requirements](power.md)
- [System Configuration](../system/governor.md)
- [Troubleshooting Guide](../troubleshooting/display.md)
