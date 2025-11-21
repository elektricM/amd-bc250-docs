# Display & No Signal Issues

Display problems are the #1 issue new BC-250 users encounter. This guide covers all known display-related issues and their solutions.

---

## Quick Diagnosis

**Symptoms checklist:**

- [ ] Complete black screen, no signal
- [ ] Display works in BIOS but not in OS
- [ ] Display flickers or goes black intermittently
- [ ] Display works on first boot, fails after reboot
- [ ] No display after BIOS flash

**Most common causes:**

1. Missing GPU drivers + no nomodeset
2. BIOS settings not applied (CMOS not cleared)
3. Incompatible display adapter (active DP-HDMI)
4. IOMMU enabled in BIOS
5. Kernel 6.15+ driver issues

---

## No Display During Installation

### Problem: Black Screen When Booting Installer

**Symptoms:**
- USB boots, but screen goes black
- No installer appears
- Monitor shows "No Signal"

**Cause:**
Installer doesn't have BC-250 GPU drivers. Linux framebuffer (KMS) attempts to initialize GPU and fails.

**Solution: Use nomodeset**

!!!tip "Fedora: Use Basic Graphics Mode"
    In Fedora installer boot menu, select "Troubleshooting" → "Install in Basic Graphics Mode". This enables nomodeset automatically.

**Manual method (any distro):**

1. At GRUB boot menu, press **e** to edit
2. Find line starting with `linux` or `linuxefi`
3. Go to end of that line
4. Add a space, then type: `nomodeset`
5. Press **Ctrl+X** to boot

Example:
```bash
# Before:
linux /vmlinuz root=live:CDLABEL=Fedora quiet

# After:
linux /vmlinuz root=live:CDLABEL=Fedora quiet nomodeset
```

**What nomodeset does:**
- Disables kernel mode setting (KMS)
- Forces fallback to VESA/UEFI framebuffer
- Allows display without GPU drivers
- Enables installation to proceed

!!!warning "Must Remove After Install"
    Once Mesa drivers are installed, nomodeset MUST be removed or GPU acceleration won't work.

---

## No Display After Installation

### Problem: Boots But Black Screen

**Symptoms:**
- Installation completed successfully
- System boots (fan spins, LED lights up)
- But display shows nothing

**Cause:**
Same as installer - no GPU drivers yet.

**Solution:**

**Option 1: Boot with nomodeset (temporary fix)**

1. At GRUB boot menu (shows briefly), press **e**
2. Find `linux` line
3. Add `nomodeset` at end
4. Press **Ctrl+X** to boot
5. Install drivers (see [driver installation guide](../linux/distributions.md))
6. Remove nomodeset from GRUB permanently

**Option 2: Install drivers from recovery mode**

Some distros offer recovery/safe mode that includes basic display drivers.

**Permanent fix:**

After booting with nomodeset:

```bash
# Install drivers (distro-specific)
# Fedora:
curl -s https://raw.githubusercontent.com/mothenjoyer69/bc250-documentation/refs/heads/main/fedora-setup.sh | sh

# Then remove nomodeset:
sudo nano /etc/default/grub
# Remove 'nomodeset' from GRUB_CMDLINE_LINUX_DEFAULT
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot
```

---

## Display Works in BIOS, Fails in OS

### Problem: Can Access BIOS But Not Boot OS

**Symptoms:**
- BIOS menu displays fine
- Boot process starts
- Screen goes black when OS loads

**Cause:**
OS tries to initialize GPU with KMS, fails because drivers missing or incompatible.

**Solution:**
Add nomodeset to kernel parameters (same as above sections).

**If drivers ARE installed:**

Check kernel version:
```bash
uname -r
```

