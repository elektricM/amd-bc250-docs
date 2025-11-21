# Display Connectivity

Guide to connecting displays to the BC-250 and troubleshooting display issues.

## Display Output Overview

### Available Connectors

- **DisplayPort:** 1x full-size DisplayPort 1.4
- **HDMI:** None (requires adapter)
- **Resolution Support:** Up to 8K @60Hz or 4K @120Hz
- **HDR Support:** Yes (HDR10)

!!!info "DisplayPort Only"
    The BC-250 only has DisplayPort output. For HDMI displays, you'll need a DP to HDMI adapter.

## DisplayPort Direct Connection

### Recommended Setup

**DisplayPort Cable:**
- **Version:** DisplayPort 1.4 certified
- **Length:** 1-2m (longer cables can cause issues)
- **Quality:** Use certified cables (VESA DP certified)

**Resolution Support:**
- 1920x1080 (1080p) @ 144Hz+
- 2560x1440 (1440p) @ 144Hz+
- 3840x2160 (4K) @ 120Hz
- 7680x4320 (8K) @ 60Hz

!!!success "Native DisplayPort Recommended"
    If your monitor has DisplayPort, use it directly. This avoids adapter compatibility issues.

### Audio Over DisplayPort

**Status:** Works for most users

**Confirmed Working Monitors:**
- MSI 27CQ6F (direct DP connection)
- Various Dell/HP monitors
- Most modern DisplayPort monitors

**Configuration:**
```bash
# Check audio devices
aplay -l

# Select HDMI/DisplayPort audio in system settings
# Usually appears as "HDMI/DisplayPort" or "AMD/ATI"
```

**Troubleshooting Audio:**
- Some monitors don't pass through audio
- Check monitor specs for audio support
- Verify speakers are enabled on monitor
- Test with headphones/external speakers first

## DP to HDMI Adapters

### Adapter Types

**Passive Adapters (Not Recommended):**
- **Audio:** Usually works
- **Issues:** Compatibility problems common
- **Use Case:** Testing only

**Active Adapters (Recommended for 4K):**
- **Audio:** Can have issues
- **Benefits:** Better signal quality, higher resolutions
- **Use Case:** 4K displays

### Known Issues with Adapters

!!!warning "Audio Problems with Adapters"
    Many DP to HDMI adapters break audio functionality. This is a known limitation.

**Common Symptoms:**
- Display works, no audio
- Audio works intermittently
- Audio dropouts/clicking

**Workarounds:**
1. Use USB audio adapter/DAC
2. Use 3.5mm audio cable (no audio output on BC-250)
3. Use Bluetooth audio
4. Try different adapter brand

### Tested Adapter Compatibility

| Adapter Type | Display Works | Audio Works | Notes |
|--------------|---------------|-------------|-------|
| Generic Passive | Usually | Sometimes | Hit or miss |
| Cable Matters Active | Yes | No | 4K works, no audio |
| Club3D Active | Yes | Sometimes | Sporadic audio issues |
| StarTech Active | Yes | No | Reliable display, no audio |

!!!info "Audio Adapter Limitation"
    If you need audio, consider a USB audio adapter ($10-20) as a reliable solution.

## Common Display Problems

### No Display on Boot

**Symptoms:**
- Monitor shows "No Signal"
- System appears to be running (fans spin)
- Power LED on board is lit

**Causes:**
1. No GPU drivers installed
2. Incorrect kernel parameters
3. Bad cable/adapter
4. Monitor incompatibility

**Solutions:**

**Step 1: Boot with nomodeset**
```bash
# At GRUB, press 'e' to edit boot entry
# Find line starting with 'linux' or 'linuxefi'
# Add 'nomodeset' to end of line
# Press Ctrl+X to boot
```

**Step 2: Verify cable/adapter**
- Try different DisplayPort cable
- Try display on another system
- Remove adapter if using one

**Step 3: Check BIOS settings**
- Verify display output is enabled
- Try resetting BIOS to defaults

[See Display Troubleshooting Guide](../troubleshooting/display.md) for detailed steps.

### Black Screen After Login

**Symptoms:**
- GRUB menu displays
- Login screen displays
- Black screen after logging in

**Cause:** Desktop environment issue, usually Wayland

**Solutions:**

**Option 1: Switch to X11**
1. At login screen, select user
2. Click gear icon (bottom right)
3. Select "GNOME on Xorg" or "Plasma (X11)"
4. Log in

**Option 2: Disable Wayland**
```bash
# Edit GDM config
sudo nano /etc/gdm/custom.conf

# Uncomment this line:
WaylandEnable=false

# Save and reboot
```

### Display Works But Low Resolution

**Symptoms:**
- Display detected but stuck at 1024x768 or 1920x1080
- Higher resolutions not available
- Refresh rate limited to 60Hz

**Causes:**
- GPU drivers not loaded
- Using software rendering (llvmpipe)
- Bad cable limiting bandwidth

**Check Current Driver:**
```bash
glxinfo | grep "OpenGL renderer"
# Should show: AMD Radeon Graphics (RADV GFX1013)
# If shows: llvmpipe - drivers not working
```

**Solutions:**
1. Install Mesa 25.1+ drivers
2. Remove nomodeset from GRUB
3. Use certified DisplayPort cable
4. Update monitor firmware

[See Mesa Installation Guide](../linux/mesa.md)

### Flickering or Artifacts

**Symptoms:**
- Screen flickers occasionally
- Visual artifacts (lines, blocks)
- Colors incorrect

**Causes:**
- Bad cable
- Interference
- Overclocking too high
- Insufficient cooling

**Solutions:**
1. Replace DisplayPort cable
2. Reduce GPU overclock
3. Check GPU temperature
4. Try different monitor input

