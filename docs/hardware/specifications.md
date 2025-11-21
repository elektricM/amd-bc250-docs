# Hardware Specifications

Complete technical specifications for the AMD BC-250 board.

## APU Overview

The BC-250 features a cut-down PS5 APU (codenamed "Oberon" / "Cyan Skillfish"):

### CPU Specifications

- **Cores:** 6x Zen 2 cores (2 cores disabled from original 8-core design)
- **Base Clock:** ~3.5 GHz
- **Architecture:** Zen 2 microarchitecture
- **Instruction Set:** x86-64
- **Cache:** Shared L3 cache (reduced from PS5 config)

!!!info "CPU Performance"
    The CPU is intentionally cut down for mining purposes. While adequate for gaming and general computing, it's not the board's primary strength.

### GPU Specifications

- **Architecture:** RDNA 2 (same generation as RX 6000 series)
- **Compute Units:** 24 CUs (down from 36 CUs in full PS5 APU)
- **Codename:** Cyan Skillfish (gfx1013)
- **Base Frequency:** 1500 MHz (locked without governor)
- **Maximum Frequency:** 2000-2230 MHz (with kernel patch and governor)
- **Performance:** Comparable to RX 6600 / RTX 3060 Ti in gaming workloads

!!!success "GPU Features"
    - Hardware ray tracing support (RDNA 2 RT cores)
    - FSR (FidelityFX Super Resolution) compatible
    - Vulkan 1.3 support
    - No video encoding/decoding (VCN disabled)

### Memory Configuration

- **Total Memory:** 16GB GDDR6
- **Memory Type:** GDDR6 (PS5 specification)
- **Memory Speed:** 14 Gbps
- **Memory Bus:** 256-bit
- **Memory Bandwidth:** ~448 GB/s

!!!warning "Memory Split Required"
    The 16GB is shared between CPU and GPU. You must configure the split in BIOS:

    - **512MB GPU / 15.5GB CPU:** Recommended for desktop/light gaming
    - **4GB GPU / 12GB CPU:** Recommended for modern games
    - **Dynamic allocation:** Available but can cause issues with some applications

[See VRAM Configuration Guide](../bios/vram.md) for detailed setup instructions.

## Physical Specifications

### Board Dimensions

- **Form Factor:** Custom mining board (non-standard)
- **Length:** Approximately 200mm
- **Width:** Approximately 115mm
- **PCB Thickness:** Standard
- **Weight:** ~400g (with heatsink)

### Connectors and Headers

#### Power Connectors

- **Main Power:** 1x PCIe 8-pin (6+2 pin)
- **Power Delivery:** Direct 12V input
- **Maximum Draw:** 220W TDP (measured up to 235W in extreme cases)

!!!danger "Power Requirements"
    Use a quality PSU with at least 220W available on the 12V rail. Poor quality power supplies can cause instability and system crashes.

#### Display Output

- **DisplayPort:** 1x full-size DisplayPort 1.4
- **Resolution Support:** Up to 4K @120Hz, 8K @60Hz
- **Audio:** Audio over DisplayPort (compatibility varies)
- **HDMI:** None (requires DP to HDMI adapter)

#### Storage

- **M.2 Slot:** 1x M.2 2280 slot (PCIe Gen 3 x2)
- **Speed:** ~1 GB/s maximum
- **USB:** 1x USB 3.0 port (Type-A)
- **USB Speed:** ~480 MB/s (SATA speed equivalent)

#### Fan Headers

- **Primary Fan:** 1x 4-pin PWM header (J1)
- **Secondary Fan:** 1x 4-pin PWM header (J4003)
- **Voltage:** 12V
- **Control:** PWM (Pulse Width Modulation)

!!!tip "Fan Control"
    The nct6687 kernel module enables PWM control. Without it, sensors are read-only.

#### Other Headers

- **Power Button:** 2-pin header (rear of board)
- **Debug Header:** 20-pin AMD HDT1 debug connector
- **SPI Flash:** Header for BIOS flashing
- **Super I/O:** NCT6686/6687 chip for sensors and fan control

## Heatsink and Cooling

### Stock Heatsink

