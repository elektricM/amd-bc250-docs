# Prerequisites

Before you start, make sure you have everything you need. Missing a critical component will block your progress.

## Essential Hardware

### The Board

**AMD BC-250 Board**

- Where to buy: AliExpress, eBay
- What to look for: Any BIOS version is fine (P2.00, P3.00, P4.00, P5.00)
- Avoid: Boards without heatsink or obvious physical damage

### Power Supply

**12V PSU with PCIe 8-pin connector**

Minimum requirements:

- **Wattage:** 300W+ recommended (250W minimum)
- **Connector:** PCIe 8-pin (6+2 pin)
- **Voltage:** 12V rail with adequate amperage

**Power draw by use case:**

- Idle: 50-80W
- Light gaming: 100-150W
- Heavy gaming: 150-200W
- Maximum load (RT on): Up to 235W

**Recommended PSUs:**

- **Budget:** Flex ATX 500W
- **Compact:** FSP500-50FGBBI
- **Ultra-compact:** MeanWell LOP-300-12 (bare PCB, requires knowledge)
- **Full ATX:** Any 400W+ ATX PSU you have lying around

!!!warning "6-pin Adapters"
    Some users report success with 6-pin to 8-pin adapters, but this is NOT recommended. The board CAN draw more than 150W (the 6-pin limit).

### Cooling

**High Static Pressure 120mm Fan(s)**

The stock heatsink is passive and designed for rack-mount chassis airflow. You MUST add active cooling.

**Minimum:** 1x 120mm fan
**Recommended:** 2x 120mm fans in push-pull

**Best fans (by community testing):**

1. **Arctic P12 Max** - Best value, 6.9 mmH2O static pressure
2. **Noctua NF-A12x25** - Premium option, quieter
3. **Arctic P14 Max** - Larger option if you have space

!!!danger "Fan Requirements"
    You NEED high static pressure fans (3+ mmH2O). Standard case fans won't push enough air through the dense fin array.

**Heatsink modification:** Required for axial fans

- Straighten bent fins (they're often bent from manufacturing/shipping)
- OR cut top of fin array for easier airflow (advanced)

[Full cooling guide →](../hardware/cooling.md)

### Display Connection

**DisplayPort Cable or Adapter**

The board has NATIVE DisplayPort output.

**Options:**

- **Best:** Native DisplayPort cable (1080p/1440p/4K, audio works)
- **Good:** Passive DP to HDMI adapter (1080p/1440p, audio works)
- **Avoid:** Active DP to HDMI adapter (video works, audio broken)

!!!info "Audio Limitation"
    Native DP audio is partially broken in Linux. Use passive adapters for best results, or USB audio as workaround.

[Display troubleshooting →](../troubleshooting/display.md)

### BIOS Flashing

**FAT32 USB Stick (any size)**

- For flashing modded BIOS
- Any cheap USB 2.0/3.0 stick works
- Must be formatted as FAT32

---

## Essential Software

### Linux Distribution

The BC-250 requires Linux for GPU support. Windows has NO drivers.

**Recommended (easiest setup):**

- **Fedora 42 or 43 Workstation** - Most tested, good documentation
- **Bazzite** - Gaming-focused, works out-of-box

**Also work well:**

- CachyOS (best performance, harder setup)
- Manjaro
- Arch Linux (for advanced users)

**Avoid:**

- Kernel 6.15.0-6.15.6 and 6.17.8+ (GPU driver issues)
- SteamOS (Mesa too old)
- Ubuntu (packages too old without PPAs)

[Full distribution comparison →](../linux/distributions.md)

### Modded BIOS Files

Download from: [bc250-bios repository](https://gitlab.com/TuxThePenguin0/bc250-bios/)

Required files:

- `BC250_3.00_CHIPSETMENU.ROM` - Main BIOS file
- BIOS flashing utility (included in repo)

[BIOS flashing guide →](../bios/flashing.md)

---

## Highly Recommended

### Thermal Management

**Thermal Paste**

- Stock paste is old and dried out
- **Recommended:** Arctic MX-4, Thermal Grizzly Kryonaut
- **Budget:** Any non-conductive paste
- **Advanced:** PTM7950 phase-change pad (best temps)

**Thermal Pads (for VRAM)**

The backplate gets hot (GDDR6 chips underneath).

- **Size:** 1mm or 1.5mm thickness
- **Option:** Add heatsink to backplate for passive cooling

### Storage

**M.2 NVMe SSD**

The board has one M.2 2280 slot (PCIe Gen 3 x4).

- **Minimum:** 256GB for OS + a few games
- **Recommended:** 1TB for comfortable gaming library
- **Note:** USB 3.0 port can handle external SSDs (limited to ~1 GB/s)

### Connectivity

**USB WiFi/Bluetooth Adapter**

The board has NO built-in wireless.

- **Budget:** Any USB WiFi dongle
- **Recommended:** Realtek RTL8822BU chipset (in-kernel driver as of 6.12+)
- **Alternative:** USB-C DAC/headphones for audio

### Case/Mounting

The board is bare PCB, so you'll want some kind of enclosure or mounting.

**Options:**

- **3D printed cases** - Many designs on Printables
- **GPU enclosures** - Some fit the BC-250
- **DIY:** Standoffs + acrylic/wood sheet
- **Rack mount:** Original use case (need compatible chassis)

See the BC250 Discord for community case designs and 3D printable options.

---

## Optional but Useful

### Tools

- **Phillips screwdriver** - For mounting, cable management
- **Thermal paste applicator** - Or use credit card
- **Multimeter** - For troubleshooting power issues

### Recovery Tools

**CH341A BIOS Programmer**

For BIOS recovery if flashing fails (rare but possible).

- Clips onto BIOS chip for external flashing
- Only needed if you brick the BIOS (uncommon)

### Monitoring/Tuning

These are installed via Linux package manager after setup:

- **nvtop** - GPU monitoring
- **sensors** (lm-sensors) - Temperature monitoring
- **mangohud** - In-game FPS overlay
- **CoolerControl** - Fan curve configuration

---

---

## Before You Order

### Check You Have

From existing hardware:

- [ ] Monitor with HDMI or DisplayPort
- [ ] Keyboard and mouse
- [ ] Another PC to download Linux ISO
- [ ] USB stick for installer (8GB+)
- [ ] Basic tools (screwdriver)

### Plan Your Build

- [ ] Where will it go? (desk, TV console, rack)
- [ ] How will you cool it? (fan placement)
- [ ] How will you mount/enclose it?
- [ ] Audio solution? (passive adapter, USB DAC, or DisplayPort)

---

## Ready to Order?

Once you have everything on this list, you're ready to proceed:

1. [BIOS Flashing](../bios/flashing.md) - First step after hardware arrival
2. [Quick Start Guide](quick-start.md) - Fast-track setup
3. [Full Linux Setup](../linux/distributions.md) - Detailed installation guide

---

**Community Tip:** Order the BC-250 first (longest shipping time from AliExpress), then order everything else while you wait.
