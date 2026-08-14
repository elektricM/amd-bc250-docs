# 8 Core CPU Unlock

The BC-250 ships with 6 of its 8 Zen 2 cores active. The other two are **not fused off** — they are masked by a writable SMU register, and flipping that register brings them back. This is the CPU counterpart to the [40 CU Unlock](40cu-unlock.md).

!!!success "Credits"
    The BIOS-side unlock is the work of **[Forbidden-Darkness](https://github.com/Forbidden-Darkness/AMD-BC-250-UEFI-v2.2-Firmware-Menu-Script)**, whose `MeiMeiDXE-T-v2` mod ships a purpose-built `Bc250CoreUnlockDxe` driver, and of the **BC-250 Telegram community**, who found, tested and documented it. The Linux-side tool and the register documentation on this page come from **[GabriWar](https://github.com/GabriWar/bc250-core-cu-unlock)**, derived by reverse engineering that driver. The matching 8-core ACPI tables are **[mendesrr](https://github.com/mendesrr/bc250-acpi-fix-updated-8c)**'s rebuild of the **[bc250-collective](https://github.com/bc250-collective/bc250-acpi-fix)** fix.

## What It Does

One register controls core availability:

| Register | What it does | Stock | Unlocked |
|----------|--------------|-------|----------|
| `SMN 0x0115A870` | CPU core enable bitmask, one bit per core | `0x77` (cores 3 and 7 masked) | `0xFF` (all 8 cores) |

`0x77` is `0b0111_0111` — bits 3 and 7 clear. That is one core disabled per CCX, the symmetric pattern you get from a firmware down-core, not from defect harvesting.

The mask is not writable directly. It is set through an **SMU mailbox command, message `0x98`**, reached over SMN via the PCI config index/data pair `0xB8`/`0xBC` on device `00:00.0`:

```
SmnWrite32(0x3B10A80, 0)           # clear response register
SmnWrite32(0x3B10A88, 0x115A870)   # arg0 = target register
SmnWrite32(0x3B10A8C, 0)           # arg1
SmnWrite32(0x3B10A20, 0x98)        # message id
poll 0x3B10A80 until == 1 (ok) or 0xFC..0xFF (error)
```

CPU topology is fixed by firmware at reset, so the cores do not appear until the platform re-enumerates.

!!!danger "Stop the governor first, including for reads"
    `cyan-skillfish-governor-smu` drives the same `0xB8`/`0xBC` index/data pair on `00:00.0`. Every SMN access here is a write to the index register followed by a read or write of the data register, and the governor can land between the two. You then read or write a completely different SMN address than the one you selected.

    Stop the service before touching the mailbox, do the work, start it again. This applies to reads as much as writes, so a "just checking the mask" one-liner is not safe either.

    Reported from a second board in [#41](https://github.com/elektricM/amd-bc250-docs/pull/41).

!!!important "Warm reset keeps it, cold boot loses it"
    | reset | result |
    |---|---|
    | **warm** (`reboot`, `systemctl reboot`) | mask preserved → cores stay unlocked |
    | **cold** (`poweroff`, PSU switch, unplug) | mask reverts to `0x77` → back to 6 cores |

    Nothing is written to flash by the Linux method, so **cutting power is a guaranteed way back to a stock 6-core machine** if anything misbehaves.

## Two Ways To Do It

### Method 1: Linux (no BIOS flash)

Applies the unlock at boot and warm-reboots once so firmware re-enumerates. Reverted by any cold boot, which is also its safety net.

```bash
git clone https://github.com/GabriWar/bc250-core-cu-unlock
cd bc250-core-cu-unlock

sudo ./bc250-8core-unlock.sh status     # show the current mask
sudo ./bc250-8core-unlock.sh apply      # unlock now, then: sudo reboot
sudo ./bc250-8core-unlock.sh install    # persist: installs and enables a systemd unit
```

The optional systemd unit re-applies the mask after a cold boot. **It never reboots for you** — the cores appear on your next reboot, whenever you choose to do one.

!!!danger "Do not have anything reboot for you here"
    An earlier version of that tool rebooted automatically so firmware would re-enumerate in the same session. On a real board it **bootlooped**: the reset it triggered did not preserve the mask, so every boot read `0x77`, re-applied and reset again. Both intended safeguards also failed — an attempt counter under `/var` (not reliably writable that early in boot) and `systemctl reboot` (cannot work before D-Bus is up). Set the mask, then let the user choose when to reboot.

Requires `pciutils` (`setpci`) and root.

### Method 2: BIOS flash (permanent, no extra reboot)

The `MeiMeiDXE-T-v2` mod contains `Bc250CoreUnlockDxe`, which performs the same SMU sequence pre-OS on every boot, so there is no double-boot and nothing to install in the OS. Get it from [Forbidden-Darkness/AMD-BC-250-UEFI-v2.2-Firmware-Menu-Script](https://github.com/Forbidden-Darkness/AMD-BC-250-UEFI-v2.2-Firmware-Menu-Script) and follow the [BIOS Flashing Guide](../bios/flashing.md).

!!!warning "This is a P3.00-based mod"
    It is built on P3.00 and flashes the boot block. If you are running P5.00 or a different mod, this is a downgrade and you will lose whatever that mod unlocked. Have a [hardware programmer and a verified backup](../bios/recovery.md) before flashing. If you only want to try the cores out, use Method 1 first — it cannot brick anything.

## Are The Extra Cores Healthy?

**On the board tested, yes — completely.** This looks like market segmentation rather than defect harvesting.

Per-physical-core `stress-ng --cpu-method all --verify` (20 s each). `--verify` validates the *result* of each computation, so a marginal core shows up as wrong answers rather than merely lower throughput:

| core | bogo-ops/s | vs median | verify failures |
|---|---|---|---|
| 0 | 1526.19 | +0.0% | 0 |
| 1 | 1526.74 | +0.1% | 0 |
| 2 | 1529.30 | +0.2% | 0 |
| **3** | **1530.10** | **+0.3%** | **0** |
| 4 | 1524.21 | −0.1% | 0 |
| 5 | 1525.53 | +0.0% | 0 |
| 6 | 1522.13 | −0.2% | 0 |
| **7** | **1524.00** | **−0.1%** | **0** |

Cores 3 and 7 are the newly unlocked ones. Whole-die spread is ±0.3%, which is measurement noise — core 3 was in fact the fastest core on the die. A 60 s all-16-thread `--verify` run passed with zero failures and zero machine-check events.

The repo ships `test-cores.sh` to run this sweep on your own board. As with the CU unlock, silicon lottery applies and this is a single board — any core reporting verify failures is producing wrong results and should not be trusted.

## Performance

### Where The New Cores Land

Firmware does not slot the new cores into the CPU numbers matching their mask bits. It enumerates them last:

| Silicon | Linux CPUs | SMT siblings |
|---|---|---|
| The original six cores (enabled under `0x77`) | 0-5 | 8-13 |
| The newly unlocked pair (mask bits 3 and 7) | 6-7 | 14-15 |

SMT siblings pair CPU `N` with CPU `N+8`, so the stock configuration lives on exactly the CPU set `0-5,8-13`. That set is what makes a fair benchmark possible on a single boot.

### A Clock-Matched A/B On One Boot

Pinning to the stock CPU set reproduces the 6-core machine without reverting anything: same silicon, same clocks, same boot. An unpinned `--cpu 12` run would not be a fair baseline, because the scheduler would spread the 12 workers over all eight cores.

```bash
taskset -c 0-5,8-13 stress-ng --cpu 12 --cpu-method all --verify --metrics-brief -t 60   # stock six cores
stress-ng --cpu 16 --cpu-method all --verify --metrics-brief -t 60                       # all eight cores
```

| Config | bogo ops (60 s) | bogo ops/s | vs baseline |
|---|---|---|---|
| Stock six cores (`taskset -c 0-5,8-13`) | 732,343 | 12,205 | baseline |
| **All eight cores, 16 threads** | **1,002,698** | **16,711** | **+36.9%** |

That is slightly above the +33.3% the core count alone predicts, which is within run-to-run variance for a 60 s sample. Both runs passed `--verify` with `failed: 0`, so the benchmark doubles as a wrong-results check on the newly unlocked cores.

Tested by: @Weijtmans. BC-250, Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29, both runs on one boot at identical clock settings.

### 7-zip On The First Board

7-zip multithreaded benchmark, from before the method above existed:

| Config | 7-zip total MIPS |
|---|---|
| 6 cores, CPU OC at 3800 MHz | 53,610 |
| **8 cores, stock clocks** | **68,039** |

That is **+26.9%** with the 6-core baseline overclocked and the 8-core run at stock clocks, so treat it as indicative rather than clock-matched. It is still a useful second data point from a second board and a different workload.

Scaling is best in threaded workloads. Anything memory-bound is still limited by the 450 MHz FCLK, which this does not change.

## Known Issues

- **GPU frequency reporting breaks.** After unlocking, `pp_dpm_sclk` reports nonsense (for example `1: 15Mhz` where it should read `1500Mhz`) and `gpu_busy_percent` may return empty. The GPU still clocks correctly according to your [governor](governor.md) curve — this is a monitoring bug, not a performance one. Reported by the BC-250 Telegram community and reproduced on the tested board.
- **Your ACPI tables need updating.** The 6-core SSDTs leave CPUs 12-15 without C-states — see [below](#acpi-tables-must-be-updated-too).
- **Any existing overclock or undervolt is no longer valid.** See [below](#your-overclock-needs-re-validating).
- **The upstream detect tool is a tuner, not a diagnostic.** Its core-detection path applies an SMU frequency and voltage state as a side effect, which resets any undervolt scale you had, and it writes an `overclock.conf` into the working directory. Do not reach for it as a read-only status check. Reported from a second board in [#41](https://github.com/elektricM/amd-bc250-docs/pull/41).
- **Other mask values may exist.** Every board checked so far reads `0x77`, but a board that masks a different pair of cores would read something else. The Linux tool refuses to act on any mask it does not recognise rather than guessing.

## ACPI Tables Must Be Updated Too

!!!warning "The 6-core ACPI fix leaves 4 threads with no C-states"
    If you use the community [ACPI fix](https://github.com/bc250-collective/bc250-acpi-fix), its tables are wrong for an 8-core machine and you should update them.

`SSDT-CST.aml` declares one processor object per **thread**. The 6-core tables stop at `C00B` — 12 threads. Unlock all 8 cores and you have 16, so **CPUs 12-15 receive no cpuidle states at all**: they cannot enter any C-state and burn power at idle.

Measured on the tested board, before updating the tables:

```
cpu0:  4 idle states
cpu6:  4 idle states
cpu12: 0 idle states   <-- no C-states
cpu14: 0 idle states
cpu15: 0 idle states
```

The 8-core rebuild by **[mendesrr](https://github.com/mendesrr/bc250-acpi-fix-updated-8c)** extends the declarations to `C00F`, covering all 16 threads. That repo's README has the install steps for Bazzite, SteamOS and CachyOS.

On Arch/CachyOS the short version is:

```bash
sudo mkdir -p /etc/initcpio/acpi_override/
cd /etc/initcpio/acpi_override/
sudo wget https://github.com/mendesrr/bc250-acpi-fix-updated-8c/raw/refs/heads/main/SSDT-CST.aml \
          https://github.com/mendesrr/bc250-acpi-fix-updated-8c/raw/refs/heads/main/SSDT-PST.aml
sudo mkinitcpio -P
sudo reboot
```

Back up any existing `.aml` files **outside** `/etc/initcpio/acpi_override/` first. The `acpi_override` hook globs `*.aml` there, so a stray copy alongside the new ones would load two sets of tables.

Check it worked:

```bash
cpupower idle-info
# every CPU, including 12-15, should now report C-states
```

## Your Overclock Needs Re-validating

Two extra cores change the electrical and thermal behaviour of the whole SoC. A curve tuned at 6 cores is not a curve that holds at 8, and the failure mode is not a clean error — it is intermittent instability that reads like a bad game, a bad driver or a bad kernel.

- **Load-line droop gets worse.** Eight cores pulling current sag the rail further than six at the same requested voltage. A voltage that was marginal-but-stable at 6 cores can land under the silicon's floor at 8 during an all-core transient. Undervolts break first.
- **Thermals rise.** Measured on the tested board: **82.4 °C Tctl at stock clocks** under a 16-thread load, before any overclock.
- **The shared budget shifts.** CPU and GPU draw from one SoC power and thermal envelope. Two more cores take a bigger slice, leaving the GPU less headroom than it had. If you have also done the [40 CU Unlock](40cu-unlock.md), both ends now compete harder.
- **Your sweet spot moves.** Peak-throughput frequency, the efficiency knee, and the point where more clock stops buying anything are all functions of the thermal envelope — and that envelope just changed.

After unlocking, redo the work in this order:

1. **Per-core correctness** on all 8 cores (`test-cores.sh`) before tuning anything.
2. **Re-benchmark** and find the new plateau. Frequencies that paid off at 6 cores may sit past the knee at 8.
3. **Re-check droop** — measure actual voltage under an all-core load, not at idle, and compare against what you asked for. Widen the margin wherever it sagged.
4. **Re-cliff the undervolt** at 8 cores. Do not carry the old curve across.
5. **Re-validate thermals** under sustained load rather than a burst, with CPU and GPU loaded *together* — the co-load case is where the shared envelope actually bites.

Treat your old numbers as a starting hypothesis, not a configuration.

## Verifying

```bash
lscpu | grep -E 'Core\(s\) per socket|^CPU\(s\)'
# Core(s) per socket:  8
# CPU(s):              16

sudo ./bc250-8core-unlock.sh status
# SMN 0x0115A870 = 0xFF  enabled=[0 1 2 3 4 5 6 7] disabled=[]
```

If the mask reads `0xFF` but `lscpu` still shows 6 cores, firmware has not re-enumerated yet — warm reboot.
