# Introduction to the AMD BC-250

The AMD BC-250 is an ex-cryptocurrency mining board featuring a cut-down PlayStation 5 APU. What started as specialized mining hardware has become a surprisingly capable budget gaming and compute platform thanks to extensive community reverse-engineering and Linux driver development.

## What is the BC-250?

The BC-250 was originally designed for cryptocurrency mining, likely Ethereum, before being repurposed and sold on the surplus market. It features:

- **CPU:** 6x Zen 2 cores running at ~3.5GHz
- **GPU:** 24 RDNA2 Compute Units (codename "Cyan Skillfish")
- **Memory:** 16GB GDDR6 shared between CPU and GPU
- **Connectivity:** DisplayPort, M.2 NVMe slot, USB 3.0 ports
- **Power:** PCIe 8-pin connector, 220W TDP

## Key Capabilities

### Gaming Performance

With proper Linux setup, the BC-250 delivers performance comparable to:

- RX 6600 / GTX 1660 Ti range
- 1080p gaming at medium-high settings
- Ray tracing capable (though limited)
- Frame generation support via FSR

**Example Performance:**
- Cyberpunk 2077: 60-90 FPS (1080p, high settings, FSR enabled)
- Control: 40 FPS with ray tracing
- DMC 5: 100+ FPS (1080p, high settings)

### Compute & AI

- LLM inference via llama.cpp (Vulkan): ~60 tokens/sec for 8B models
- Stable Diffusion: ~1.1 it/s (512x512, SD1.5)
- 10-12GB usable VRAM depending on configuration

### Limitations

!!!warning "Linux Only for Graphics"
    Windows has NO GPU driver support. You must use Linux for any graphics acceleration, gaming, or compute workloads.

**Other limitations:**

- No native audio over DisplayPort (workarounds available)
- No built-in WiFi/Bluetooth (USB adapters work)
- Limited instruction set (some AVX features missing)
- High idle power consumption (~50-80W without optimization)

## Why BC-250?

### The Good

**Budget Gaming:** One of the cheapest ways to build a capable 1080p gaming PC

**Massive Community:** Active Discord with 1000+ members sharing mods, troubleshooting, and improvements

**Open Documentation:** Multiple GitHub repositories with setup guides, BIOS mods, and driver patches

**Hackable:** Modded BIOS unlocks VRAM configuration, overclocking, and other features

### The Challenges

**Requires Work:** Not plug-and-play. Expect to flash BIOS, configure Linux, and troubleshoot

**Cooling Needed:** Stock heatsink requires modification or replacement for reliable operation

**No Warranty:** Ex-mining hardware sold "as-is"

**Power Hungry:** Even at idle, draws more power than modern APUs

## Who Is This For?

The BC-250 is ideal if you:

- Want a cheap Linux gaming machine
- Enjoy tinkering and customization
- Need budget compute/AI inference
- Like unique hardware projects
- Want to learn Linux and system building

**Not recommended if:**

- You need Windows gaming support
- You want plug-and-play experience
- You need production-stable hardware
- You want modern power efficiency

## What's Next?

Ready to get started? Check out:

- [Quick Start Guide](quick-start.md) - Fast-track setup checklist
- [Prerequisites](prerequisites.md) - What you need to buy
- [BIOS Flashing](../bios/flashing.md) - Essential first step

## Board Versions

Most BC-250 boards are functionally identical, but you may encounter different BIOS versions (P2.00, P3.00, P4.00, P5.00). All can be flashed to the community-modded BIOS for optimal performance.

Some boards have minor heatsink variations (number of connecting tabs on fin tops), but these don't significantly affect cooling performance.

---

**Community Resources:**

- [GitHub Documentation](https://github.com/elektricM/amd-bc250-docs)
- [BIOS Repository](https://gitlab.com/TuxThePenguin0/bc250-bios/)
- Discord Server (1000+ members, link in GitHub)
