# Stability Troubleshooting Guide

This guide addresses system crashes, freezes, random reboots, and instability issues on the BC-250. These problems are often interrelated and can stem from hardware, BIOS configuration, kernel compatibility, or power delivery issues.

---

## Quick Diagnostic Checklist

Before diving into specific issues, check these common causes:

1. **Kernel version**: Are you running kernel 6.15.0-6.15.6 or 6.17.8+? (Known to cause GPU crashes)
2. **BIOS settings**: Did you clear CMOS after flashing BIOS?
3. **VRAM allocation**: Are you using 512MB dynamic with ZRAM enabled?
4. **IOMMU**: Is it disabled in BIOS?
5. **Governor voltage**: Are your voltage settings stable for your frequency?
6. **Power supply**: Is your PSU sufficient and stable?

---

## System Crashes and Freezes

### GPU-Related System Crashes

**Symptoms**: Entire system freezes or crashes during GPU load, requires hard reboot

**Root Cause**: On the BC-250, the GPU and CPU are tightly integrated. When the GPU crashes, the entire system crashes because the driver attempts to reset the GPU, which is not possible on this APU architecture.

**Quote from community**:
> "The GPU crashes, the whole system crashes, and requires a reboot. Unfortunately, since these chips are based on the GPU, when the GPU crashes, the whole system crashes."

**Solutions**:

1. **Check kernel version** (Most critical)
   ```bash
   uname -r
   ```
   - **AVOID kernel 6.15.0-6.15.6 and 6.17.8+** - Known to cause random GPU crashes under load
   - **Recommended**: 6.15.7-6.17.7 (best performance) or 6.12.x-6.14.x LTS (stable)
   - If on broken version, install working kernel immediately

2. **Verify governor voltage stability**
   - Run benchmark: `vkmark` or `superposition`
   - If it crashes, your voltage is too low for your frequency
   - Increase voltage by 10-15mV and test again
   - Quote: "Low voltage leads to a crash in almost every case, not an oberon fault"

3. **Check GPU temperature throttling**
   ```bash
   sensors
   ```
   - GPU should stay below 85°C under load
   - If hitting 90°C+, you're likely thermal throttling
   - Some boards freeze at 60-65°C if cooling is inadequate

4. **Test with locked frequency**
   - Edit `/etc/oberon-config.yaml`:
   ```yaml
   opps:
     - frequency:
       - min: 1500
       - max: 1500
     - voltage:
       - min: 900
       - max: 900
   ```
   - Restart governor: `systemctl restart oberon-governor`
   - If stable at locked frequency, it's a governor tuning issue

### Thermal-Related Freezes

**Symptoms**: System freezes or shuts off when temperatures exceed certain thresholds

**Community experience**:
> "Others told me their board simply turns off when overheated. Mine seems to have something above 60/65 degree (freeze, sometimes white screen). Maybe severe peaks or something?"

**Solutions**:

1. **Monitor temperatures during stress**
   ```bash
   watch -n 1 sensors
   ```
   - APU edge temp: Should stay below 85°C
   - Some boards are more temperature-sensitive than others

2. **Improve cooling**
   - Ensure fan is high static pressure (Arctic P12 Pro: 6.9mm H2O recommended)
   - Check thermal paste application on APU die
   - Consider PTM7950 phase change pad instead of paste
   - Verify heatsink contact pressure

3. **Check if thermal paste dried out**
   - Symptom: Rapid temperature spikes
   - Solution: Reapply thermal paste (or use PTM7950)

### Overclocking Instability

**Symptoms**: Crashes during benchmarks or gaming, artifacts, system freezes

**Diagnostic steps**:

1. **Find your stability limit**
   - Quote: "Run benchmark until it crashes, then throw like 10-15 mV on top"
   - Start conservative: 2000MHz @ 1000mV
   - Increase frequency in 50MHz increments
   - If crashes occur, add 10-15mV voltage

2. **Known frequency issues**
   - 980MHz: "Very wonky on any voltage"
   - 1000MHz: "Sometimes wonky but give it more voltage or change loadline in BIOS"
   - 700mV: Hard minimum voltage cap
   - 2230MHz+: Most boards require 1050mV+, some need 1100mV

