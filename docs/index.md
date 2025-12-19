# AMD BC250 Documentation

Welcome to the comprehensive documentation for the **AMD BC250** board - a powerful ex-mining board featuring a cut-down PlayStation 5 APU.

## What is the BC250?

The AMD BC250 is a compact motherboard built around AMD's "Cyan Skillfish" APU, originally designed for cryptocurrency mining. The community has transformed this hardware into a capable Linux gaming and desktop system.

### Key Specifications

- **CPU:** 6x AMD Zen 2 cores @ ~3.5GHz
- **GPU:** 24 RDNA2 Compute Units (1536 shaders)
- **Memory:** 16GB GDDR6 shared memory
- **TDP:** 220W (50W idle - 235W max load)
- **OS Support:** Linux only (no Windows GPU drivers)

!!! warning "Important"
    The BC250 requires **Linux** for GPU functionality. Windows drivers do not support the GPU, making it CPU-only on Windows.

## Quick Links

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } __Getting Started__

    ---

    New to BC250? Start here for board introduction, prerequisites, and quick setup guide.

    [:octicons-arrow-right-24: Get Started](getting-started/introduction.md)

-   :fontawesome-solid-microchip:{ .lg .middle } __Hardware__

    ---

    Detailed hardware specifications, power requirements, cooling solutions, and pinouts.

    [:octicons-arrow-right-24: Hardware Guide](hardware/specifications.md)

-   :material-linux:{ .lg .middle } __Linux Setup__

    ---

    Distribution guides, kernel configuration, Mesa installation for Fedora, Bazzite, Arch, and more.

    [:octicons-arrow-right-24: Linux Setup](linux/distributions.md)

-   :material-flash:{ .lg .middle } __BIOS & Firmware__

    ---

    BIOS flashing procedures, VRAM configuration, overclocking, and recovery.

    [:octicons-arrow-right-24: BIOS Guide](bios/flashing.md)

-   :material-gamepad-variant:{ .lg .middle } __Gaming & Performance__

    ---

    Game compatibility database, performance tips, benchmarks, and FSR setup.

    [:octicons-arrow-right-24: Gaming Guide](gaming/compatibility.md)

-   :material-wrench:{ .lg .middle } __Troubleshooting__

    ---

    Common issues and solutions for display, boot, performance, and stability problems.

    [:octicons-arrow-right-24: Troubleshooting](troubleshooting/display.md)

</div>

## Critical Requirements

Before you begin, be aware of these essential requirements:

!!! danger "Kernel Version"
    **AVOID Linux kernel 6.15.0-6.15.6 and 6.17.8-6.17.10** - Known to cause GPU driver failures. Use **6.15.7-6.17.7** or **6.17.11+** for best performance, or **6.12.x-6.14.x LTS** for stability.

!!! warning "Mesa Version"
    **Mesa 25.1.3+ minimum**, 25.1.5+ recommended for proper RADV driver support.

!!! info "BIOS Configuration"
    **P3.00 BIOS** with **512MB dynamic VRAM allocation** is the recommended configuration for most use cases.

!!! tip "Installation Boot Parameter"
    Use `nomodeset` as a kernel boot parameter during OS installation. Remove after drivers are installed.

## Recommended Linux Distributions

Based on extensive community testing:

1. **Fedora 42/43** - Most tested, beginner-friendly, Mesa 25.1+ in repos
2. **Bazzite** - Gaming-focused, works out-of-box
3. **CachyOS** - Best performance for advanced users
4. **Arch Linux** - Maximum control and flexibility
5. **Debian/PikaOS** - Stable with low power consumption

## Community

This documentation is built from the collective knowledge of the BC250 community, combining:

- **1000+ Discord community members** sharing real-world testing and troubleshooting
- **100+ verified solutions** for common issues
- **30+ tested games** with performance data
- **Multiple distribution-specific setup guides**

### Contributing

This documentation is based on community Discord discussions and the [BC250 GitHub repository](https://github.com/mothenjoyer69/bc250-documentation). If you find errors or have improvements, please contribute!

## Documentation Status

!!! success "Content Sources"
    - **Primary:** BC250 Discord community (9,716 technical messages analyzed)
    - **Secondary:** [mothenjoyer69/bc250-documentation](https://github.com/mothenjoyer69/bc250-documentation) - hardware pinouts, specifications
    - **Last Updated:** December 19, 2025

---

## Quick Start Checklist

For those eager to get started, here's a minimal checklist:

- [ ] Verify you have proper cooling (high static pressure fan recommended)
- [ ] Ensure 300W+ 12V power supply with 8-pin PCIe connector
- [ ] Download Linux distribution ISO (Fedora 43 or Bazzite recommended for beginners)
- [ ] Flash BIOS to P3.00 with 512MB dynamic VRAM allocation
- [ ] Install Linux with `nomodeset` boot parameter
- [ ] Install Mesa 25.1.5+ and RADV driver
- [ ] Install GPU governor for optimal performance
- [ ] Remove `nomodeset` from boot parameters

For detailed step-by-step instructions, see the [Quick Start Guide](getting-started/quick-start.md).