### HDMI 2.1 / 4K @120Hz Issues

**Limitation:** DP to HDMI adapters often limited to HDMI 2.0

**HDMI 2.0 Limits:**
- 4K @ 60Hz
- No 4K @ 120Hz
- Limited HDR

**HDMI 2.1 Requirement:**
- Requires active DP 1.4 to HDMI 2.1 adapter
- Still may have compatibility issues

!!!tip "Use Native DisplayPort"
    For high refresh rate 4K gaming, use a native DisplayPort monitor instead of adapter.

## Multiple Display Support

### Limitations

**Hardware:**
- Only 1 physical DisplayPort output
- No multi-monitor support from single board

**Options for Multiple Displays:**

**Option 1: USB DisplayLink Adapter**
- Add USB to HDMI/DisplayPort adapter
- Works for desktop use
- Not suitable for gaming (high latency)

**Option 2: DisplayPort MST Hub**
- Split single DP into multiple displays
- Shares bandwidth between displays
- Works for productivity
- Limited resolution per display

**Option 3: Multiple BC-250 Boards**
- Use separate board per monitor
- Impractical for most users

## Display Configuration

### Setting Resolution and Refresh Rate

**KDE Plasma:**
1. System Settings → Display and Monitor
2. Select your display
3. Choose resolution and refresh rate
4. Apply

**GNOME:**
1. Settings → Displays
2. Select resolution from dropdown
3. Click Apply

**Command Line (xrandr):**
```bash
# List available modes
xrandr

# Set mode
xrandr --output DisplayPort-0 --mode 1920x1080 --rate 144
```

### Custom Resolutions

Some monitors may require custom modelines:

```bash
# Generate modeline
cvt 2560 1440 144

# Add to xrandr
xrandr --newmode "2560x1440_144.00" ...
xrandr --addmode DisplayPort-0 "2560x1440_144.00"
```

### HDR Configuration

HDR support in Linux is improving but still experimental:

**Check HDR Support:**
```bash
# KDE Plasma 6+: HDR toggle in display settings
# GNOME: Limited HDR support
```

**Notes:**
- HDR support varies by desktop environment
- KDE Plasma 6+ has best HDR support
- May require Wayland session
- Game-specific HDR may not work

## Audio Solutions

Since audio over HDMI adapters is unreliable, here are alternative solutions:

### Option 1: USB Audio Adapter

**Recommended Adapters:**
- Creative Sound Blaster Play! 4
- Sabrent USB Audio Adapter
- FiiO K3 DAC (audiophile option)

**Setup:**
1. Plug USB audio adapter into BC-250 USB port
2. Connect speakers/headphones to adapter
3. Select USB audio device in system settings

### Option 2: Bluetooth Audio

**Requirements:**
- USB Bluetooth adapter
- Bluetooth speakers/headphones

**Setup:**
```bash
# Install Bluetooth tools
sudo dnf install bluez bluez-tools  # Fedora
sudo pacman -S bluez bluez-utils    # Arch

# Enable Bluetooth
sudo systemctl enable --now bluetooth

# Pair device (use GUI or bluetoothctl)
```

**Latency Warning:** Bluetooth adds ~100-200ms latency, noticeable in gaming

### Option 3: Monitor with Displayport + Speakers

If your monitor has DisplayPort input AND built-in speakers:
- Audio over DisplayPort usually works
- Check monitor supports audio input
- Enable speakers in monitor settings

## Troubleshooting Checklist

### Before Asking for Help

1. **Verify hardware:**
   - Cable is securely connected both ends
   - Monitor works with another device
   - Power LED on monitor is lit

2. **Check software:**
   ```bash
   # GPU detected?
   lspci | grep VGA

   # Driver loaded?
   lsmod | grep amdgpu

   # Mesa version?
   glxinfo | grep "OpenGL version"
   ```

3. **Test with nomodeset:**
   - If display works with nomodeset, driver issue
   - If no display with nomodeset, hardware issue

4. **Try different cable/adapter:**
   - Cables can fail
   - Adapters have compatibility issues

5. **Check logs:**
   ```bash
   # Check for errors
   dmesg | grep -i amdgpu
   journalctl -b | grep -i drm
   ```

## Display Recommendations by Use Case

### Gaming @ 1080p 144Hz
- **Display:** Any 1080p 144Hz+ DisplayPort monitor
- **Cable:** DP 1.4 certified cable
- **Expected:** Works flawlessly

### Gaming @ 1440p 144Hz
- **Display:** 1440p 144Hz+ DisplayPort monitor
- **Cable:** DP 1.4 certified, <2m length
- **Expected:** Works well

### 4K @ 60Hz
- **Display:** 4K 60Hz monitor with DisplayPort OR HDMI
- **Cable:** DP 1.4 cable OR active DP-to-HDMI adapter
- **Audio:** Use USB audio if adapter needed

### 4K @ 120Hz
- **Display:** 4K 120Hz DisplayPort monitor
- **Cable:** DP 1.4 certified cable
- **Note:** May need to manually set 120Hz in settings

### TV Connection (Living Room Gaming)
- **Display:** 4K TV with HDMI 2.0+
- **Adapter:** Active DP to HDMI 2.0 adapter
- **Audio:** Use TV speakers (if adapter supports audio) OR Bluetooth/USB audio
- **Note:** Test adapter audio before permanent setup

## See Also

- [Troubleshooting Display Issues (Detailed)](../troubleshooting/display.md)
- [Hardware Specifications](specifications.md)
- [BIOS Configuration](../bios/flashing.md)
- [Mesa Driver Installation](../linux/mesa.md)