If kernel is 6.15 or higher:
```bash
# Downgrade to 6.12-6.14
# Fedora:
sudo dnf install kernel-6.14.x

# Update GRUB to use older kernel as default
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

!!!danger "Kernel 6.15+ Breaks GPU"
    Kernel 6.15+ has driver incompatibility with BC-250. Always use 6.12-6.14 LTS.

---

## No Display After BIOS Flash

### Problem: Flashed BIOS, Now No Display

**Symptoms:**
- USB BIOS flash appeared successful
- Board powers on (fan spins)
- No display at all, not even BIOS

**Causes:**
1. CMOS not cleared (most common)
2. BIOS flash corrupted
3. Display adapter incompatibility
4. IOMMU enabled by default in new BIOS

**Solutions (in order of likelihood):**

**1. Clear CMOS (fixes 90% of cases)**

!!!success "This Usually Fixes It"
    Most "bricked after flash" boards are just waiting for CMOS clear.

Method A: Remove battery
1. Power off, unplug everything
2. Remove CR2032 CMOS battery
3. Wait 60 seconds (or press power button 5 times with battery out)
4. Replace battery
5. Plug back in, power on

Method B: Use CMOS jumper
1. Power off, unplug
2. Locate CMOS clear jumper (check board pinout)
3. Move jumper to clear position
4. Wait 10 seconds
5. Return jumper to normal position
6. Power on

**2. Try Different Display Adapter**

- If using active DP-HDMI adapter, try passive
- If using passive, try native DP cable
- Try different monitor if possible

**3. Reflash BIOS**

If board still doesn't display:
- BIOS may be corrupted
- Reflash using USB method again
- Or use hardware programmer (CH341A)

[BIOS recovery guide →](../bios/recovery.md)

---

## Intermittent Black Screens

### Problem: Display Works, Then Randomly Goes Black

**Symptoms:**
- Display works initially
- Goes black during gaming or after some time
- Requires reboot to restore

**Possible causes:**

**1. Overheating**

Check temperatures:
```bash
sensors
```

If GPU >90°C:
- Improve cooling (better fans, straighten fins)
- Lower clock speeds in governor config
- Increase fan speed

**2. Unstable Overclock**

If using custom governor settings:

```bash
# Edit /etc/oberon-config.yaml
# Reduce max_frequency or increase voltage
sudo systemctl restart oberon-governor
```

**3. Power Supply Issues**

- Insufficient wattage (need 300W+)
- Bad 8-pin connection
- Voltage sag under load

Test with lower power limit:
```bash
# Limit max frequency to reduce power draw
# Edit /etc/oberon-config.yaml
max_frequency: 1500
```

**4. Display Adapter Overheating**

Some active adapters overheat and fail.
- Switch to passive adapter
- Ensure adapter has airflow

---

## Display Flickering / Artifacts

### Problem: Image Has Visual Glitches

**Symptoms:**
- Screen flickers
- Colored artifacts
- Horizontal lines
- Partial display corruption

**Causes & Solutions:**

**1. Bad Display Cable**
- Try different DP or HDMI cable
- Ensure cable supports your resolution/refresh rate

**2. Refresh Rate Too High**
- Some passive adapters limited to 60Hz
- Reduce refresh rate in display settings:
  ```bash
  # KDE: System Settings → Display
  # GNOME: Settings → Displays
  ```

**3. VRAM Allocation Issues**

If using fixed VRAM allocation and seeing artifacts:
- May need more VRAM
- Switch from 4GB to 6GB or 8GB VRAM in BIOS

**4. Unstable GPU Overclock**

Reduce frequency or increase voltage in governor config.

---

## Display Works But No BIOS Menu

### Problem: Can't Access BIOS Setup

**Symptoms:**
- Display shows boot process
- Can't enter BIOS menu
- Del/F2 keys don't work

**Solutions:**

**1. Press Del EARLIER**
- Start pressing Del immediately when powering on
- Some boards have very short window

**2. Try Different Keys**
- Del (most common)
- F2
- F12
- Esc

**3. Boot Too Fast**
- Add delay to GRUB timeout
- Or hold Shift during boot for GRUB menu
- Then reboot from GRUB to BIOS

**4. Display Adapter Issue**
- Some adapters don't initialize fast enough for BIOS
- Try different adapter
- Try native DP cable

**5. Keyboard Not Detected**
- Try different USB port
- Try USB 2.0 instead of 3.0
- Ensure keyboard plugged in before power on

---

## Special Case: Active vs Passive Adapters

### Understanding the Difference

**Passive DP-to-HDMI adapters:**
- Simple physical connector conversion
- No power needed
- Limited to 1080p60 or 1440p60 typically
- Audio works properly
- Cost: $5-10

**Active DP-to-HDMI adapters:**
- Contains electronics/chip for signal conversion
- Powered from DP port
- Supports 4K60Hz, 4K120Hz
- **Audio does NOT work on BC-250**
- Cost: $15-30

### Known Issues with Active Adapters

!!!warning "Audio Broken on Active Adapters"
    Active DP-HDMI adapters consistently break audio on BC-250. Video works, audio doesn't.

**Why:** Active adapters re-encode the signal. BC-250's DP audio implementation is non-standard, and active adapters can't properly decode it.

**Workarounds:**
1. Use passive adapter (best solution)
2. Use USB DAC for audio
3. Use native DP monitor
4. Use USB-C headphones

### Recommended Adapters

**Passive adapters that work:**
- Amazon Basics DP to HDMI adapter
- Cable Matters DP to HDMI adapter
- Most cheap unbranded passive adapters

**Avoid:**
- Any adapter advertising "4K120" or "8K"
- Adapters with USB power ports
- Adapters with status LEDs (usually active)

---

## IOMMU Related Display Issues

### Problem: Display Fails with Certain Adapters

**Symptoms:**
- Works with some display adapters, not others
- Random "no signal" issues
- Display works initially then fails

**Cause:**
IOMMU (AMD's virtualization feature) can interfere with display output on some adapters.

**Solution:**

Disable IOMMU in BIOS:

1. Enter BIOS setup (Del during boot)
2. Navigate to Advanced or Chipset menu
3. Find "IOMMU" or "AMD IOMMU"
4. Set to **Disabled**
5. Save and exit

Verify disabled in OS:
```bash
dmesg | grep -i iommu
# Should show "AMD-Vi: AMD IOMMU disabled"
```

!!!info "When to Enable IOMMU"
    Only enable IOMMU if you're doing GPU passthrough for virtualization. For normal use, keep it disabled.

---

## Kernel-Specific Display Issues

### Kernel 6.15+ No Display

**Problem:**
- Upgraded kernel to 6.15+
- Now no display after boot

**Cause:**
Kernel 6.15+ has driver incompatibility with BC-250.

**Solution:**

Boot older kernel:
1. At GRUB, select "Advanced options"
2. Select kernel 6.14 or earlier
3. Boot

Make older kernel default:
```bash
# Fedora:
sudo grubby --set-default /boot/vmlinuz-6.14.x

