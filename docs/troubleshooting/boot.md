# Boot Troubleshooting Guide

Complete guide to diagnosing and fixing boot issues on the BC-250. This covers everything from black screens during installation to kernel panics and bootloader problems.

---

## Quick Diagnosis Flowchart

**Start here to identify your issue:**

1. **Does the board power on?** (Fan spins, LED lights up)
   - **NO** → See [Hardware Issues](#hardware-power-issues)
   - **YES** → Continue to #2

2. **Can you see BIOS menu?** (Press Del during boot)
   - **NO** → See [Hardware Issues](#hardware-power-issues)
   - **YES** → Continue to #3

3. **What happens when you try to boot?**
   - **Black screen during installer** → [No Display During Installation](#no-display-during-installation)
   - **Black screen after installation** → [No Display After Installation](#no-display-after-installation)
   - **Boots to black screen, no OS** → [Black Screen After GRUB](#black-screen-after-grub)
   - **System hangs/freezes during boot** → [Boot Hangs](#boot-hangs-or-freezes)
   - **Kernel panic or error messages** → [Kernel Panics](#kernel-panic-on-boot)
   - **GRUB not appearing** → [GRUB/Bootloader Issues](#grubbootloader-problems)

---

## No Display During Installation

### Problem: Black Screen When Booting Installer

**Symptoms:**
- USB boots, but screen goes black
- No installer appears
- Monitor shows "No Signal"
- Works in BIOS but not in installer

**Cause:**
Installer doesn't have BC-250 GPU drivers. The Linux framebuffer (KMS) attempts to initialize the GPU and fails, resulting in no display.

**Solution: Use nomodeset Parameter**

!!!tip "Fedora: Use Basic Graphics Mode"
    In Fedora installer boot menu:

    1. Select **"Troubleshooting"**
    2. Choose **"Install in Basic Graphics Mode"**

    This enables `nomodeset` automatically.

#### Manual Method (Any Distro)

1. At GRUB boot menu, press **e** to edit boot entry
2. Find the line starting with `linux` or `linuxefi`
3. Go to the end of that line
4. Add a space, then type: `nomodeset`
5. Press **Ctrl+X** or **F10** to boot

**Example:**
```bash
# Before:
linux /vmlinuz root=live:CDLABEL=Fedora quiet

# After:
linux /vmlinuz root=live:CDLABEL=Fedora quiet nomodeset
```

#### What nomodeset Does

- **Disables kernel mode setting (KMS)** - Prevents kernel from initializing GPU
- **Forces fallback to VESA/UEFI framebuffer** - Uses basic display mode
- **Allows display without GPU drivers** - Works with any GPU
- **Enables installation to proceed** - You can complete the install

!!!warning "MUST Remove After Driver Installation"
    Once Mesa drivers are installed, `nomodeset` MUST be removed or GPU acceleration won't work. See [Removing nomodeset](#removing-nomodeset-after-driver-installation).

---

## No Display After Installation

### Problem: Installation Complete But No Display on Boot

**Symptoms:**
- Installation completed successfully
- System boots (fan spins, LED lights up, disk activity)
- Display shows "No Signal" or stays black
- Can't reach login screen

**Cause:**
Same issue as installer - GPU drivers not yet installed. The system attempts to initialize graphics and fails.

**Solution Options:**

#### Option 1: Boot with nomodeset (Recommended)

1. Power on and **immediately** watch for GRUB menu (appears for 5 seconds)
2. When GRUB appears, press **e** to edit
3. Find the line with `linux` (usually starts with `linux /boot/vmlinuz`)
4. Add `nomodeset` to the end of that line
5. Press **Ctrl+X** to boot

If GRUB doesn't appear:
- Try holding **Shift** (BIOS) or **Esc** (UEFI) during boot
- Or tap the key repeatedly right after BIOS screen

6. Once booted, install drivers (see distribution guides)
7. Remove `nomodeset` permanently (see below)

#### Option 2: Recovery Mode

Some distributions offer recovery/safe mode with basic graphics:

1. Select "Advanced options" in GRUB
2. Choose kernel with "(recovery mode)" suffix
3. Select "Resume normal boot" with networking
4. Install drivers once booted
5. Reboot normally

#### Option 3: Reinstall with Drivers Pre-Downloaded

1. Boot installer again with `nomodeset`
2. Install OS
3. Before reboot, chroot into installed system
4. Install Mesa 25.1+ while still in installer
5. Reboot without `nomodeset`

---

## Black Screen After GRUB

### Problem: GRUB Shows, Select Entry, Then Black Screen

**Symptoms:**
- GRUB menu appears and works
- Select a boot entry
- Screen goes black and nothing happens
- System seems frozen

**Cause:**
Kernel is loading but can't initialize display due to missing/incompatible drivers.

**Immediate Fix:**

1. Reboot and get to GRUB menu
2. Highlight your boot entry
3. Press **e** to edit
4. Add `nomodeset` to kernel line
5. Press **Ctrl+X** to boot

**Long-term Fix:**

After booting with nomodeset:

```bash
# Check if drivers installed
glxinfo | grep "OpenGL version"
# Should show Mesa 25.1+

# If not installed, install drivers first
# (See distribution-specific guides)

# Then verify kernel version
uname -r
# Should be 6.12.x - 6.14.x
# AVOID 6.15+
```

If kernel is 6.15+:
```bash
# Fedora - install older kernel
sudo dnf install kernel-6.14.*

# List available kernels
sudo grubby --info=ALL

# Set 6.14 as default
sudo grubby --set-default /boot/vmlinuz-6.14.*
```

!!!danger "Kernel 6.15+ Breaks BC-250 GPU"
    Kernel versions 6.15 and higher have driver incompatibility with Cyan Skillfish GPUs. Always use 6.12.x - 6.14.x LTS kernels.

---

## Boot Hangs or Freezes

### Problem: System Starts Booting But Hangs

**Symptoms:**
- Boot process starts (text scrolling or logo)
- Stops at specific point
- No error message, just frozen
- Doesn't reach login screen

**Common Causes & Solutions:**

#### 1. IOMMU Conflicts

**Diagnosis:**
System hangs after "AMD-Vi" messages in boot log.

**Solution:**
Disable IOMMU in BIOS:

1. Reboot and press **Del** to enter BIOS
2. Navigate to **Advanced** or **Chipset Configuration**
3. Find **IOMMU** or **AMD IOMMU**
4. Set to **Disabled**
5. Save and exit (F10)

Verify after boot:
```bash
dmesg | grep -i iommu
# Should show: "AMD-Vi: AMD IOMMU disabled"
```

!!!info "When IOMMU is Needed"
    Only enable IOMMU if doing GPU passthrough for virtualization. For normal desktop/gaming use, keep disabled.

#### 2. VRAM Allocation Not Applied

**Diagnosis:**
You set 512MB dynamic VRAM in BIOS but settings didn't stick.

**Cause:**
BIOS settings don't persist after flashing unless CMOS is cleared.

**Solution:**
Clear CMOS:

**Method A: Remove Battery**
1. Power off completely
2. Unplug power cable
3. Remove CR2032 CMOS battery
4. Wait 60 seconds (or press power button 5 times)
5. Replace battery
6. Power on, enter BIOS
7. Reconfigure VRAM allocation
8. Save and exit

**Method B: CMOS Jumper**
1. Power off and unplug
2. Locate CLR_CMOS jumper on board
3. Move jumper to clear position (see board pinout)
4. Wait 10 seconds
5. Return jumper to normal position
6. Power on and reconfigure BIOS

!!!success "This Fixes Most 'Bricked After Flash' Issues"
    If your board seems dead after BIOS flash, 90% chance clearing CMOS fixes it.

#### 3. Insufficient RAM (ZRAM Conflict)

**Diagnosis:**
System hangs when loading desktop or applications.

**Cause:**
ZRAM (compressed swap) conflicting with 512MB dynamic VRAM allocation.

**Solution:**
Disable ZRAM:

```bash
# Temporarily disable
sudo swapoff /dev/zram0

# Permanently disable
sudo systemctl disable systemd-zram-setup@zram0.service

# Or reduce ZRAM size
# Edit /etc/systemd/zram-generator.conf
[zram0]
zram-size = ram / 4  # Instead of ram / 2
```

**Alternative:**
Switch from 512MB dynamic to fixed allocation (10GB RAM / 6GB VRAM) in BIOS.

#### 4. Boot Timeout Issues

**Diagnosis:**
System hangs for 90+ seconds then continues.

**Possible causes:**
- Waiting for network timeout
- systemd service failing
- Missing swap partition

**Solution:**
Check boot logs:
```bash
# After successful boot
journalctl -b | grep -i "time"
journalctl -b | grep -i "fail"

# Look for timed out services
systemctl --failed
```

Disable problematic services:
```bash
sudo systemctl disable <service-name>
```

---

## Kernel Panic on Boot

### Problem: Kernel Panic Error Message

**Symptoms:**
- Boot process starts
- Kernel panic message appears
- Often mentions "unable to mount root"
- System halts completely

**Common Causes:**

#### 1. Wrong Kernel Version

**Error messages:**
- "Kernel panic - not syncing: VFS: Unable to mount root fs"
- "amdgpu initialization failed"
- GPU-related crash messages

**Solution:**
Boot older kernel:

1. At GRUB menu, select **"Advanced options"**
2. Choose older kernel (6.12 or 6.13)
3. If system boots successfully, set as default:

```bash
# List all installed kernels
sudo grubby --info=ALL

# Set older kernel as default
sudo grubby --set-default /boot/vmlinuz-6.13.*

# Or remove problematic kernel
sudo dnf remove kernel-6.15.*  # Fedora
sudo apt remove linux-image-6.15.*  # Debian/Ubuntu
```

#### 2. Corrupted initramfs

**Error messages:**
- "Failed to execute /init"
- "Unable to find root device"
- "No init found"

**Solution:**
Regenerate initramfs:

1. Boot from USB installer
2. Chroot into installed system:

```bash
# Mount root partition
sudo mount /dev/nvme0n1p2 /mnt  # Adjust device name
sudo mount /dev/nvme0n1p1 /mnt/boot/efi  # If separate boot

# Chroot
sudo arch-chroot /mnt  # Arch-based
# Or on Debian/Ubuntu:
for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt$i; done
sudo chroot /mnt

# Regenerate initramfs
# Fedora/RHEL:
sudo dracut -f

# Debian/Ubuntu:
sudo update-initramfs -u -k all

# Arch:
sudo mkinitcpio -P

# Exit chroot and reboot
exit
sudo umount -R /mnt
sudo reboot
```

#### 3. Missing Drivers in initramfs

**Solution:**
Ensure amdgpu driver in initramfs:

```bash
# Fedora/RHEL - edit /etc/dracut.conf.d/amdgpu.conf
add_drivers+=" amdgpu "

# Debian/Ubuntu - edit /etc/initramfs-tools/modules
amdgpu

# Arch - edit /etc/mkinitcpio.conf
MODULES=(amdgpu)

# Regenerate initramfs (commands above)
```

---

## GRUB/Bootloader Problems

### GRUB Not Appearing

**Symptoms:**
- System boots directly to black screen
- No boot menu shown
- Can't select options or edit entries

**Solution:**

#### Show GRUB Menu

Edit GRUB configuration to always show menu:

```bash
# Boot from live USB with nomodeset
# Mount system partition and chroot (see above)

sudo nano /etc/default/grub

# Find and change:
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=0

# To:
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=10

# Update GRUB
# Fedora:
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Debian/Ubuntu:
sudo update-grub

# Arch:
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

#### Force GRUB to Show

Temporary methods:
- Hold **Shift** during boot (BIOS systems)
- Tap **Esc** repeatedly during boot (UEFI systems)
- Interrupt boot by power cycling during GRUB load

### GRUB Command Line or Rescue Mode

**Symptoms:**
- Boot drops to `grub>` prompt
- Or `grub rescue>` prompt
- No boot entries shown

**Cause:**
GRUB can't find its configuration or boot files.

**Solution from grub> prompt:**

```bash
# List partitions
grub> ls
(hd0) (hd0,gpt1) (hd0,gpt2)

# Find root partition (try each)
grub> ls (hd0,gpt2)/boot
# If you see vmlinuz files, that's your boot partition

# Set root
grub> set root=(hd0,gpt2)
grub> set prefix=(hd0,gpt2)/boot/grub

# Load config
grub> insmod normal
grub> normal
```

**Solution from grub rescue> prompt:**

```bash
grub rescue> set prefix=(hd0,gpt2)/boot/grub
grub rescue> set root=(hd0,gpt2)
grub rescue> insmod normal
grub rescue> normal
grub rescue> boot
```

After successful boot, reinstall GRUB:

```bash
# Fedora:
sudo grub2-install /dev/nvme0n1
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Debian/Ubuntu:
sudo grub-install /dev/nvme0n1
sudo update-grub

# Arch:
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### systemd-boot Issues

For distributions using systemd-boot instead of GRUB:

**Problem:**
No boot entries or entries don't work.

**Solution:**

```bash
# Boot from live USB
sudo mount /dev/nvme0n1p2 /mnt
sudo mount /dev/nvme0n1p1 /mnt/boot

# Reinstall systemd-boot
bootctl --path=/mnt/boot install

# Regenerate entries
sudo arch-chroot /mnt  # Or appropriate chroot command
bootctl update
exit

# Reboot
```

---

## Removing nomodeset After Driver Installation

Once drivers are installed, you MUST remove `nomodeset` for GPU acceleration to work.

### Permanent Removal

**Fedora/RHEL:**
```bash
# Edit GRUB config
sudo nano /etc/default/grub

# Find line:
GRUB_CMDLINE_LINUX_DEFAULT="quiet nomodeset"

# Remove 'nomodeset':
GRUB_CMDLINE_LINUX_DEFAULT="quiet"

# Update GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Reboot
sudo reboot
```

**Debian/Ubuntu:**
```bash
sudo nano /etc/default/grub

# Remove 'nomodeset' from:
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"

# Update
sudo update-grub
sudo reboot
```

**Arch:**
```bash
sudo nano /etc/default/grub

# Remove 'nomodeset'

# Update
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

**systemd-boot:**
```bash
sudo nano /boot/loader/entries/arch.conf

# Remove 'nomodeset' from options line
options root=UUID=xxx rw nomodeset  # Before
options root=UUID=xxx rw            # After

# Save and reboot
```

### Verify Removal

After reboot:
```bash
# Check kernel command line
cat /proc/cmdline
# Should NOT contain 'nomodeset'

# Check GPU acceleration working
glxinfo | grep "direct rendering"
# Should show: "direct rendering: Yes"

# Check Vulkan
vulkaninfo --summary | grep deviceName
# Should show: AMD Radeon Graphics (RADV GFX1013)
```

---

## BIOS Settings That Affect Boot

Certain BIOS settings can cause boot problems:

### IOMMU (AMD-Vi)

**Recommendation:** **DISABLED** for normal use

**Location:** Advanced → AMD CBS → NBIO → IOMMU

**Why disable:**
- Causes display adapter conflicts
- Boot hangs with certain hardware
- Not needed unless doing GPU passthrough

### VRAM Allocation

**Recommendation:** **512MB (Dynamic)** for most users

**Location:** Advanced → Chipset → UMA Frame Buffer Size

**Important:**
- Settings don't stick without CMOS clear after flash
- Fixed allocations (4GB, 6GB, 8GB) more stable for some use cases
- Dynamic 512MB can conflict with ZRAM

### Secure Boot

**Recommendation:** **DISABLED**

**Why:**
- Custom kernels won't boot with Secure Boot
- Patched kernels (frequency range mod) need it off
- Most Linux distros work better with it disabled

### CSM (Compatibility Support Module)

**Recommendation:** **DISABLED** (pure UEFI mode)

**Why:**
- Modern Linux distros prefer UEFI
- CSM can cause boot order issues
- DisplayPort initialization better in UEFI mode

---

## Hardware Power Issues

### Board Powers On But Won't Boot

**Symptoms:**
- Fan spins
- LED lights up
- No display, no boot activity
- No BIOS access

**Checks:**

#### 1. Verify 8-Pin Power Connected
- Must be firmly seated
- Check for bent pins
- Ensure PSU rail providing 12V
- Test with multimeter if possible

#### 2. Minimum Power Requirements
- 300W PSU minimum
- 400W+ recommended for stability
- Dell 220W bricks NOT sufficient under load
- Use ATX PSU or server PSU (HP/Dell 750W+)

#### 3. CMOS Battery
- Try booting WITH battery installed
- Some boards won't POST without CMOS battery
- Use fresh CR2032

#### 4. Short Circuit Check
- Inspect board for metal debris
- Check mounting standoffs not shorting traces
- Look for bent capacitors touching heatsink

---

## Diagnostic Commands

### Check Boot Process

```bash
# View current boot
journalctl -b

# Previous boot (if current boot fails)
journalctl -b -1

# Kernel messages
dmesg | less

# Boot time analysis
systemd-analyze blame

# Critical errors only
journalctl -p 3 -b
```

### Check GPU Initialization

```bash
# GPU detected?
lspci | grep VGA
# Should show: AMD/ATI Device [1002:13fe]

# Driver loaded?
lsmod | grep amdgpu

# GPU initialization errors
dmesg | grep -i amdgpu | grep -i error

# Firmware loading
dmesg | grep -i firmware
```

### Check Display Connection

```bash
# Displays detected
xrandr --listproviders
# Should show: Provider 0: AMD Radeon Graphics

# DRM devices
ls -la /sys/class/drm/
# Should show card0, card0-DP-1

# Current display mode
xrandr
```

### Check Kernel Command Line

```bash
# Current boot parameters
cat /proc/cmdline

# Should show parameters like:
# BOOT_IMAGE=/vmlinuz root=UUID=xxx rw quiet

# Should NOT contain 'nomodeset' after drivers installed
```

---

## Recovery Procedures

### Complete Boot Failure Recovery

If system completely fails to boot after changes:

**Step 1: Boot Live USB**
- Use same distro as installed
- Boot with `nomodeset` if needed

**Step 2: Chroot Into System**
```bash
# Mount root partition
sudo mount /dev/nvme0n1p2 /mnt

# Mount boot if separate partition
sudo mount /dev/nvme0n1p1 /mnt/boot

# Mount system directories
for i in /dev /dev/pts /proc /sys /run; do
    sudo mount -B $i /mnt$i
done

# Chroot
sudo chroot /mnt
```

**Step 3: Fix Configuration**
```bash
# Restore GRUB defaults
sudo nano /etc/default/grub
# Add nomodeset back temporarily

# Regenerate config
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Or reinstall GRUB entirely
sudo grub2-install /dev/nvme0n1
```

**Step 4: Install/Reinstall Drivers**
```bash
# Install Mesa 25.1+ (see distro guides)
# Regenerate initramfs
# Exit chroot and reboot
```

### BIOS Recovery

If BIOS appears corrupted:

**Method 1: Reflash via USB**
1. Create bootable USB with BIOS files
2. Boot to USB (should happen automatically)
3. Wait for flash process (LED blinks)
4. Power off when complete
5. Clear CMOS
6. Boot and reconfigure

**Method 2: Hardware Programmer**
If USB method fails:

1. Get CH341A programmer (~$5 on AliExpress)
2. Locate SPI flash chip on board (W25Q64)
3. Use programmer to flash known-good BIOS
4. See BIOS recovery guide for detailed steps

[BIOS Recovery Guide →](../bios/recovery.md)

---

## Common Boot Issues Summary

| Symptom | Most Likely Cause | Quick Fix |
|---------|-------------------|-----------|
| Black screen during install | No drivers | Add `nomodeset` |
| Black screen after install | Drivers not installed | Boot with `nomodeset`, install Mesa |
| Boots but no GRUB | GRUB timeout 0 | Hold Shift during boot |
| Kernel panic on boot | Wrong kernel version | Boot 6.12-6.14 kernel |
| Hangs at boot | IOMMU enabled | Disable IOMMU in BIOS |
| GRUB shows, then black | Missing/wrong drivers | Add `nomodeset`, install Mesa 25.1+ |
| Works after BIOS flash, then fails | CMOS not cleared | Clear CMOS battery |
| Works, then breaks after update | Kernel 6.15+ installed | Downgrade to 6.14 |

---

## Advanced: Boot with Custom Parameters

For testing or troubleshooting, add these parameters at GRUB:

```bash
# Basic display
nomodeset

# Disable AMD GPU completely (use CPU graphics)
modprobe.blacklist=amdgpu

# More verbose boot messages
quiet → remove this
loglevel=7

# Single user mode (root shell)
systemd.unit=rescue.target

# Emergency mode (minimal system)
systemd.unit=emergency.target

# Disable specific hardware
amdgpu.sg_display=0    # Scatter-gather display (pre-6.10)
iommu=off              # Disable IOMMU
acpi=off               # Disable ACPI (not recommended)

# Debug specific subsystem
amdgpu.debug=0xffff    # AMD GPU debug output
drm.debug=0x1f         # DRM debug output
```

---

## FAQ

**Q: I can access BIOS but OS won't boot at all. What do?**
A: Add `nomodeset` to kernel parameters at GRUB. This is the #1 solution for BC-250 boot issues.

**Q: System worked yesterday, today won't boot. Nothing changed.**
A: Check if system updated kernel. Run `uname -r` from live USB after mounting system drive. If 6.15+, that's the problem.

**Q: Installed drivers but still have black screen?**
A: Did you remove `nomodeset` after installing drivers? It must be removed from GRUB config.

**Q: How do I know if my kernel is the problem?**
A: BC-250 requires kernels 6.12.x - 6.14.x. Kernels 6.15+ break GPU drivers. Avoid 6.10 and below (too old).

**Q: BIOS seems dead after flashing. Bricked?**
A: 90% chance you just need to clear CMOS. Remove battery for 60 seconds. Almost never truly bricked.

**Q: Can I prevent these boot issues?**
A: Yes! Don't update to kernel 6.15+, keep Mesa 25.1+, disable IOMMU in BIOS, always clear CMOS after BIOS flash.

---

**Related Guides:**
- [Display Issues](display.md) - Display-specific troubleshooting
- [Linux Installation](../linux/distributions.md) - Distribution setup guides
- [BIOS Flashing](../bios/flashing.md) - How to flash custom BIOS
- [Mesa Installation](../linux/mesa.md) - Installing Mesa drivers
- [BIOS Recovery](../bios/recovery.md) - Unbricking procedures