- **Type:** Passive aluminum fin stack
- **Fin Count:** High-density vertical fins
- **Orientation:** Fins run front-to-back
- **Mounting:** Screwed directly to PCB
- **Base:** Direct contact with APU die

!!!warning "Stock Cooling Inadequate"
    The stock heatsink is designed for passive or low-airflow rack cooling. For desktop gaming use, active cooling is **required**.

### Thermal Interface

- **APU Thermal Compound:** May be dried out on used boards
- **Memory Thermal Pads:** On underside of board
- **Recommended Refresh:** Replace thermal paste and pads for optimal performance

[See Cooling Guide](cooling.md) for recommended solutions.

## Power Consumption

### Measured Power Draw

| State | Power Consumption |
|-------|-------------------|
| Idle (no governor) | 85-105W |
| Idle (with governor) | 65-85W |
| Desktop use | 70-90W |
| Light gaming | 120-150W |
| AAA gaming | 160-200W |
| Maximum (Cyberpunk RT) | 235W |

!!!info "Power Optimization"
    Installing the GPU governor can save 20-30W at idle by reducing GPU frequency to 1000 MHz.

### Power Efficiency

- **Efficiency:** Moderate (mining-optimized, not efficiency-optimized)
- **Idle Power:** Higher than typical desktop due to GDDR6 memory
- **Comparison:** Less efficient than modern desktop components but acceptable for the performance level

## Limitations and Notes

### Known Hardware Limitations

#### No Video Encode/Decode

- **VCN (Video Core Next):** Disabled in hardware or firmware
- **Hardware Encoding:** Not available
- **Hardware Decoding:** Not available
- **Software Fallback:** CPU decoding works but is power-hungry

!!!failure "No VCN Support"
    There is no way to enable hardware video encoding/decoding. The silicon may have been binned without working VCN, or it's disabled in SMU firmware.

#### IOMMU Issues

- **IOMMU:** Does not work reliably
- **Virtualization:** GPU passthrough not possible
- **Workaround:** Disable IOMMU in BIOS and kernel parameters

#### Memory Architecture

- **Unified Memory:** CPU and GPU share the same 16GB pool
- **Dynamic Allocation:** Can cause issues with some games
- **Static Split:** More reliable but less flexible

### Unsupported Features

- **Windows Gaming:** No Windows GPU drivers available
- **Secure Boot:** Not supported
- **TPM:** Not present
- **Thunderbolt:** Not available

## Comparison to Similar Hardware

### vs. PlayStation 5

| Feature | BC-250 | PlayStation 5 |
|---------|--------|---------------|
| CPU Cores | 6 cores | 8 cores |
| CPU Clock | ~3.5 GHz fixed | Up to 3.5 GHz (variable) |
| GPU CUs | 24 CUs | 36 CUs |
| GPU Clock | 2000 MHz max | 2230 MHz (variable) |
| Memory | 16GB GDDR6 | 16GB GDDR6 |
| VCN | Disabled | Enabled |
| Form Factor | Mining board | Console |

### vs. Desktop GPUs

**Approximate Performance Equivalents:**

- **Rasterization:** Between RX 6600 and RX 6600 XT
- **Ray Tracing:** Similar to RX 6600 (entry-level RT performance)
- **Compute:** Similar to RX 6600 (RDNA 2 architecture)
- **Memory:** 16GB total (configurable split) vs 8GB dedicated VRAM

!!!success "Gaming Performance"
    For 1080p gaming, the BC-250 performs admirably, achieving 60+ FPS in most modern games at high settings.

## Verification Commands

Check your hardware specifications with these commands:

```bash
# Check CPU information
lscpu | grep -E "Model name|CPU\(s\)|Thread|Core"

# Check GPU information
lspci | grep VGA
vulkaninfo | grep deviceName

# Check memory information
free -h
vulkaninfo | grep -i memory

# Check Mesa version
glxinfo | grep "OpenGL version"

# Check kernel version
uname -r

# Check sensors
sensors
```

## See Also

- [Power Requirements](power.md)
- [Cooling Solutions](cooling.md)
- [Display Connectivity](display.md)
- [BIOS Configuration](../bios/flashing.md)