# Or remove 6.15 kernel entirely
sudo dnf remove kernel-6.15.x
```

### Kernel 6.10+ and sg_display Parameter

**Note:** Kernel 6.10+ doesn't need `amdgpu.sg_display=0` parameter

If you have this in your GRUB config and kernel ≥6.10, you can remove it:

```bash
sudo nano /etc/default/grub
# Remove amdgpu.sg_display=0
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## Hardware-Level Display Issues

### Dead Display Port

**Symptoms:**
- Never had display working
- Tried everything above
- Multiple adapters don't work

**Diagnosis:**

1. **Verify board powers on:**
   - Fan spins
   - LED lights
   - Gets warm

2. **Test on different monitor**

3. **Try all four display port lanes:**
   - Some DP ports are actually DP++ (dual mode)
   - Try different orientation/pins

**If truly dead:**

This is rare but possible. Board may have damaged DP controller.

**Options:**
- Return to seller (if recently purchased)
- Use board for headless compute
- Hardware repair (advanced)

---

## Diagnostic Commands

### Check Display Connection

```bash
# List displays detected
xrandr

# Should show:
# DisplayPort-0 connected ...

# Check DRM
ls /sys/class/drm/
# Should show card0, card0-DP-1, etc.
```

### Check GPU Initialization

```bash
# Check GPU detected
lspci | grep VGA
# Should show: AMD Radeon Graphics

# Check driver loaded
lsmod | grep amdgpu
# Should show amdgpu module

# Check for errors
dmesg | grep -i amdgpu
# Look for errors or failures
```

### Check Mesa/Driver

```bash
# Mesa version
glxinfo | grep "OpenGL version"
# Should be Mesa 25.1+

# Vulkan
vulkaninfo | grep deviceName
# Should show: AMD Radeon Graphics (RADV GFX1013)
```

---

## Recovery Procedures

### If Completely Unable to Get Display

**Step 1: Verify hardware**
- Power supply working? (8-pin firmly connected)
- Fans spinning?
- LED lights on?

**Step 2: Clear CMOS**
- Remove battery 60 seconds
- Press power button 5 times while battery out
- Replace, power on

**Step 3: Reflash BIOS**
- Create bootable USB with modded BIOS
- Flash again
- Clear CMOS again

**Step 4: Hardware programmer**
- Use CH341A to reflash BIOS directly
- Bypass potentially corrupted BIOS

[BIOS recovery guide →](../bios/recovery.md)

### If Display Works in Linux But Not Windows

!!!info "No Windows Drivers"
    BC-250 has NO Windows GPU drivers. Display will only work in BIOS and Linux. This is expected and cannot be fixed.

---

## FAQ

**Q: Display worked yesterday, now doesn't. What changed?**
A: Check if kernel updated (`uname -r`). If now on 6.15+, boot older kernel.

**Q: Can I use HDMI directly?**
A: No, board only has DisplayPort. Must use DP cable or adapter.

**Q: Will USB-C to HDMI work?**
A: No, board doesn't have USB-C DisplayPort alt mode. Use the DP port.

**Q: Why does BIOS show but Linux doesn't?**
A: BIOS uses UEFI framebuffer. Linux tries to use GPU drivers which may be missing or broken.

**Q: Display sometimes works, sometimes doesn't?**
A: Likely loose cable, bad adapter, or overheating. Check all connections and temps.

---

**Related Guides:**
- [BIOS Flashing](../bios/flashing.md)
- [Linux Installation](../linux/distributions.md)
- [Hardware Setup](../hardware/power.md)
- [BIOS Recovery](../bios/recovery.md)