3. **Example stable configurations** (varies by silicon lottery):
   ```
   1000MHz @ 700mV (conservative)
   1500MHz @ 850mV (balanced)
   2000MHz @ 1000mV (recommended max)
   2230MHz @ 1050mV (overclock, may be unstable)
   ```

4. **Community-tested safe points** (one user's example):
   ```
   MHz   mV
   350   570
   860   600
   1090  650
   1280  700
   1460  750
   1620  800
   1760  850
   1890  900
   2030  950
   2140  1000
   2230  1050
   ```
   Note: These are NOT universal - test your own board

### Governor-Related Instability

**Symptoms**: Random crashes during load changes, frequency spikes, voltage issues

**Common issues**:

1. **Governor not starting correctly**
   - Check status: `systemctl status oberon-governor`
   - If failing, governor may be installed in wrong location
   - Reinstall following distribution-specific guide

2. **Voltage too low at startup**
   - Some boards crash on "default governor settings"
   - Edit `/etc/oberon-config.yaml` to increase min voltage
   - Quote: "Hey all, my board crashes on default governor settings. Did I loose the silicon lottery?"

3. **Frequency/voltage mismatch**
   - Symptom: Crashes when GPU load increases/decreases
   - Solution: Reduce frequency range or increase voltage headroom
   - Example fix:
   ```yaml
   opps:
     - frequency:
       - min: 1000
       - max: 2000
     - voltage:
       - min: 750
       - max: 1000
   ```

---

## Random Reboots

### Power Supply Issues

**Symptoms**: System suddenly powers off and restarts, especially under load

**Causes**:

1. **Insufficient PSU wattage**
   - BC-250 can draw up to 235W during gaming (Cyberpunk with RT)
   - Add 20-30W for fans, storage, peripherals
   - **Recommended minimum**: 300W PSU
   - Meanwell LOP-300 is popular choice

2. **PSU voltage instability**
   - Some cheap PSUs cannot deliver stable 12V under varying load
   - Check PSU rail voltage if possible
   - Quote: "I have two. They run okay under 940 mV will heat up and shut off under load occasionally. Absolutely cannot handle the 1000 plus required to run the board at its full 2232 mhz"

3. **Poor cable connections**
   - Loose 8-pin PCIe power connector
   - Use 16 AWG wire minimum for custom builds
   - Ensure clean solder joints on power rails

**Solutions**:

1. Test with lower TDP:
   - Reduce max frequency to 1500MHz
   - Lower voltage to 850mV
   - If stable, it's a power delivery issue

2. Measure actual power draw:
   - Use power meter on PSU input
   - If approaching PSU limit, upgrade PSU

### Overheating Auto-Shutoff

**Symptoms**: System powers off cleanly when temperature exceeds threshold

**Solution**: This is actually protective behavior - improve cooling rather than disabling

---

## Kernel Panics

### Broken Kernel Versions - GPU Driver Failures

**Symptoms**: Kernel panics, GPU errors in dmesg, system crashes under GPU load on 6.15.0-6.15.6 or 6.17.8+

**Critical issue**: Kernel 6.15.0-6.15.6 and 6.17.8+ break GPU driver support for BC-250

**Solution**:

1. **Install working kernel (6.15.7-6.17.7 or 6.12-6.14 LTS)**

   **Arch/Manjaro**:
   ```bash
   # Option 1: Install working 6.15.7-6.17.7 kernel
   sudo pacman -S linux  # Check version is in working range
   # Option 2: Install LTS kernel for guaranteed stability
   sudo pacman -S linux-lts linux-lts-headers

   # Set as default in bootloader
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```

   **Fedora**:
   ```bash
   # Install older kernel
   sudo dnf install kernel-6.14.x

   # Set as default
   sudo grubby --set-default /boot/vmlinuz-6.14.x
   ```

   **CachyOS**: Use LTS kernel option during installation

2. **Check dmesg for GPU errors**
   ```bash
   dmesg | grep -i "amdgpu\|gpu\|drm"
   ```
   - Look for reset failures, initialization errors
   - If seeing "GPU reset failed" - kernel version issue

### ACPI Errors on Boot

**Symptoms**: System boots but crashes shortly after, ACPI errors in logs

**Quote from user**:
> "Live usb boots up most of the time but with ACPI errors. It stays on for a few seconds then crashes with black/green screen."

**Solutions**:

1. Clear CMOS and reset BIOS to defaults
2. Reflash BIOS if corruption suspected
3. Disable ACPI features in BIOS if available
4. Try different kernel boot parameters:
   ```
   acpi=off
   noapic
   ```

---

## Voltage-Related Instability

### Symptoms

- Artifacts in games (textures going black/missing)
- Random crashes under load
- System freezes when GPU frequency changes

### Finding Your Voltage Requirements

**Method 1: Progressive testing**
1. Start at safe voltage (1000mV)
2. Lower by 25mV increments
3. Test with 30-minute gaming session or benchmark
4. When crashes occur, add back 50mV for safety margin

**Method 2: Binary search**
1. Test at 700mV (minimum)
2. If crashes, test 850mV (midpoint)
3. Narrow down until you find minimum stable voltage
4. Add 25-50mV safety margin

### Voltage Warnings

**Do not exceed**:
- 1100mV for general use
- 1150mV absolute maximum (reduces lifespan, increases heat)

**Quote from community**:
> "I'd guess we are not super close to the voltage limit. If we're assuming it behaves similarly to other AMD APUs, I'd expect 1.1v and possibly 1.15v to be safe. Cooling just becomes harder the more voltage you push through."

### Voltage Drop Issues

Some boards experience voltage drops under load causing instability:

**BIOS fix** (if available):
> "A slightly adjusted bios gives the cpu/gpu a little more overhead when it comes to cpu/gpu droop, it now gives a little more voltage to stabilize."

**Governor workaround**:
- Set higher base voltage to account for droop
- Example: If you need 1000mV effective, set to 1025mV to account for drops

---

## ZRAM Conflicts with Dynamic VRAM

### The Issue

**Critical finding**:
> "If anyone is having issues with games crashing. I was having issues with RDR2 crashing when using ZRAM with the 512MB VRAM (dynamic allocation) set. Going to a fixed allocation of 10/6 seems to have resolved that."

**Explanation**: ZRAM (compressed swap in RAM) conflicts with dynamic VRAM allocation (512MB setting), causing memory allocation failures and crashes.

### Solutions

**Option 1: Disable ZRAM (Recommended for gaming)**
```bash
# Systemd-based distros
sudo systemctl stop zram-swap
sudo systemctl disable zram-swap

# Or remove package
sudo dnf remove zram  # Fedora
sudo pacman -R zram-generator  # Arch
```

**Option 2: Use fixed VRAM allocation**
- Enter BIOS (usually Delete or F2 on boot)
- Navigate to AMD CBS → UMA Frame Buffer Size
- Change from "Auto" (512MB) to fixed allocation:
  - 10GB CPU / 6GB GPU (gaming)
  - 12GB CPU / 4GB GPU (general use)
  - 8GB CPU / 8GB GPU (balanced)

**Option 3: Reduce ZRAM size**
```bash
# If you must use both, limit ZRAM
# Edit /etc/systemd/zram-generator.conf
[zram0]
zram-size = 4096  # 4GB instead of 8GB
```

**Quote from successful fix**:
> "Never mind I applied the setting and it works wonders so far no crashing anymore set the zram swap to 8gb"

---

## Memory Errors and RAM Issues

### Insufficient RAM Crashes

**Symptoms**:
- Games crash with "low system RAM" errors
- System becomes unresponsive when memory fills
- OOM (Out of Memory) killer activating

**Causes**:

1. **BIOS settings not sticking**
   - Quote: "BIOS settings don't stick after flashing via USB if CMOS isn't cleared"
   - Your 512MB dynamic setting may revert to 4GB or 8GB fixed
   - This leaves only 8-12GB for system RAM

2. **Swap not working properly**
   - Some users report swap failing to activate
   - ZRAM conflicts (see above section)

**Solutions**:

1. **Verify actual RAM allocation**
   ```bash
   free -h
   ```
   - Should show ~15-15.5GB total RAM with 512MB dynamic VRAM
   - If showing only 10-12GB, BIOS setting not applied

2. **Clear CMOS after BIOS flash** (Critical step)
   - Power off completely
   - Locate CMOS jumper or remove battery for 30 seconds
   - Restore jumper/battery
   - Boot to BIOS and reconfigure settings
   - Quote: "For anyone wondering, I would just like to reiterate the importance of clearing cmos after flashing the bios. I've just spent half a day diagnosing black screen issue, which ended up being caused by the non-clearance of cmos"

3. **Double-check BIOS settings after reboot**
   - Settings may appear to save but not actually apply
   - Reboot and verify in BIOS that 512MB is still selected
   - May require 2-3 CMOS clears to "stick"

4. **Enable proper swap**
   - If not using ZRAM, create traditional swap:
   ```bash
   # Create 8GB swap file
   sudo dd if=/dev/zero of=/swapfile bs=1G count=8
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile

   # Make permanent
   echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab
   ```

### RAM Timing/Frequency Issues

**Warning**: Unstable RAM settings can permanently corrupt BIOS

**Quote**:
> "Its not actually 450, its like 1750, but you can modify it. I wouldn't recommend it though, ppl over in the russian BC250 chat reported that unstable ram settings would frequently result in BIOS corruption, requiring the BIOS to be reflashed"

**If you must overclock RAM**:
1. Have hardware flasher ready (CH341A or Raspberry Pi Pico)
2. Back up working BIOS first
3. Test incrementally
4. If system fails to boot, CMOS clear may not be enough - reflash required

---

## Audio/Video Freezing

### Game Audio Stuttering

**Symptoms**: Audio clicks, pops, or stutters during gameplay

**Solutions**:

1. **Change audio sample rate**
   - Issue: Some games work better at 44.1kHz vs 48kHz
   - Quote: "I changed alsa and pulseaudio settings to make the sample rate 44.1khz rather than 48khz and all of a sudden sound worked fine - it wasn't choppy anymore - but all audio was pitched down"
   - Recommendation: Keep at 48kHz unless specific game requires change

2. **Use correct audio output**
   - Passive DisplayPort to HDMI: Audio works
   - Active DP to HDMI adapters: Audio often broken
   - USB audio: Most reliable for quality audio

3. **Audio-related performance issues**
   - Quote: "I saw someone on reddit saying to get better performance in linux to change the audio thing. Usually the menu was locked at 30 fps but after changing audio settings it was more than 90"
   - Try switching between PulseAudio and PipeWire
   - Adjust audio buffer sizes

### Video Freezing Then Crash

**Symptoms**: Screen freezes, then goes black after 10 seconds

**Quote**: "Sadly it did not work for me once I execute koboldcpp screen freeze and 10 seconds after screen goes black"

**Solutions**:
1. Check GPU memory allocation (see VRAM section)
2. Verify governor is active and responding
3. Monitor GPU temperature during freeze
4. Test with lower resolution/settings

---

## BIOS-Related Stability Issues

### IOMMU Instability

**Critical**: IOMMU is broken on BC-250 and causes crashes

**Quote**:
> "Make sure IOMMU is disabled in bios. Having it enabled can cause weird crashes and problems like this"

**How to disable**:
1. Enter BIOS
2. Navigate to Advanced → NB Configuration
3. Find IOMMU option
4. Set to **Disabled**
5. Save and exit

**If IOMMU option not visible**: Flash modded BIOS to unlock advanced settings

### BIOS Settings Not Persisting

**Symptoms**:
- Settings appear to save but revert on reboot
- System unstable despite "correct" BIOS configuration
- VRAM allocation not actually applied

**Root cause**: CMOS needs clearing after USB BIOS flash

**Complete fix procedure**:

1. **Clear CMOS properly**
   - Shut down completely (not just reboot)
   - Disconnect power
   - Locate CLR_CMOS jumper near BIOS chip
   - Short pins for 10 seconds OR remove CR2032 battery for 30 seconds
   - Restore jumper/battery
   - Reconnect power

2. **Reconfigure BIOS**
   - Boot to BIOS (usually Delete key)
   - Set UMA Frame Buffer Size: 512MB (Auto)
   - Disable IOMMU
   - Set fan profile (Full Speed recommended for testing)
   - Disable unused features
   - Save and exit

3. **Verify settings stuck**
   - Immediately reboot to BIOS
   - Check that 512MB is still selected
   - If reverted, repeat CMOS clear
   - May take 2-3 attempts

4. **Confirm in OS**
   ```bash
   # Check total RAM (should be ~15.5GB with 512MB VRAM)
   free -h

   # Check VRAM allocation
   glxinfo | grep "Video memory"
   # or
   vulkaninfo | grep -i memory
   ```

### BIOS Corruption from Unstable Settings

**Symptoms**: Board won't boot after changing RAM timings or voltages

**Prevention**:
- Never change RAM timing/frequency without hardware flasher available
- Test overclock changes one parameter at a time
- Keep notes of working configurations

**Recovery**:
1. Try CMOS clear (may not work for corruption)
2. If CMOS clear fails, hardware reflash required:
   - Use CH341A programmer or Raspberry Pi Pico
   - Flash known-good BIOS (P3.00 modded recommended)
   - See BIOS recovery guide for detailed steps

---

## Power State and Sleep Issues

### System "Freezes" When Sleeping

**Symptoms**: Screen goes black, appears frozen, but power button wakes it

**Quote**:
> "Bazzite freezes when its about to sleep, had to hard reset everytime. That's interesting, mine 'freezes' when it sleeps, but hitting the power button wakes it up."

**Explanation**: Board lacks proper sleep states, enters pseudo-sleep that looks like freeze

**Solutions**:
1. Disable sleep/suspend in power settings
2. Use "power button wakes" as intended behavior
3. Configure screen blanking instead of system sleep

### Sleep State Power Issues

**Quote**:
> "Yes, I've tried. I made CPU fall into idle states, unfortunately it doesn't save more than 2-3W. Best result that I've got is around 65W. Without proper sleep states on GPU, there is no way."

**Reality**: BC-250 lacks proper power state support in Linux
- Idle power: 65-85W (with governor)
- Cannot achieve low-power sleep states
- SMU (System Management Unit) doesn't support Linux sleep properly

---

## Systematic Stability Testing

### Step-by-Step Diagnosis Process

If experiencing general instability, follow this process:

**Phase 1: Baseline (Safe Configuration)**
1. Use kernel 6.12-6.14 LTS
2. Set BIOS: 512MB VRAM, IOMMU disabled
3. Clear CMOS, verify settings stick
4. Governor: 1500MHz @ 900mV locked (min=max)
5. Disable ZRAM
6. Test for 1 hour gaming

**Phase 2: If Baseline Stable**
- Enable governor frequency scaling
- Start at 1000-2000MHz range, 750-1000mV
- Test each increment for 30 minutes
- Gradually increase max frequency

**Phase 3: If Baseline Unstable**
- Check kernel version first
- Monitor temperatures during stress
- Test PSU with multimeter under load
- Verify BIOS flash was successful
- Consider hardware issue

### Stress Testing Tools

**GPU stress**:
```bash
# Benchmark tools
vkmark
superposition  # Unigine Superposition benchmark

# Continuous load
furmark  # Extreme stress test
```

**CPU stress**:
```bash
stress-ng --cpu 6 --timeout 300s
```

**Combined stress**:
```bash
# Run game for 1 hour
# Monitor with:
watch -n 1 'sensors; echo "---"; free -h'
```

### Monitoring During Testing

**Terminal 1 - Temperatures**:
```bash
watch -n 1 sensors
```

**Terminal 2 - Frequencies and voltages**:
```bash
watch -n 1 'cat /sys/class/drm/card0/device/pp_od_clk_voltage'
```

**Terminal 3 - Memory**:
```bash
watch -n 1 'free -h'
```

**Terminal 4 - System logs**:
```bash
journalctl -f | grep -i "error\|fail\|crash\|amdgpu"
```

---

## Common Error Messages

### "GPU reset failed"
- **Cause**: Kernel 6.15.0-6.15.6 or 6.17.8+, overclocking instability, or GPU crash
- **Fix**: Install working kernel (6.15.7-6.17.7 or 6.12-6.14 LTS), reduce frequency/increase voltage

### "Out of memory"
- **Cause**: VRAM exhausted, BIOS settings not applied, ZRAM conflict
- **Fix**: Verify 512MB VRAM set, disable ZRAM, check with `free -h`

### "Low system RAM"
- **Cause**: Fixed VRAM allocation using too much, BIOS settings reverted
- **Fix**: Set 512MB dynamic allocation, clear CMOS

### ACPI errors on boot
- **Cause**: BIOS corruption, incompatible kernel, hardware issue
- **Fix**: Reflash BIOS, try different kernel, clear CMOS

### "Adapter not found" / "Unknown graphics adapter"
- **Cause**: Mesa version too old, drivers not installed, VRAM not allocated
- **Fix**: Install Mesa 25.1.3+, verify RADV driver, check VRAM in BIOS

---

## Hardware Failure Indicators

### When to Suspect Hardware Problems

**Signs of actual hardware failure**:
1. Instability persists with all BIOS/kernel combinations
2. Board crashes even at stock 1500MHz @ 900mV
3. Artifacts appear even at low temperatures and safe voltages
4. Memory errors in memtest86+
5. Random shutoffs with known-good PSU

**Testing for hardware issues**:
1. **Memtest86+**: Test RAM/memory controller
2. **Swap PSU**: Borrow known-good PSU to test
3. **Test on mining BIOS**: Some boards more stable on original P2.00/P4.00
4. **Visual inspection**: Look for damaged components, scorch marks
5. **Compare with second board**: If available

**Quote on hardware variance**:
> "I find that BC-250s come in two flavors those that achieve 27 tok/s and those that crank 34 tok/s on the model above."

Silicon lottery is real - some boards are less stable than others.

---

## Prevention Best Practices

### Setup Recommendations

1. **Always clear CMOS after BIOS flash**
2. **Verify settings after every reboot** (first few boots)
3. **Test stability before gaming** (run benchmark first)
4. **Start conservative, increase gradually** (frequency/voltage)
5. **Monitor temps in first few gaming sessions**
6. **Keep hardware flasher available** (CH341A ~$5 on AliExpress)

### Maintenance

1. **Check thermal paste every 6 months** (or use PTM7950)
2. **Clean heatsink fins** from dust buildup
3. **Verify fan operation** regularly
4. **Monitor kernel updates** (avoid 6.15.0-6.15.6 and 6.17.8+)
5. **Back up working configurations** (BIOS settings, governor config)

### Documentation

Keep notes of:
- Working frequency/voltage combinations
- BIOS version and settings
- Kernel version when stable
- Governor configuration file
- Any custom modifications

---

## Quick Reference: Stable Configurations

### Conservative (Maximum Stability)
- Kernel: 6.12 LTS
- Governor: 1500MHz @ 900mV (locked)
- VRAM: 512MB dynamic
- ZRAM: Disabled
- Expected temps: 65-75°C gaming

### Balanced (Recommended)
- Kernel: 6.13 or 6.14
- Governor: 1000-2000MHz, 750-1000mV
- VRAM: 512MB dynamic
- ZRAM: Disabled or 4GB max
- Expected temps: 70-80°C gaming

### Performance (Requires Good Cooling)
- Kernel: 6.14 patched
- Governor: 700-2230MHz, custom voltage curve
- VRAM: 512MB dynamic
- ZRAM: Disabled
- Expected temps: 75-85°C gaming
- Requires: High static pressure fan (Arctic P12 Pro or better)

---

## Getting Help

If problems persist after trying these solutions:

1. **Gather information**:
   ```bash
   # Create diagnostic report
   {
     echo "=== System Info ==="
     uname -a
     echo "=== GPU Info ==="
     lspci | grep VGA
     echo "=== Memory ==="
     free -h
     echo "=== Temperatures ==="
     sensors
     echo "=== Governor Status ==="
     systemctl status oberon-governor
     echo "=== Recent Errors ==="
     journalctl -p err -n 50
   } > bc250-diagnostic.txt
   ```

2. **Check Discord** #bc250-chat channel
3. **Search GitHub issues** on mothenjoyer69/bc250-documentation
4. **Provide specific details**: Exact error messages, kernel version, BIOS version, your configuration

---

## Summary: Most Common Fixes

| Problem | Most Likely Fix |
|---------|----------------|
| Random GPU crashes | Install working kernel (6.15.7-6.17.7 or 6.12-6.14 LTS) |
| BIOS settings not sticking | Clear CMOS after flashing |
| Games crashing with ZRAM | Disable ZRAM or use fixed VRAM allocation |
| General instability | Disable IOMMU in BIOS |
| Overclocking crashes | Increase voltage by 10-15mV |
| System won't boot after BIOS flash | Clear CMOS, may need hardware reflash |
| Thermal shutoffs | Improve cooling, check thermal paste |
| Low memory crashes | Verify 512MB VRAM applied, check with `free -h` |

**Remember**: The BC-250 requires patience and methodical troubleshooting. Start with safe settings and gradually optimize. Keep notes and backups of working configurations.
