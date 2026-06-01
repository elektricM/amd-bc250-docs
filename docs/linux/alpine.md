# Alpine Linux Setup Guide

<img src="https://cdn.simpleicons.org/alpinelinux" alt="Alpine Linux" width="48"/>

Alpine Linux works on the BC-250, but setup is more manual than on mainstream desktop distributions. It is best suited to advanced users who want a lean server build, efficient compute-focused system, custom desktop environment, or OpenRC-based installation with minimal overhead.

**Status:** Working with manual setup  
**Difficulty:** Advanced  
**Init System:** OpenRC  
**Best For:** Minimal systems, server workloads, custom builds, and low resource usage

---

## Why Choose Alpine?

### Advantages

- Very small base system
- Low RAM and power usage (150 MB & 35 W)
- OpenRC instead of systemd
- Good fit for server or appliance-style deployments
- Easy to keep lean if you only install what you need

### Considerations

- Much less BC-250 testing than Fedora, Bazzite, Arch, or Debian
- Graphics and Vulkan setup is manual
- Governor installation is manual
- Some packages and workflows differ across Alpine branches
- Not the easiest choice for gaming-focused installs

!!! info "Best Use Case"
    Alpine is a strong choice if you want a stripped-down BC-250 system, especially for GPU/CPU power server use or custom environments where low overhead matters more than convenience.

---

## BIOS Requirements

Before installing Alpine, ensure BIOS is configured:

1. Flash modified BIOS (P3.00 or later recommended)
2. Set VRAM allocation to 512MB dynamic (recommended for shared VRAM / UMA, see [VRAM Configuration](../bios/vram.md#option-1-512mb-dynamic))

For LLM or GPU-compute workloads, consider a fixed split such as [Option 3: 8GB RAM / 8GB VRAM](../bios/vram.md#option-3-fixed-8gb-ram-8gb-vram) or [Option 4: 12GB RAM / 4GB VRAM](../bios/vram.md#option-4-fixed-12gb-ram-4gb-vram), depending on whether you want to prioritize GPU memory or system RAM.

See [BIOS Flashing Guide](../bios/flashing.md).

---

## Installation Overview

### Prerequisites

- Alpine installation media
- Ethernet connection recommended
- Passive DP-to-HDMI adapter if needed
- Comfort with manual post-install configuration

### Base Installation

1. Boot the Alpine installer
2. If the display does not initialize properly, boot once with `nomodeset`
3. Run the normal Alpine install process with `setup-alpine`
4. Install to disk as usual
5. Reboot into the installed system

!!! warning "Remove nomodeset After Setup"
    If you used `nomodeset` during installation, remove it after the graphics stack is working. Leaving it enabled will prevent normal GPU acceleration.

---

## Post-Installation Setup

### 1. Update the Base System

!!! info "sudo on Alpine"
    Some Alpine installs do not include `sudo` by default. If `sudo` is missing, either install it with `apk add sudo` or use `doas` instead. For a quick shell-level compatibility shortcut, you can temporarily run `alias sudo="doas"`.

```bash
sudo apk update
sudo apk upgrade
```

---

### 2. Install a Working Kernel

Before installing the kernel, review the known-broken BC-250 ranges in [Kernel Configuration](kernel.md).

!!! warning "Kernel Selection Still Matters"
    Avoid known-broken BC-250 kernel ranges such as 6.15.0-6.15.6 and 6.17.8-6.17.10. Prefer confirmed working releases such as 6.18.18 LTS, 6.19.x stable, or 6.17.11+ where available.

Alpine's canonical kernel package is `linux-lts`:

```bash
sudo apk add linux-lts
```

Reboot after kernel installation:

```bash
sudo reboot
```

After reboot, confirm the active kernel:

```bash
uname -a
```

If `uname -r` reports a kernel in one of the broken ranges above, install a known-good `linux-lts` build before continuing.

---

### 3. Install Firmware and Graphics Packages

Setup drivers and firmware:

```bash
sudo apk add linux-firmware-amdgpu # base amdgpu drivers
sudo apk add mesa mesa-gl mesa-dri-gallium # mesa drivers
sudo apk add mesa-vulkan-ati vulkan-loader vulkan-tools # vulkaninfo ...
sudo apk add mesa-demos # glxinfo ...
```

If you need extra Vulkan development tools later:

```bash
sudo apk add vulkan-loader-dev glslang-dev spirv-headers shaderc cmake
```

---

### 4. Configure the Bootloader and Rebuild Boot Files

If you added `nomodeset` during installation, remove it from your bootloader configuration after Mesa and Vulkan are working.

See [Environment Variables](../drivers/environment.md#mitigationsoff) for details on `mitigations=off`.

#### Option A: extlinux (Alpine default)

Most Alpine installs use `extlinux` by default. Edit:

```bash
sudo nano /etc/update-extlinux.conf
```

Use a performance-oriented default line such as:

```bash
default_kernel_opts="quiet amdgpu.sg_display=0 mitigations=off"
```

Then rebuild boot files:

```bash
sudo mkinitfs
sudo update-extlinux
sudo reboot
```

#### Option B: GRUB (optional)

If your Alpine install uses GRUB instead, make sure GRUB is already installed and configured first (`grub` plus `grub-efi` for UEFI or `grub-bios` for legacy BIOS). Then edit:

```bash
sudo nano /etc/default/grub
```

Use:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.sg_display=0 mitigations=off"
```

Then rebuild boot files:

```bash
sudo mkinitfs
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

---

### 5. Verify AMDGPU and Vulkan

After reboot, verify that firmware is present, the kernel driver loaded correctly, and the system is using the AMD GPU instead of software rendering:

```bash
lsmod | grep amdgpu
# Should show: amdgpu

dmesg | grep -i amdgpu
# Use this if you need the full amdgpu log for troubleshooting

vulkaninfo --summary
# Should list the AMD RADV Vulkan driver and GPU0
```

---

### 6. Install the GPU Governor

The BC-250 still benefits heavily from a governor on Alpine. Community testing shows a working manual setup using the SMU branch.

Install build dependencies:

```bash
sudo apk add git rust cargo libdrm-dev dbus
```

Enable D-Bus:

```bash
sudo rc-service dbus start
sudo rc-update add dbus default
```

Clone and build the governor:

```bash
git clone --branch smu https://github.com/filippor/cyan-skillfish-governor.git cyan-skillfish-governor-smu
cd cyan-skillfish-governor-smu
cargo build --release
```

Install the binary and config:

```bash
sudo install -Dm755 target/release/cyan-skillfish-governor-smu /usr/local/bin/cyan-skillfish-governor-smu
sudo mkdir -p /etc/cyan-skillfish-governor-smu
sudo cp default-config.toml /etc/cyan-skillfish-governor-smu/config.toml
```

Install the required D-Bus policy before the first test run:

```bash
sudo tee /etc/dbus-1/system.d/com.cyan.skillfishgovernor.conf > /dev/null << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
"http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
<policy user="root">
<allow own="com.cyan.SkillFishGovernor"/>
<allow send_destination="com.cyan.SkillFishGovernor"/>
</policy>
<policy context="default">
<allow send_destination="com.cyan.SkillFishGovernor"/>
</policy>
</busconfig>
EOF

sudo rc-service dbus restart
```

Test it manually first:

```bash
sudo cyan-skillfish-governor-smu --verbose /etc/cyan-skillfish-governor-smu/config.toml
```

---

### 7. Create an OpenRC Service for the Governor

Create `/etc/init.d/cyan-skillfish-governor-smu`:

```bash
#!/sbin/openrc-run
name="cyan-skillfish-governor-smu"
description="GPU governor for AMD Cyan Skillfish APU"
depend() {
    need dbus
}
command="/usr/local/bin/cyan-skillfish-governor-smu"
command_args="/etc/cyan-skillfish-governor-smu/config.toml"
command_background=true
pidfile="/run/${RC_SVCNAME}.pid"
output_log="/var/log/cyan-skillfish-governor-smu.log"
error_log="/var/log/cyan-skillfish-governor-smu.log"
```

Then enable it:

```bash
sudo chmod +x /etc/init.d/cyan-skillfish-governor-smu
sudo rc-update add cyan-skillfish-governor-smu default
sudo rc-service cyan-skillfish-governor-smu start
sudo rc-service cyan-skillfish-governor-smu status
```

If you edit the config later:

```bash
sudo rc-service cyan-skillfish-governor-smu restart
```

---

## Troubleshooting

### Governor Does Not Start
**Symptoms:**  
- Bad GPU performance  
- On start or restart, `start-stop-daemon` reports `no matching processes found`  

**Solution:**  
- `dbus` is installed and enabled  
- The D-Bus policy file at `/etc/dbus-1/system.d/com.cyan.skillfishgovernor.conf` is present  
- `libdrm-dev` was present during build  
- The binary is installed in `/usr/local/bin/`  
- The config file exists at `/etc/cyan-skillfish-governor-smu/config.toml`  

```bash
sudo rc-service cyan-skillfish-governor-smu status
sudo /usr/local/bin/cyan-skillfish-governor-smu --verbose /etc/cyan-skillfish-governor-smu/config.toml # to debug
```

### D-Bus AccessDenied Error
**Symptoms:**  
- Running `sudo /usr/local/bin/cyan-skillfish-governor-smu --verbose /etc/cyan-skillfish-governor-smu/config.toml` returns:  
  `org.freedesktop.DBus.Error.AccessDenied: Connection ":1.0" is not allowed to own the service "com.cyan.SkillFishGovernor"`  

### Invalid Voltage/Frequency Curve
**Symptoms:**  
- The governor starts, but clocks or voltage behavior is unstable  
- Performance tuning changes do not apply correctly  

**Solution:**  
Keep the voltage curve monotonic: higher frequencies should not use less mV.

---

## Community Resources

- **Alpine Linux:** [alpinelinux.org](https://alpinelinux.org/)
- **Alpine Wiki:** [wiki.alpinelinux.org](https://wiki.alpinelinux.org/)
- **GPU Governor SMU Branch:** [cyan-skillfish-governor-smu](https://github.com/filippor/cyan-skillfish-governor/tree/smu)

---

## Related Guides

- [Debian Setup](debian.md)
- [Arch Linux Setup](arch.md)
- [Fedora Setup](fedora.md)
- [GPU Governor](../system/governor.md)
