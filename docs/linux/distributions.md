# Linux Distribution Recommendations

Choosing the right Linux distribution for your BC-250 is important for a smooth experience. This guide covers tested distributions with their pros, cons, and suitability.

## Quick Recommendations

| User Type | Recommended Distro | Reason |
|-----------|-------------------|--------|
| **Beginners** | Fedora 42/43 or Bazzite | Easy setup, works out-of-box, good documentation |
| **Gaming Focus** | Bazzite | Steam Deck experience, pre-configured for gaming |
| **Performance** | CachyOS | Optimized packages, best frame times |
| **Advanced Users** | Arch Linux | Full control, latest packages |
| **Stability** | Debian/PikaOS | Rock-solid, good for production work |

## Fedora 42/43 (Most Recommended for Beginners)

### Overview

**Status:** Highly recommended, most tested
- **Desktop:** GNOME or KDE Plasma
- **Kernel:** 6.15.7-6.17.7 (recommended) or 6.12-6.14 LTS (stable)
- **Mesa:** 25.1+ in mainline repos (Fedora 43)

### Pros

- Easiest setup with automated scripts
- Mesa 25.1 now in official repositories (no COPR needed)
- Extensive community documentation
- Good power efficiency (~10W less idle vs some distros)
- Strong hardware support

### Cons

- Kernel 6.15.0-6.15.6 and 6.17.8+ break BC-250 (use 6.15.7-6.17.7 or 6.12-6.14 LTS)
- Some users report MTG Arena crashes specifically on Fedora
- Auto-updates can break things if not careful

### Setup Resources

- Automated script: [mothenjoyer69/bc250-documentation](https://github.com/mothenjoyer69/bc250-documentation)
- Governor COPR: `filippor/bazzite`
- [Detailed Fedora Setup Guide](fedora.md)

### Installation Notes

!!!warning "Use Basic Graphics Mode"
    During installation, select "Troubleshooting" → "Install in Basic Graphics Mode" to avoid black screen issues.

## Bazzite (Best for Gaming)

### Overview

**Status:** Steam Deck-like experience, works OOTB
- **Base:** Fedora Atomic (immutable)
- **Desktop:** Deck UI or Desktop Mode (GNOME/KDE)
- **Kernel:** Custom kernel with BC-250 patches included
- **Mesa:** 25.1+ out-of-box

### Pros

- Works out-of-box with latest ISO
- Includes GPU frequency patch natively (up to 2230MHz)
- Immutable system (harder to break)
- Governor installation script available
- Steam Deck UI for couch gaming
- Pre-configured for gaming

### Cons

- Immutable system harder to customize
- Some users report sleep/wake issues
- Package installation more complex (rpm-ostree)
- Updates can occasionally break things (use pinning)

### Setup

```bash
# After installation, install governor:
curl -s https://raw.githubusercontent.com/vietsman/bc250-documentation/refs/heads/main/oberon-setup.sh | sudo sh

# Pin working version after successful boot:
rpm-ostree pin 0
```

### Recovery

If an update breaks your system:

```bash
# Rollback to previous version
rpm-ostree rollback
systemctl reboot
```

## CachyOS (Best Performance)

### Overview

**Status:** Best gaming performance, requires advanced setup
- **Base:** Arch Linux with optimized repos
- **Kernel:** 6.12 LTS (cachyos-lts)
- **Scheduler:** BORE scheduler for better frame times
- **Mesa:** 25.1+ by default

### Pros

- Best overall gaming performance
- BORE scheduler improves frame latency
- Optimized packages (v3/v4 CPU instructions)
- Kernel manager GUI for easy patching
- Latest software

### Cons

- **Cannot install ISO directly** on BC-250
- Must install Arch first, then migrate to CachyOS
- Kernel 6.15.0-6.15.6 and 6.17.8+ cause panics (use 6.15.7-6.17.7 or LTS)
- More complex setup

### Installation Method

**Option 1: Arch + Migration (Recommended)**

1. Install Arch Linux following the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
2. Use `linux-lts` kernel (6.12.x - 6.14.x)
3. Boot with `nomodeset` if needed (remove after driver installation)
4. Use CachyOS migration script from their documentation
5. Install CachyOS LTS kernel during migration
6. Install kernel manager: `pacman -S cachyos-kernel-manager`
7. Select LTS kernel (6.12.x)

**Option 2: Custom ISO (Advanced)**

Build CachyOS ISO with LTS kernel:

```bash
git clone https://github.com/CachyOS/CachyOS-Live-ISO
cd CachyOS-Live-ISO
# Replace stable kernel with LTS
grep -rl 'linux-cachyos' ./ | xargs sed -i 's/linux\-cachyos/linux\-cachyos\-lts/g'
# Build ISO (follow repo instructions)
```

## Arch Linux (Maximum Control)

### Overview

**Status:** Works well, requires manual configuration
- **Desktop:** Your choice
- **Kernel:** 6.12-6.14 LTS recommended
- **Mesa:** 25.1+ from official repos

### Pros

- Rolling release (latest packages)
- Full control over system
- Excellent documentation (Arch Wiki)
- Works with bc250-arch automated script
- AUR provides extensive software library

### Cons

- Manual setup required
- More maintenance needed
- Easier to break if not careful
- Steeper learning curve

### Installation

**Option 1: Automated Script**

```bash
# Clone and run bc250-arch script
git clone https://github.com/pnbarbeito/bc250-arch
cd bc250-arch
./install.sh
```

**Option 2: Manual Installation**

Follow the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide), then:

```bash
# Install required packages after base installation:
pacman -S base-devel cmake git mesa vulkan-radeon

# Install governor (see system/governor.md)
```

## Debian / PikaOS (Stable Choice)

### Overview

