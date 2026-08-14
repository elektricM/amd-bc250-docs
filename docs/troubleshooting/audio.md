# DisplayPort Audio: Silence, Desync or Slow Pitch

Audio over the BC-250's DisplayPort output can be completely silent, or play slightly slow (late, at a low pitch, with periodic crackle) while video stays perfect. Both are the same bug: the board firmware programs the DisplayPort audio clock for a reference clock the hardware does not have. This page explains the mechanism, how to measure it in two minutes, and a fix that needs no kernel build.

This is also the real story behind ["active DP-HDMI adapters break audio"](display.md#special-case-active-vs-passive-adapters). The adapters were never at fault. Passive adapters route audio through a different, unaffected clock path, which is why they seem immune (details below).

---

## Symptoms

- No audio at all. Most often through an active DP-HDMI adapter or a strict TV/AVR chain that refuses to lock to the off-spec stream
- Audio plays but runs ~17.65% slow: out of sync (drifting further behind the longer it plays), pitch slightly low, periodic crackle. A 30.00 s file takes 36.46 s
- Video playback stutter that looks like a network or GPU problem. PipeWire clocks its whole graph off the DisplayPort sink, so a sink draining at ~39.5 kHz stalls video that is synced to audio
- Everything *looks* healthy: `eld_valid 1`, `hw_params` reports `rate: 48000`, PipeWire shows the sink, xrun counters stay at zero

Whether you get silence or slow audio depends on the sink. Some displays play the off-spec stream honestly (slow), others never lock (silent). Same register, same fix.

---

## Root cause

The audio digital timing oscillator (DTO) for the DisplayPort path divides the DP reference clock down to 24 MHz using a phase/module register pair. The firmware programs the module value for a reference clock of 728.631 MHz, but the actual DPREFCLK on the BC-250 is 600.000 MHz:

| Register (DCN 2.0.1) | dword index | Value | Meaning |
|---|---|---|---|
| `DCCG_AUDIO_DTO_SOURCE` | `0x16B` | `0x00100010` | `DTO_SEL`=1 → DTO1, the DisplayPort path |
| `DCCG_AUDIO_DTO1_PHASE` | `0x16E` | `240000` | 24.000 MHz target |
| `DCCG_AUDIO_DTO1_MODULE` | `0x16F` | **`7286310`** | assumes reference = **728.631 MHz** |
| `CLK4_CLK2_CURRENT_CNT` | `0x1B27F` | **`6000`** | actual reference = **600.000 MHz** |

The audio codec therefore gets `600.000 MHz × 240000/7286310 = 19.763 MHz` instead of 24.000 MHz, and every sample rate is scaled by `600000/728631 = 0.82346`:

```text
48000 Hz × 0.82346 = 39526 Hz   predicted
                     39525-39526 Hz   measured, on two different boards
```

The factor is constant and proportional: identical across 32k/44.1k/48k/96k/192k and across every display mode. The wrong value is written by the **board firmware**, not the driver, and it is restored on **every modeset** (resolution change, game launch, fullscreen toggle). Confirmed independently on two boards by register read plus measuring the real ALSA consumption rate (recipe below). Writing the correct value restores audio instantly, verified at 3840x2160@60 and 1920x1080@120.

!!!note "This replaces the earlier spread-spectrum explanation"
    This page previously attributed the desync to `dcn201_clk_mgr_construct()` reading GPU-clock spread spectrum into the DP reference clock. That code path exists, but it is not what causes this: patching the shipped `amdgpu` so `dce_clock_read_ss_info()` is never called changes nothing (39526.1 Hz before and after), the spread-spectrum arithmetic predicts a value 0.19% *low* (5988750) where the measured value is 21.4% *high* (7286310), and every driver-computed module value is a multiple of 1000, which 7286310 is not. See [issue #39](https://github.com/elektricM/amd-bc250-docs/issues/39) for the full falsification. Consequently **no kernel version fixes this**; the earlier "fixed in 6.19.10+" note was wrong.

### Why passive adapters seem fine and active adapters seem broken

`decide_signal_from_strap_and_dongle_type()` in the driver (`dc/link/link_detection.c`) treats a passive DP++ dongle as an HDMI sink: the signal becomes `SIGNAL_TYPE_HDMI_TYPE_A` and audio is clocked from **DTO0**, derived from the pixel clock, which this bug cannot touch. An active adapter is a real DisplayPort sink, so audio comes from the broken **DTO1** path, same as a native DP monitor.

So "passive works, active doesn't" is not an adapter quality problem. It was proven directly: with an active adapter silent, disconnecting the sound bar and switching the TV to its own speakers changed nothing, and one register write brought audio back instantly through the same active adapter, with nothing else touched.

---

## Diagnosis

Measure the real sample consumption rate. No root, no tools:

```bash
# Find the running playback substream while audio "plays" (sound card index varies per boot):
for s in /proc/asound/card*/pcm*p/sub*/status; do grep -q RUNNING "$s" && echo "$s"; done

# Read hw_ptr from that file twice, ~10s apart. Real rate = (hw_ptr2 - hw_ptr1) / seconds.
# Bug present:  ~39525 Hz while hw_params says rate: 48000
# Bug absent:   ~48000 Hz
```

`aplay -l`, the ELD, `hw_params`, the PipeWire sink list and xrun counters all read identical whether audio works or not, so don't spend time on them:

- `hw_params` reports the *configured* rate, not the clock actually ticking.
- `eld connection_type: DisplayPort` proves nothing; the kernel fills it from the connector type alone.
- xruns stay at 0 because the sink isn't underrunning. The *source* is overrunning, and PipeWire silently discards the surplus samples; `pw-top` sits at `ERR 0` indefinitely while the problem continues.

To confirm at the register level (root required), read the four dword indices from the table above via `/sys/kernel/debug/dri/<n>/amdgpu_regs`, or with umr.

!!!warning "umr pitfalls on this ASIC"
    If you use umr on the BC-250: the display IP block is named `dcn203`, **not** `dcn201`, and an explicit `dcn201` register path fails. Only the `DCCG_AUDIO_DTO*` registers read back truthfully; umr's `DIG*`/`DP*` names map to wrong offsets on this ASIC and report "audio muted / no packets" even while audio plays perfectly. And `umr --write` interprets the value as **hex**, so writing decimal `6000000` actually writes `0x6000000` = 100663296. Never pass a `*` wildcard path to a write.

---

## Fix (no kernel build)

Writing the correct module value takes effect immediately, even on a running stream. The correct value is derived from the hardware itself: `CLK4_CLK2_CURRENT_CNT × 1000` (6000 → 6000000 on this board). Because every modeset restores the wrong value, and Steam Game Mode changes resolution on every game launch, the write has to be re-applied by a small watcher service:

```python
#!/usr/bin/python3
import glob, os, time
IDX_SRC, IDX_PHASE, IDX_MOD, IDX_CLK = 0x16B, 0x16E, 0x16F, 0x1B27F
fd = os.open(glob.glob('/sys/kernel/debug/dri/*/amdgpu_regs')[0], os.O_RDWR)
rd = lambda i: int.from_bytes(os.pread(fd, 4, i * 4), 'little')
while True:
    src, phase, mod, clk = rd(IDX_SRC), rd(IDX_PHASE), rd(IDX_MOD), rd(IDX_CLK)
    want = clk * 1000                  # counter is 100 kHz units, module wants kHz*10
    if (src >> 4) & 3 == 1 and phase == 240000 and 4_000_000 <= want <= 12_000_000 \
            and mod != want:
        os.pwrite(fd, want.to_bytes(4, 'little'), IDX_MOD * 4)
    time.sleep(1)
```

The guard only acts when DTO1 (the DisplayPort path) is selected at the expected 24 MHz phase, so HDMI-TMDS via a passive adapter is deliberately left alone, and the service is safe to leave enabled across display swaps. Run it as a systemd service (`Type=simple`, `WantedBy=multi-user.target`).

Two service-setup mistakes that cost real debugging time:

- Do **not** combine `After=graphical.target` with `WantedBy=multi-user.target`. That is an ordering cycle, and systemd silently deletes the service's start job at boot. It is invisible when you test with a manual `systemctl start`.
- On Fedora Atomic / Bazzite, install the script to `/usr/local/bin` and point the unit there. SELinux denies systemd (`init_t`) execute on files in `/home` (`user_home_t`), producing a `203/EXEC` crash loop.

A maintained version of this watcher, with `status`, `apply`, `revert`, `watch` and `install-service` commands and both pitfalls above handled, is available as `bc250-audio-dto-fix.sh` in [bc250-tools](https://github.com/Weijtmans/bc250-tools).

!!!note "Up to one second of wrong audio after a modeset"
    The firmware rewrites the register on each modeset and the watcher corrects it on its next pass. With a 1 s interval that means up to a second of silent or slow audio after a resolution change or game launch, then it recovers on its own.

If you cannot or don't want to run the watcher: a passive DP-to-HDMI adapter (DTO0 path) or a USB DAC sidesteps the bug entirely.

---

## Pitfalls when testing

- **Check the default sink before concluding the fix failed.** If audio was broken for a while, WirePlumber may have saved *Dummy Output* (`auto_null`) as the preferred sink in `~/.local/state/wireplumber/default-nodes`, and audio then stays silent even with the clock fixed. `wpctl status`, then `wpctl set-default` onto the DisplayPort sink. (`speaker-test -D hw:X,Y` bypasses PipeWire, which makes it a good isolation tool.)
- **Unplugging the HDMI cable at the TV end of an active adapter does not force a modeset.** The adapter is the DP sink and holds the link up regardless of its downstream side. To force a real modeset, change the resolution GPU-side or unplug at the board end.
- **To test whether an event restores the wrong value, write the correct value first, then trigger the event.** Triggering a modeset while the wrong value is already present overwrites wrong with wrong and looks like "no effect".

---

## Credit

Root cause identification, the spread-spectrum falsification and the watcher approach: **Fleischfrau** in [issue #39](https://github.com/elektricM/amd-bc250-docs/issues/39) (LG TV, native DisplayPort). Independent confirmation on a second board, the active-adapter exoneration, the DTO0/DTO1 passive-vs-active analysis and the umr/service pitfalls: **Elgar Weijtmans** (@Weijtmans), verified at 4K60 and 1080p120 through a UGREEN 8K active DP-HDMI adapter (Realtek RTD2173) → Samsung TV → HDMI-ARC → Sonos Beam, CEC intact, on Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29. The original DCN201 spread-spectrum source analysis this page previously carried was community work by Trov, essdee and mzk10 of the BC-250 Discord; the code path they described is real, it just turned out not to be what programs this register.
