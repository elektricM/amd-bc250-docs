# DisplayPort Audio: Desync, Delay and Crackle

Audio over the BC-250's DisplayPort output can arrive slightly late, at a slightly low pitch, or with periodic crackle, while video stays perfect. This page explains the actual root cause (a spread-spectrum bug in the display driver, specific to this GPU), how to tell it apart from an adapter problem, and how to fix it.

This is a different problem from [active DP-HDMI adapters breaking audio](display.md#special-case-active-vs-passive-adapters). That one is about the adapter re-encoding the stream. The one on this page is in the kernel and affects the DisplayPort output itself, so it can show up even on a native DP monitor or a passive adapter.

---

## Symptoms

- Audio and video out of sync, usually audio around 100 to 200 ms late
- Audio pitch slightly too low
- Periodic crackle or clicking as the audio buffer drifts
- Video is unaffected

If instead you get no audio at all on an active adapter, that is the adapter issue, not this one. See [Active vs Passive Adapters](display.md#special-case-active-vs-passive-adapters).

---

## Root cause

The BC-250's Cyan Skillfish GPU is the only shipping AMD GPU that uses the **DCN201** display block, so this code path is effectively BC-250 only.

The DisplayPort reference clock is programmed with spread spectrum applied, even though the board firmware says DisplayPort spread spectrum should be off. DisplayPort audio is clocked off that same reference, so dithering the reference clock dithers the recovered audio sample clock. Video tolerates the dither, audio timing does not.

The chain, confirmed against the mainline kernel source:

1. `dcn201_clk_mgr.c`, `dcn201_clk_mgr_construct()` sets `dprefclk_ss_percentage = 0`, then calls `dce_clock_read_ss_info()` at the end of the function with no condition.
2. `dce_clock_read_ss_info()` (in `dce_clk_mgr.c`) queries the VBIOS. The GPU-PLL query returns unsupported on this VBIOS revision, so it falls through to the DisplayPort query.
3. `bios_parser2.c`, `get_ss_info_v4_2()` returns `smu_info->gpuclk_ss_percentage` (the GPU core clock's spread spectrum, a nonzero value like `0x0177`) for the DisplayPort query.
4. That nonzero percentage is applied to the DisplayPort reference clock in `dce_adjust_dp_ref_freq_for_ss()`, which shifts the audio clock.

The VBIOS field that actually says "DisplayPort spread spectrum is off" (`integrated_info->dp_ss_control = 0`) is read in `dcn201_link_init()` and honored for link training, but the clock manager never looks at it. That mismatch is the bug.

!!!note "Not fixed in mainline as of kernel 7.1.x"
    Earlier notes here said this was resolved in 6.19.10 or newer. That refers to downstream community kernels (patched Bazzite / CachyOS builds), not mainline. Mainline 7.1.3 still calls `dce_clock_read_ss_info()` unconditionally in `dcn201_clk_mgr_construct()`, so the desync is present. A cleaner fix that gates the call on `dp_ss_control` was being prepared for the kernel mailing list at the time of writing but is not merged yet. Verify per kernel rather than trusting a version number (see below).

---

## Diagnosis

First confirm the audio sink exists and is valid. On a working DisplayPort link you get a present, valid ELD:

```bash
aplay -l
# card 0: Generic [HD-Audio Generic], device 3: HDMI 0 [<your monitor name>]

cat /proc/asound/card*/eld*
# monitor_present   1
# eld_valid         1
# connection_type   DisplayPort
```

If `eld_valid` is `1` and the audio format looks correct, but you still hear the delay, pitch or crackle, that points at the reference-clock spread spectrum rather than the adapter or the EDID.

Capture the EDID with `cat`, not a file-manager copy (copying a sysfs file gives a 0-byte result):

```bash
cat /sys/class/drm/card*-DP-1/edid > ~/edid
edid-decode ~/edid   # look for the Audio Data Block and sample rates
```

While audio is playing, the negotiated stream parameters confirm the format actually in use:

```bash
cat /proc/asound/card*/pcm*p/sub*/hw_params   # rate, format, channels during playback
```

### Is the fix in my kernel?

Version numbers are unreliable here because the fix has only lived in downstream kernels so far. Check the source directly. `kernel-devel` ships headers only, so you need the matching kernel source tree (for example from `dnf download --source kernel` or your distro's kernel-source package):

```bash
grep -n dp_ss_control <kernel-source>/drivers/gpu/drm/amd/display/dc/clk_mgr/dcn201/dcn201_clk_mgr.c
```

If `dcn201_clk_mgr_construct()` guards `dce_clock_read_ss_info()` with a `dp_ss_control` check, the fix is present. If the call is unconditional, it is not.

---

## Fixes

### Hardware workarounds (no rebuild)

These sidestep the bug rather than fixing it:

- A passive DP-to-HDMI adapter or a native DisplayPort monitor keeps things simplest.
- A USB DAC moves audio off DisplayPort entirely, which is the reliable route if you cannot rebuild the kernel.

### Kernel patch (the actual fix)

There is no runtime knob. `ignore_dpref_ss` is a `dc_config` field, not a module parameter and not reachable through `amdgpu.dcdebugmask`, so the fix needs a rebuilt amdgpu module or kernel.

The patch clears the DisplayPort spread spectrum when the firmware says it is off. Applied to `dcn201_clk_mgr_construct()`:

```c
-    dce_clock_read_ss_info(clk_mgr);
+    /* BC-250: get_ss_info_v4_2() reports the shared GPU-clock spread spectrum
+     * on the DisplayPort query, which desyncs DP audio. Only read DP-ref SS
+     * when the VBIOS says DP spread spectrum is enabled. */
+    if (bp->integrated_info && bp->integrated_info->dp_ss_control)
+        dce_clock_read_ss_info(clk_mgr);
```

Building a patched amdgpu module follows the same flow as the [40 CU unlock](../system/40cu-unlock.md) build (kernel source, patch, compile, rebuild the initramfs). If you already rebuild the module for the 40 CU unlock, add this patch in the same pass. This patch has not been tested on our reference board yet, so treat the exact build steps as reported rather than verified here.

!!!warning "Reverts on kernel update"
    Like the 40 CU unlock, a patched out-of-tree module is reverted by every kernel update. You rebuild after each update, or pin your kernel.

---

## Credit

Root cause analysis and the DCN201 patch are community work from the BC-250 Discord (Trov, with cross-checking from essdee and mzk10). The code path above was re-confirmed against the mainline kernel source.