**Status:** Very stable, requires some manual work
- **Kernel:** 6.12-6.14 (Xanmod recommended)
- **Mesa:** 25.1.3+ from experimental repos
- **Desktop:** GNOME or KDE Plasma

### Pros

- Extremely stable
- Fast and secure
- Low power consumption (55W idle under Plasma)
- Good for production/work use
- Mature ecosystem

### Cons

- Requires manual compilation of some components
- Fewer gaming-specific optimizations
- Mesa from experimental repositories
- Qt 6.83+ required for Plasma to work properly

### PikaOS (Debian-based Gaming Distro)

**Features:**
- Debian base with gaming focus
- Mesa 25.1+ out-of-box
- GPU frequency patch included by default
- Easier than vanilla Debian for gaming

**Setup:**
- Download and install PikaOS
- Works mostly out-of-box
- Install governor manually

## Manjaro (Easy Arch Alternative)

### Overview

**Status:** Works out-of-box
- **Base:** Arch Linux (user-friendly)
- **Desktop:** KDE Plasma recommended
- **Kernel:** 6.14+

### Pros

- Boots from USB without issues
- Latest KDE Plasma works well
- Oberon governor installs easily
- Good hardware detection
- Easier than Arch for beginners

### Cons

- Some boot issues for certain users (IOMMU-related)
- Less tested than Fedora/Bazzite
- Delayed package updates compared to Arch

### Notes

- Works well with KDE + Wayland
- Some games work better than on Fedora (e.g., MTG Arena)

## Also Works Well

### Ubuntu

**Status:** Works with updated Mesa
- **Mesa:** 25.1.5 available via PPAs
- **Kernel:** 6.12+ recommended
- **Desktop:** GNOME by default

**Pros:**
- Large user base
- Extensive documentation
- Long-term support (LTS) versions

**Cons:**
- Requires adding PPAs for latest Mesa
- Less tested on BC-250 than Fedora/Bazzite
- May need manual configuration

**Setup:**
```bash
# Add PPA for Mesa 25.1.5+
sudo add-apt-repository ppa:kisak/kisak-mesa
sudo apt update && sudo apt upgrade

# Install governor (build from source)
```

### SteamOS

**Status:** Now works - Mesa updated
- **Base:** Arch-based immutable system
- **Mesa:** Now includes 25.1+ in recent updates
- **Desktop:** KDE Plasma with Steam Deck UI

**Pros:**
- Steam Deck experience
- Gaming-optimized
- Immutable system

**Cons:**
- Less flexible than standard distros
- Valve's update schedule
- Limited testing on BC-250

**Note:** Previously not recommended due to old Mesa, but recent updates have addressed this issue.

### Kernel Compatibility

!!!danger "Avoid Broken Kernel Versions"
    Kernel 6.15.0-6.15.6 and 6.17.8+ cause kernel panics and GPU initialization failures on BC-250. **Use 6.15.7-6.17.7 for best performance or 6.12-6.14 LTS for stability.**

## Distribution Comparison Table

| Feature | Fedora | Bazzite | CachyOS | Arch | Debian |
|---------|--------|---------|---------|------|--------|
| **Ease of Setup** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| **Gaming Performance** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Stability** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Documentation** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Power Efficiency** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Customization** | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

## Desktop Environment Recommendations

### GNOME

**Works on:** Fedora, Bazzite, Arch, Debian
- Modern, clean interface
- Good Wayland support
- Resource-efficient
- Some users report issues (test carefully)

### KDE Plasma

**Works on:** Fedora, Manjaro, Arch, Debian
- Highly customizable
- Wayland support improving (Plasma 6+)
- Feature-rich
- Generally stable on BC-250
- **Most tested and recommended**

### Cinnamon

**Works on:** Fedora, Debian
- Traditional desktop
- X11-based (very stable)
- Lightweight
- Good choice for stability

## Installation Flow Comparison

### Fedora (Easiest)

1. Download Fedora 42/43 Workstation ISO
2. Flash to USB
3. Boot in "Basic Graphics Mode"
4. Install normally
5. Run setup script
6. Done

### Bazzite (Gaming-Focused)

1. Download Bazzite ISO (Desktop or Deck variant)
2. Flash to USB
3. Boot and install
4. Run governor installation script
5. Optional: Configure Deck UI
6. Done

### CachyOS (Performance)

1. Download Arch ISO
2. Install Arch following the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
3. Use `linux-lts` kernel during installation
4. Boot Arch
5. Run CachyOS migration script
6. Install CachyOS LTS kernel
7. Configure system
8. Install governor

## Common Issues by Distribution

### Fedora

- **Issue:** Kernel 6.15.0-6.15.6 and 6.17.8+ break GPU
- **Solution:** Pin kernel to 6.14

### Bazzite

- **Issue:** Updates sometimes break system
- **Solution:** Use `rpm-ostree pin 0` and rollback if needed

### CachyOS/Arch

- **Issue:** Governor doesn't auto-start on boot
- **Solution:** Run game once to activate, or check service status

### Debian

- **Issue:** Requires manual Mesa compilation
- **Solution:** Use experimental repos or wait for stable update

## Switching Distributions

If you want to try a different distribution:

1. **Back up your data** (M.2 SSD contents)
2. **Flash new distro** to USB
3. **Boot and install** (will overwrite previous install)
4. **Reconfigure BIOS** if needed
5. **Reinstall software** and restore data

!!!tip "Test Before Committing"
    Try distributions as live USB before installing to ensure compatibility with your hardware.

## See Also

- [Fedora Detailed Setup Guide](fedora.md)
- [Kernel Requirements](kernel.md)
- [Mesa Driver Installation](mesa.md)
- [Getting Started Guide](../getting-started/quick-start.md)
