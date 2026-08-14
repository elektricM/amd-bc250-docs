# Display Connectivity

Guide to connecting displays to the BC-250 and troubleshooting display issues.

## Display Output Overview

### Available Connectors

- **DisplayPort:** 1x full-size DisplayPort 1.4
- **HDMI:** None (requires adapter)
- **Resolution Support:** Up to 4K @120Hz
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

**Passive Adapters (Recommended):**
- **Audio:** Usually works reliably
- **Cost:** Inexpensive ($5-10)
- **Use Case:** Most users, TV connections
- **Note:** DP 1.2 max (1440p @60Hz, some support 1440p @165Hz)

**Active Adapters:**
- **Audio:** Silent out of the box on many setups. Caused by the board's DisplayPort audio clock, not the adapter, and works with the [DP audio clock fix](../troubleshooting/audio.md)
- **Cost:** More expensive ($15-30)
- **Use Case:** 4K @60Hz+ on HDMI displays, HDMI-CEC
- **Issues:** Apply the audio fix before writing the adapter off

### Known Issues with Adapters

!!!info "The adapters were never the problem"
    The "adapters break audio" reports trace back to a bug on the board itself: the firmware programs the DisplayPort audio clock for a reference clock the hardware does not have. Active adapters are real DisplayPort sinks, so they receive that off-spec audio stream and often refuse to lock (silence). Passive adapters make the driver use a different, unaffected clock path, which is why they seem immune. See [DisplayPort Audio: Silence, Desync or Slow Pitch](../troubleshooting/audio.md) for the mechanism and a fix that needs no kernel build.

**Common Symptoms:**
- Display works, no audio (active adapters: the sink refuses the off-spec stream)
- Audio plays but slow, out of sync, or with periodic crackle (sinks that tolerate it)
- Audio dropouts/clicking

**Workarounds:**
1. Apply the [DP audio clock fix](../troubleshooting/audio.md), which fixes audio through the adapter you already have
2. Use a passive adapter (unaffected clock path, but 1080p/1440p limits and no CEC)
3. Use USB audio adapter/DAC
4. Use Bluetooth audio

### Tested Adapter Compatibility

| Adapter Type | Display Works | Audio Works | Notes |
|--------------|---------------|-------------|-------|
| UGREEN 8K Active (Realtek RTD2173) | Yes | Yes, with the [DP audio clock fix](../troubleshooting/audio.md) | Silent without the fix. Verified at 4K60 and 1080p120; HDMI-CEC works. Tested by @Weijtmans |
| Generic Passive | Usually | Sometimes | Hit or miss |
| Cable Matters Active | Yes | No | 4K works, no audio |
| Club3D Active | Yes | Sometimes | Sporadic audio issues |
| StarTech Active | Yes | No | Reliable display, no audio |

!!!note "The 'No audio' rows predate the root cause discovery"
    The Cable Matters/Club3D/StarTech results were collected before the [DP audio clock bug](../troubleshooting/audio.md) was identified. The mechanism predicts they fail for the same reason and would work with the fix, but that has not been re-tested. If you own one, re-test with the fix applied and report back.

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

## HDMI-CEC Through Active Adapters

The BC-250 can join an HDMI-CEC bus (TV power on/off, input switching, TV-remote control) even though it has no HDMI port: a CEC-capable **active** DP-to-HDMI adapter tunnels CEC over the DisplayPort AUX channel, and the mainline kernel picks it up (`CONFIG_DRM_DISPLAY_DP_AUX_CEC`). No extra hardware, no Pulse-Eight dongle.

**Verified working:** UGREEN 8K active DP-HDMI adapter (Realtek RTD2173). `/dev/cec0` appears, the board registers as a CEC 2.0 Playback Device (logical address 8), and traffic is bidirectional: the TV answers a power-status query in 87 ms and an OSD-name query in 29 ms. Tested by @Weijtmans on Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29, Samsung TV.

!!!note "The kernel's CEC adapter list is not a compatibility matrix"
    The kernel documentation names a few known CEC-tunnelling chipsets (Parade PS175/PS176/PS186, MegaChips 2900, some Club3D models). That list is one maintainer's observations; the RTD2173 is not on it and works fine. Don't rule an adapter out because its chipset isn't listed; check for `/dev/cec0`.

**Passive adapters physically cannot do CEC.** CEC tunnelling lives in a DPCD register block (0x3000) that only a real DP sink implements. You can verify your adapter's support directly:

```bash
sudo dd if=/dev/drm_dp_aux0 bs=1 skip=$((0x3000)) count=1 2>/dev/null | xxd
# Output = CEC-capable (DPCD CEC block present)
# No output = no CEC tunnelling (all passive adapters, many active ones)
```

**Quick start** (`cec-ctl` is in `v4l-utils`):

```bash
ls /dev/cec*
# Should show: /dev/cec0

cec-ctl -d /dev/cec0 --playback   # register as a Playback device on the bus
cec-ctl -d /dev/cec0 -S           # scan: shows the TV, audio system, other devices
```

`/dev/cec0` is owned `root:video`, so add your user to the `video` group for non-root use, and remember group changes only apply to new logins.

!!!warning "On Bazzite and other ostree distros, usermod -aG silently does nothing"
    On an ostree system with nss-altfiles, the `video` group often exists only in the read-only `/usr/lib/group`, with no line in `/etc/group`. `usermod -aG video $USER` then validates the group through NSS, edits `/etc/group`, finds no line to append to, and exits 0 without changing anything. Seed the group line first, then add the member:

    ```bash
    grep '^video:' /usr/lib/group | sudo tee -a /etc/group
    sudo gpasswd -a $USER video
    ```

    Tested by: @Weijtmans. BC-250, Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29.

### Naming the Box on the CEC Bus

On the kernel-CEC path a device announces itself with `cec-ctl`'s default OSD name unless `--osd-name` is passed, so the TV's source list shows the box as "Playback". Two Bazzite pitfalls stack on top of that:

- Setting `CEC_OSD_NAME` in `/etc/default/cec-control` is a no-op here. `cec-control` builds `--osd-name` into its argument list but only uses that list on its libcec code path; on the `/dev/cec0` path, the one the BC-250 takes, it runs a bare `cec-ctl -d "$CEC_DEVICE" --playback`.
- `cec-onboot.service` re-runs that bare registration on every boot, clobbering any name set by hand.

Set the name directly (CEC limits OSD names to 14 characters), and reapply it after `cec-onboot` if you want it to survive reboots:

```bash
cec-ctl -d /dev/cec0 --playback --osd-name "Steam Machine"
```

[bc250-cec-identity.sh](https://github.com/Weijtmans/bc250-tools) does this and can install a small unit that makes the name persistent; [bc250-cec-check.sh](https://github.com/Weijtmans/bc250-tools) from the same repo proves whether the TV actually answers on the bus.

Renaming changes the name, not the device category: CEC has no game-console device type, and amdgpu implements no ALLM, so a TV will not switch into Game Mode by itself. Label the input as a game console in the TV's own source menu for that.

Tested by: @Weijtmans. BC-250, Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29, Samsung TV.

## Multiple Display Support

### Limitations

**Hardware:**
- Only 1 physical DisplayPort output
- No multi-monitor support from single board

**Options for Multiple Displays:**

**Option 1: USB DisplayLink Adapter**
- Add USB to HDMI/DisplayPort adapter
- Works for desktop use (plug in after boot for best results)
- **Not suitable for gaming** — high latency due to CPU-based compression, BC-250's CPU is the bottleneck
- Does not work in Steam Deck game mode

**Option 2: DisplayPort MST Hub**
- Split single DP into multiple displays
- **Maximum 2 screens** via MST on BC-250
- Shares bandwidth between displays
- Works for productivity
- Limited resolution per display

### Tested MST Hubs Compatibility  
| Adapter | Display-out | DP Version | Display Works | Audio Works | Notes |
|------------------|-----------|---------|----------|-------|------------|
| StarTech MST14DP122DP | DP (2) | 1.4 | Yes | Yes | Worked consistently with different monitors and DP cables |
| Monoprice 21972 | DP (2) | 1.2 | Mirror only | Yes | Was only able to get displays to mirror |
| ENBUER | DP (2) | 1.2? | Mirror only | Yes | Was only able to get displays to mirror |
| Generic | HDMI (2) | N/A | No | No | No audio or video output |

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

Audio through DP-HDMI adapters is fixable in software, see the [DP audio clock fix](../troubleshooting/audio.md). If you'd rather not run that, here are alternative solutions:

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
- **Audio:** TV speakers / soundbar over the adapter works with the [DP audio clock fix](../troubleshooting/audio.md); Bluetooth/USB audio as fallback
- **Note:** Test adapter audio before permanent setup

## See Also

- [Troubleshooting Display Issues (Detailed)](../troubleshooting/display.md)
- [Hardware Specifications](specifications.md)
- [BIOS Configuration](../bios/flashing.md)
- [Mesa Driver Installation](../linux/mesa.md)
