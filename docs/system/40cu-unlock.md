# 40 CU Unlock

The BC-250 ships with 24 of 40 RDNA2 Compute Units active. The remaining 16 CUs are not physically damaged, they are fused off in firmware. The community has found a way to re-enable them at the driver level, with no permanent hardware changes and no firmware modification.

!!!success "Credits"
    All of this work is by **[duggasco](https://github.com/duggasco)** and contributors. The kernel patch, helper scripts, methodology, controlled A/B testing, and academic writeup all live at **[duggasco/bc250-40cu-unlock](https://github.com/duggasco/bc250-40cu-unlock)**. If you find this useful, star their repo. This page summarises and adds distro-specific notes and field-verified thermal data.

## What It Does

Two hardware registers control CU availability on Cyan Skillfish, and both need to be modified:

| Register | What it does | Stock | Unlocked |
|----------|--------------|-------|----------|
| `CC_GC_SHADER_ARRAY_CONFIG` | Tells the driver how many CUs exist | `0xfff80000` (24 CU) | `0xffe00000` (40 CU) |
| `SPI_PG_ENABLE_STATIC_WGP_MASK` | Tells the SPI hardware where to dispatch waves | `0x07` (WGP 0-2) | `0x1F` (WGP 0-4) |

Neither register alone produces compute scaling. Both must be modified together. See duggasco's [technical report](https://github.com/duggasco/bc250-40cu-unlock/blob/main/docs/technical-report.md) for the dual-register analysis and the [whitepaper PDF](https://github.com/duggasco/bc250-40cu-unlock/blob/main/docs/whitepaper-cu-unlock.pdf) for the full methodology.

The patched `amdgpu` module writes both registers during driver init, gated by PCI device ID `0x13FE` (BC-250 only) and a kernel module parameter (default off).

## Performance

From duggasco's controlled A/B/A testing on Vulkan llama-bench `pp512`:

| Config | Frequency | Voltage | pp512 tok/s | Power | Temp |
|--------|-----------|---------|-------------|-------|------|
| Stock 24 CU | 1500 MHz | 881 mV | 230.4 | 95 W | 79 °C |
| **40 CU unlocked** | **1500 MHz** | **874 mV** | **371.6** | **125 W** | **83 °C** |
| Ratio | same | same | **1.61x** | +30 W | +4 °C |

At 2 GHz the same test bursts to 466 tok/s, but power and temps go up substantially. **1500 MHz / 900 mV is the recommended operating point** because it captures close to the theoretical 1.67x scaling without thermal trouble. Power draw at 2 GHz depends a lot on the workload:

- duggasco's controlled `llama-bench pp512` instantaneous snapshot: ~181 W, 96 °C
- 10-minute sustained `llama-bench` with governor allowed to drift to 2060-2130 MHz: 220-230 W peak, package thermal-throttles after a few minutes
- Heavier graphics workloads can push higher: the existing [power table](../hardware/power.md#measured-power-consumption) shows stock 24 CU Cyberpunk 2077 at 235 W and Furmark OC at 320 W, so a 40 CU board under similar load is reasonably expected to land above those numbers, not below

Don't read the 181 W figure as a typical sustained draw, it's a single A/B-test snapshot. Plan PSU and cooling for the upper end (see [thermal reality](#thermal-reality-check) below).

Graphics workloads see much less benefit (`glmark2` +4.4%), because 3D rendering is fill-rate bound rather than CU-bound. This is a compute unlock, not a gaming unlock.

## Requirements and Caveats

!!!warning "Not all boards will unlock cleanly"
    The 16 fused-off CUs are not necessarily silicon-healthy. Boards with a **contiguous harvest pattern** (CU 0-5 active, CU 6-9 fused, the same on all 4 shader arrays) tend to unlock the full 40 CUs and pass compute correctness tests. Boards with a **scattered harvest pattern** may have actually defective CUs that pass enumeration but fail under load. The community has been collecting harvest maps and the contiguous case appears common but not universal.

    Before flashing modprobe configs around: run `./scripts/cu_map.sh` from duggasco's repo to see your harvest pattern. If it's scattered, plan on running the per-WGP health test (see [selective CU masking](#selective-cu-masking)) and probably ending up with somewhere between 24 and 40 stable CUs rather than the full 40.

Before you enable this:

- This rebuilds the `amdgpu` kernel module out-of-tree. Every kernel update reverts the change. Plan to rebuild after upgrades or pin your kernel.
- Sustained 40 CU at 2 GHz on the stock heatsink will throttle. Plan for a governor cap at 1500 MHz, better cooling, or both. See the [thermal reality check](#thermal-reality-check) below.
- Compute is rock solid on boards where the unlock holds, graphics has not been as widely tested. If you hit corruption in games, drop back to 24 CU or mask suspect CUs.
- Secure Boot must be off or you need to sign the rebuilt module yourself.

## Installation

### Option 1: Build Script (recommended, any distro)

The cleanest path. duggasco's `bc250-enable-40cu.sh` handles the patch, build, install, modprobe config, and a backup of the stock module.

```bash
git clone https://github.com/duggasco/bc250-40cu-unlock.git
cd bc250-40cu-unlock
sudo ./scripts/bc250-enable-40cu.sh build
sudo ./scripts/bc250-enable-40cu.sh enable    # writes modprobe config and reboots
```

Requirements: `gcc`, `make`, `zstd`, and your kernel headers (`linux-headers-$(uname -r)` on Debian/Ubuntu, `kernel-devel` on Fedora, `linux-headers` on Arch).

### Option 2: Manual Patch

For when you want to do it by hand or integrate the patch into a custom kernel build:

```bash
# In your kernel source tree
cd /path/to/linux-source/drivers/gpu/drm/amd/amdgpu/
patch -p5 < /path/to/bc250-40cu-unlock/patch/bc250-40cu-amdgpu.patch

# Build only the amdgpu module
make -C /lib/modules/$(uname -r)/build M=$(pwd) -j$(nproc) modules

# Install (zstd-compressed on most modern distros)
sudo cp amdgpu.ko.zst /lib/modules/$(uname -r)/kernel/drivers/gpu/drm/amd/amdgpu/
sudo depmod -a

# Enable
echo 'options amdgpu bc250_cc_write_mode=3' | sudo tee /etc/modprobe.d/bc250-40cu.conf
sudo reboot
```

### Option 3: CachyOS / Arch with PKGBUILD

Apply `patch/bc250-40cu-amdgpu.patch` to your kernel PKGBUILD patch set, rebuild the kernel package, install, then add the modprobe config. duggasco's `scripts/bc250-enable-40cu-arch.sh` automates this if you build from CachyOS or Arch kernel sources.

## Distro-Specific Notes

### Fedora 43 / 44 (kernel build)

Fedora's `kernel-devel` package is incomplete for building out-of-tree amdgpu modules. Two symptoms and their fixes, both verified on Fedora 43 with kernel `7.0.9-105.fc43.x86_64`:

**1. Missing scripts in `/usr/src/kernels/...`**

`kernel-devel` ships without `arch/x86/tools/cpufeaturemasks.awk` and parts of `scripts/`. You need the full kernel source instead.

```bash
# Install the full source RPM
sudo dnf install kernel-debuginfo-common-$(uname -r) kernel-debuginfo-$(uname -r)
# Or get the source tarball from koji and unpack into /usr/src/linux-$(uname -r)
```

**2. `vermagic` mismatch after build (`7.0.9` instead of `7.0.9-105.fc43.x86_64`)**

The full source has `EXTRAVERSION =` empty in the Makefile and a stale `include/generated/utsrelease.h`. Without fixing both, your module loads but the kernel rejects it for vermagic mismatch.

```bash
cd /usr/src/linux-$(uname -r)

# Fix EXTRAVERSION in the top-level Makefile
sed -i 's/^EXTRAVERSION =.*/EXTRAVERSION = -105.fc43.x86_64/' Makefile

# Force the right UTS_RELEASE
cat > include/generated/utsrelease.h <<EOF
#define UTS_RELEASE "$(uname -r)"
EOF

# Build amdgpu only
make M=drivers/gpu/drm/amd/amdgpu modules
```

Adjust `-105.fc43.x86_64` to match your actual `uname -r` suffix.

### Ubuntu / Debian

Standard path. `apt install linux-headers-$(uname -r) build-essential zstd` then run the build script. No known gotchas.

### CachyOS / Bazzite

The kernel patch is being pursued upstream as [CachyOS/kernel-patches#159](https://github.com/CachyOS/kernel-patches/pull/159). Until it lands, treat it as the manual patch path with the `linux-cachyos` PKGBUILD.

## Verification

After reboot, you should see all four shader arrays log the register writes, the kernel report 40 active CUs, and Vulkan agree.

Real output from a working install (Fedora 43, kernel 7.0.9-105.fc43.x86_64):

```text
$ cat /sys/module/amdgpu/parameters/bc250_cc_write_mode
3

$ sudo dmesg | grep -E 'bc250-40cu|active_cu_number'
amdgpu 0000:01:00.0: bc250-40cu-enable: mode=3 se=0 sh=0 CC=0xfff80000->0xffe00000 SPI=0x00000007->0x0000001f
amdgpu 0000:01:00.0: bc250-40cu-enable: mode=3 se=0 sh=1 CC=0xfff80000->0xffe00000 SPI=0x00000007->0x0000001f
amdgpu 0000:01:00.0: bc250-40cu-enable: mode=3 se=1 sh=0 CC=0xfff80000->0xffe00000 SPI=0x00000007->0x0000001f
amdgpu 0000:01:00.0: bc250-40cu-enable: mode=3 se=1 sh=1 CC=0xfff80000->0xffe00000 SPI=0x00000007->0x0000001f
amdgpu 0000:01:00.0: SE 2, SH per SE 2, CU per SH 10, active_cu_number 40

$ RADV_DEBUG=info vulkaninfo --summary 2>&1 | grep num_cu
    num_cu = 40
    num_cu_per_sh = 10
```

If `active_cu_number` is 24 or `num_cu` is 24, the patched module did not load. Check that `/etc/modprobe.d/bc250-40cu.conf` exists, that you have not overwritten your patched `amdgpu.ko.zst` with a stock one from a kernel update, and that Secure Boot is off (or that you signed the module).

!!!tip "Check with dmesg"
    The authoritative check is `dmesg | grep active_cu_number`. The wrapper's `status` subcommand uses a more conservative heuristic and can occasionally underreport a working install.

## Governor Settings for Sustained Operation

40 CU at the governor's default 2 GHz pushes 96-100 °C on stock cooling, with sustained `llama-bench` peaking at 220-230 W and heavier workloads (gaming, Furmark) reasonably expected to draw more. To run sustained workloads, cap frequency at 1500 MHz and use the community-recommended voltage curve. From a working `cyan-skillfish-governor-smu` config tested under sustained Vulkan compute:

```toml
# /etc/cyan-skillfish-governor-smu/config.toml

[[safe-points]]
frequency = 350
voltage = 700

[[safe-points]]
frequency = 1500
voltage = 900

[[safe-points]]
frequency = 2000
voltage = 1000

[[safe-points]]
frequency = 2200
voltage = 1000
```

The 1000 mV ceiling at 2150-2200 MHz is the practical stable limit on stock cooling. Older config examples that ran 1050 mV at 2200 MHz are overvolted and worth trimming back. See [GPU Governor](governor.md) for the full setup.

## Thermal Reality Check

Sustained 10-minute `llama-bench` on Llama-3.2-1B Q4_K_M with 40 CU at 2 GHz on a BC-250 with the stock heatsink and two Arctic P12 Max fans in push-pull (verified):

| Metric | Average | Peak |
|--------|---------|------|
| GPU edge temp | 89.6 °C | **107 °C** |
| Package power (PPT) | 136 W | **223 W** |
| CPU temp | 96.7 °C | **100 °C (TJmax)** |
| VRM MOSFET temp | 57 °C | 58.5 °C |
| Fan speed | ~2950 RPM | **2977 RPM (ceiling)** |

Sustained throughput drops about 10% over 10 minutes (3034 → 2835 tok/s) as the package throttles. The bottleneck is heatsink dissipation and CPU thermals, not VRM headroom.

**Bottom line:** if you want to run a sustained overclock across all 40 CUs, you need effective cooling. Stock heatsinks and entry-level dual-fan setups will throttle under sustained load. With a 1500 MHz governor cap the heat stays manageable on the configuration above and the board runs comfortably. The unlock itself is solid: 25 minutes of looped Vulkan compute correctness testing produced zero `fp_errors`, zero `int_errors`, no amdgpu hangs, no resets, no oops. The thermal envelope is the constraint, not the silicon.

## Selective CU Masking

Not every unlocked CU may be silicon-healthy on every board. Boards with scattered (non-contiguous) harvest patterns may have defective CUs that pass enumeration but fail compute. duggasco ships a per-WGP health test that reboots into each WGP configuration in isolation and runs correctness checks:

```bash
sudo ./scripts/bc250-cu-health-test.sh start
./scripts/bc250-cu-mask.sh --results /var/lib/bc250-cu-health-test/results.tsv --install
```

The mask uses the standard `amdgpu.disable_cu` parameter at WGP granularity (disabling CU 6 also disables CU 7, same WGP). Full details, examples, and the `cu_map.sh` visualisation in duggasco's [README](https://github.com/duggasco/bc250-40cu-unlock#selective-cu-masking).

## Disabling and Reverting

```bash
# Remove the modprobe config and reboot to stock 24 CU
sudo ./scripts/bc250-enable-40cu.sh disable

# Restore the original amdgpu module from backup
sudo ./scripts/bc250-enable-40cu.sh restore
```

The backup module is saved at `/lib/modules/$(uname -r)/kernel/drivers/gpu/drm/amd/amdgpu/amdgpu.ko.*.bc250-backup-*` during install. Keep it. If a kernel update replaces your patched module, the rollback is instant.

## Going Deeper

- [duggasco/bc250-40cu-unlock](https://github.com/duggasco/bc250-40cu-unlock): the full repo with scripts, patch, technical report, whitepaper
- [Technical report](https://github.com/duggasco/bc250-40cu-unlock/blob/main/docs/technical-report.md): register map, 4-state controlled test, dual-register architecture analysis
- [Whitepaper PDF](https://github.com/duggasco/bc250-40cu-unlock/blob/main/docs/whitepaper-cu-unlock.pdf): 8-page academic writeup including the n=58 community harvest map survey
- [CachyOS/kernel-patches#159](https://github.com/CachyOS/kernel-patches/pull/159): upstream patch submission
- [GPU Governor](governor.md): voltage/frequency curve setup
- [Sensors & Monitoring](sensors.md): reading temps, power, fan speed
