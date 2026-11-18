# DisplayPort Audio: Silence, Desync or Slow Pitch

Audio over the BC-250's DisplayPort output, whether to a DP monitor or through an active DP-to-HDMI adapter, can be completely silent, play about 18% slow, or slowly drift out of sync with the picture, while video stays perfect. The cause is two bugs in the Linux kernel's display driver for this chip, and both are fixed upstream. **Updating the kernel fixes it: 7.2 or newer (or 7.1.10 or newer) has both fixes, and current longterm kernels have the one that matters most.** This page shows which kernels and distributions are affected, how to check your own board in two minutes, and a workaround for systems that cannot update yet.

This is also the real story behind ["active DP-HDMI adapters break audio"](display.md#special-case-active-vs-passive-adapters). The adapters were never at fault. Passive adapters route audio through a different clock path that neither bug touches, which is why they seem immune (details below).

!!!note "Earlier versions of this page were wrong"
    In August and September 2026 this page blamed the board firmware and said no kernel version fixes the bug. Both statements were wrong, and the second one told people not to update. The measurements behind them were taken on a 6.17 kernel, where the part of the driver being tested never runs (bug 1 below). See [issue #39](https://github.com/elektricM/amd-bc250-docs/issues/39) for the correction and the retraction.

---

## Symptoms

- No audio at all. Most often through an active DP-HDMI adapter or a strict TV/AVR chain that refuses to lock to the off-spec stream
- Audio plays but runs about 17.7% slow: pitch low, periodic crackle, and drifting further behind the picture the longer it plays. A 30.00 s file takes 36.46 s
- Video playback stutter that looks like a network or GPU problem. PipeWire clocks its whole graph off the DisplayPort sink, so a sink draining at ~39.5 kHz stalls video that is synced to audio
- On kernels that have only the first fix, a much milder version: audio sounds right but drifts out of sync with the picture by about 7 seconds per hour, noticeable a few minutes into a film
- Everything *looks* healthy: `eld_valid 1`, `hw_params` reports `rate: 48000`, PipeWire shows the sink, xrun counters stay at zero

Whether you get silence or slow audio depends on the sink. Some displays play the off-spec stream honestly (slow), others never lock (silent). Same cause, same fix.

---

## Which kernels are affected

Check your kernel with `uname -r` and find it below. Checked against the kernel.org stable changelogs on 27 September 2026:

| Kernel | DP audio | Fixes present |
|---|---|---|
| 7.2 and newer, 7.1.10 to 7.1.13 | Correct | Both |
| 6.12.78 and newer, 6.18.20 and newer (the longterm branches), 6.19.10 to 6.19.14, all of 7.0, 7.1.0 to 7.1.9 | Small drift, about 0.19% (7 s per hour) | Bug 1 fixed, bug 2 not |
| Everything older: 6.13 to 6.17, 6.19 before 6.19.10, 6.18 before 6.18.20, 6.12 before 6.12.78, and every 6.6 and 6.1 release | About 17.7% slow, or silent | Neither |

The longterm branches may still pick up the second fix; if they do, it will appear in their changelog as "drm/amd: Disable DP audio spread spectrum for Cyan Skillfish".

What common systems ship at the time of writing (distribution kernels move every week, so the version from `uname -r` is what counts):

| System | Kernel | DP audio |
|---|---|---|
| Bazzite stable, 44.20260820 (20 August 2026) and newer | 7.2 (7.2.4-ogc in 44.20260921) | Correct |
| Fedora 43 and 44 | 7.2.7 | Correct |
| Arch `linux` / `linux-lts` | 7.2.7 / 6.18.54 | Correct / small drift |
| CachyOS `linux-cachyos` / `linux-cachyos-lts` | 7.2.8 / 6.18.52 | Correct / small drift |
| Debian 13 (trixie) / trixie-backports | 6.12.107 / 7.1.8 | Small drift |
| Debian testing and unstable | 7.2 | Correct |
| Ubuntu 26.04 LTS | 7.0.0 | Small drift |
| SteamOS 3.8.28 stable (25 September 2026) | 6.18.50 | Small drift |
| SteamOS stable before 3.8.28 | 6.16 or older | About 17.7% slow, or silent |

Older Bazzite images carried older kernels (the 43.x images had 6.17.7-ba, with the large error), and so can patched images built on an older Bazzite base. If you rebased onto a community image, check `uname -r` rather than the image name.

### The fix: update the kernel

Update the system the normal way for your distribution, reboot, and check `uname -r` against the first table. On a current Bazzite or Fedora that is all it takes. On a longterm kernel (Debian stable, Ubuntu 26.04, the `-lts` packages) the large error is already gone and only the small drift is left; if it bothers you, switch to a 7.2 kernel where your distribution offers one, or use the [workaround](#workaround-for-older-kernels) below.

If audio is still silent after the update, check the default sink before anything else (first item under [Pitfalls](#pitfalls-when-testing)). If you installed the watcher service from an earlier version of this page, it finds nothing to correct on a fixed kernel and can be removed.

---

## Root cause: two kernel bugs

The DisplayPort audio clock is made by dividing the DisplayPort reference clock (DPREFCLK, 600 MHz on this board) down to 24 MHz with a pair of registers, `DCCG_AUDIO_DTO1_PHASE` and `DCCG_AUDIO_DTO1_MODULE`. The driver writes the module value from what it believes the reference clock is, and two separate bugs made it believe the wrong thing.

**Bug 1: the wrong clock manager (the large error).** When the display driver starts, it picks a clock manager for the chip. A revision check meant for another GPU (Beige Goby) also matched Cyan Skillfish and was tested first, so the kernel ran the DCN 3.0 clock manager instead of the DCN 2.0.1 one written for this chip. That code starts from a fixed 730 MHz reference, and after the adjustment described under bug 2 it programs a module value of 7286310 (728.631 MHz) where 6000000 is correct. Every sample rate is scaled by 600/728.631 = 0.8235, so 48 kHz plays at about 39525 Hz, by the same factor at every sample rate and display mode. Fixed by [`drm/amd: fix dcn 2.01 check`](https://github.com/torvalds/linux/commit/39f44f54afa58661ecae9c27e15f5dbce2372892) (Andy Nguyen), released in 7.0 and backported to 6.19.10, 6.18.20 and 6.12.78.

**Bug 2: spread-spectrum correction (the small error).** The VBIOS says the DisplayPort reference clock runs with a spread-spectrum downspread, so the driver lowers its idea of the reference by about 0.19%. The clock on this board does not actually run with one. With bug 1 fixed, the module value comes out just under 6000000 (5988740 in the upstream report) and audio drifts against the picture by about 7 seconds per hour. Fixed by [`drm/amd: Disable DP audio spread spectrum for Cyan Skillfish`](https://github.com/torvalds/linux/commit/ff209cd04845d819acc2fcc19b25904b4b7c3ea9) (Travis K. Bangs), released in 7.2 and backported to 7.1.10.

The driver writes the module value again on every modeset (resolution change, game launch, fullscreen toggle), which is why the workaround below has to keep re-applying it.

The confirmation comes from one board (LG TV over native DisplayPort), read and measured over a 95 s window before and after a Bazzite update by @Fleischfrau in [#39](https://github.com/elektricM/amd-bc250-docs/issues/39):

| Register (DCN 2.0.1) | dword index | 6.17.7-ba29 (Bazzite 43.20260420) | 7.2.0-ogc4.1 (Bazzite 44.20260820) |
|---|---|---|---|
| `DCCG_AUDIO_DTO_SOURCE` | `0x16B` | `DTO_SEL`=1 → DTO1, the DisplayPort path | same |
| `DCCG_AUDIO_DTO1_PHASE` | `0x16E` | `240000` (24.000 MHz target) | `240000` |
| `DCCG_AUDIO_DTO1_MODULE` | `0x16F` | **`7286310`** (assumes 728.631 MHz) | **`6000000`** (600.000 MHz) |
| `CLK4_CLK2_CURRENT_CNT` | `0x1B27F` | `6000` (actual reference 600.000 MHz) | `6000` |
| Measured rate at 48 kHz | | 39525.79 Hz (−17.655%) | 47999.76 Hz (−0.0005%) |

### Why passive adapters seem fine and active adapters seem broken

`decide_signal_from_strap_and_dongle_type()` in the driver (`dc/link/link_detection.c`) treats a passive DP++ dongle as an HDMI sink: the signal becomes `SIGNAL_TYPE_HDMI_TYPE_A` and audio is clocked from **DTO0**, derived from the pixel clock, which neither bug touches. An active adapter is a real DisplayPort sink, so audio comes from the affected **DTO1** path, same as a native DP monitor.

So "passive works, active doesn't" is not an adapter quality problem. It was proven directly: with an active adapter silent, disconnecting the sound bar and switching the TV to its own speakers changed nothing, and one register write brought audio back instantly through the same active adapter, with nothing else touched.

---

## Diagnosis

Measure the real sample consumption rate. No root, no tools:

```bash
# Find the running playback substream while audio "plays" (sound card index varies per boot):
for s in /proc/asound/card*/pcm*p/sub*/status; do grep -q RUNNING "$s" && echo "$s"; done

# Read hw_ptr from that file twice, a known time apart. Real rate = (hw_ptr2 - hw_ptr1) / seconds.
# Bug 1 (large error):  ~39525 Hz while hw_params says rate: 48000
# Bug 2 only:           off by about 90 Hz (0.19%); wait a minute between the reads to see it
# Fixed:                48000 Hz to within a few Hz
```

`aplay -l`, the ELD, `hw_params`, the PipeWire sink list and xrun counters all read identical whether audio works or not, so don't spend time on them:

- `hw_params` reports the *configured* rate, not the clock actually ticking.
- `eld connection_type: DisplayPort` proves nothing; the kernel fills it from the connector type alone.
- xruns stay at 0 because the sink isn't underrunning. The *source* is overrunning, and PipeWire silently discards the surplus samples; `pw-top` sits at `ERR 0` indefinitely while the problem continues.

To confirm at the register level (root required), read the four dword indices from the table above via `/sys/kernel/debug/dri/<n>/amdgpu_regs`, or with umr. `DCCG_AUDIO_DTO1_MODULE` reads `6000000` on a fixed kernel, `7286310` with bug 1, and just under `6000000` when only bug 2 is left.

!!!warning "umr pitfalls on this ASIC"
    If you use umr on the BC-250: the display IP block is named `dcn203`, **not** `dcn201`, and an explicit `dcn201` register path fails. Only the `DCCG_AUDIO_DTO*` registers read back truthfully; umr's `DIG*`/`DP*` names map to wrong offsets on this ASIC and report "audio muted / no packets" even while audio plays perfectly. And `umr --write` interprets the value as **hex**, so writing decimal `6000000` actually writes `0x6000000` = 100663296. Never pass a `*` wildcard path to a write.

---

## Workaround for older kernels

Only for systems that cannot run a fixed kernel yet, such as SteamOS stable before 3.8.28 or an image built on an old base, or a longterm kernel where the small drift bothers you. On a fixed kernel the watcher below never finds anything to correct.

Writing the correct module value takes effect immediately, even on a running stream. The correct value is derived from the hardware itself: `CLK4_CLK2_CURRENT_CNT × 1000` (6000 → 6000000 on this board), so the same script corrects both the large error and the small drift. Because every modeset restores the wrong value, and Steam Game Mode changes resolution on every game launch, the write has to be re-applied by a small watcher service:

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

The guard only acts when DTO1 (the DisplayPort path) is selected at the expected 24 MHz phase, so HDMI-TMDS via a passive adapter is deliberately left alone, and the service is safe to leave enabled across display swaps. Run it as a root systemd service (`Type=simple`, `WantedBy=multi-user.target`).

Two service-setup mistakes that cost real debugging time:

- Do **not** combine `After=graphical.target` with `WantedBy=multi-user.target`. That is an ordering cycle, and systemd silently deletes the service's start job at boot. It is invisible when you test with a manual `systemctl start`.
- On Fedora Atomic / Bazzite, install the script to `/usr/local/bin` and point the unit there. SELinux denies systemd (`init_t`) execute on files in `/home` (`user_home_t`), producing a `203/EXEC` crash loop.

A maintained version of this watcher, with `status`, `apply`, `revert`, `watch` and `install-service` commands and both pitfalls above handled, is available as `bc250-audio-dto-fix.sh` in [bc250-tools](https://github.com/Weijtmans/bc250-tools).

!!!note "Up to one second of wrong audio after a modeset"
    The driver rewrites the register on each modeset and the watcher corrects it on its next pass. With a 1 s interval that means up to a second of silent or slow audio after a resolution change or game launch, then it recovers on its own.

If you cannot or don't want to run the watcher: a passive DP-to-HDMI adapter (DTO0 path) or a USB DAC sidesteps the bug entirely.

---

## Pitfalls when testing

- **Check the default sink before concluding the fix failed.** If audio was broken for a while, WirePlumber may have saved *Dummy Output* (`auto_null`) as the preferred sink in `~/.local/state/wireplumber/default-nodes`, and audio then stays silent even with the clock fixed. `wpctl status`, then `wpctl set-default` onto the DisplayPort sink. (`speaker-test -D hw:X,Y` bypasses PipeWire, which makes it a good isolation tool.)
- **Unplugging the HDMI cable at the TV end of an active adapter does not force a modeset.** The adapter is the DP sink and holds the link up regardless of its downstream side. To force a real modeset, change the resolution GPU-side or unplug at the board end.
- **To test whether an event restores the wrong value, write the correct value first, then trigger the event.** Triggering a modeset while the wrong value is already present overwrites wrong with wrong and looks like "no effect".

---

## Credit

**@bangstk** (Trov on the BC-250 Discord) worked out that there are two separate bugs and which kernels have which, reported the spread-spectrum bug upstream and wrote the fix that was merged for 7.2. The spread-spectrum analysis this page first carried was their community work with essdee and mzk10, and it described bug 2 correctly. The clock-manager fix (bug 1) is by Andy Nguyen.

**@Fleischfrau** measured the error three independent ways, found that the xrun counters stay at zero through all of it, and wrote the watcher, in [issue #39](https://github.com/elektricM/amd-bc250-docs/issues/39). After @bangstk's correction they retracted their firmware explanation there and measured the same board before and after the update, which is the confirmation table above. The account has since been deleted, so their comments show as "ghost".

**@Weijtmans** confirmed the numbers on a second board before reading #39, found the variant where the bug shows up as total silence, proved the active adapter innocent, and contributed the DTO0/DTO1 passive-vs-active analysis and the umr and systemd pitfalls, on a UGREEN 8K active DP-HDMI adapter (Realtek RTD2173) → Samsung TV → HDMI-ARC → Sonos Beam, Bazzite (Fedora Atomic 43), kernel 6.17.7-ba29. They wrote the August rewrite of this page ([#43](https://github.com/elektricM/amd-bc250-docs/pull/43)) faithfully from the analysis available at the time; as @Fleischfrau said there, the error was in that analysis, not in the rewrite.

@parkj12b ([#56](https://github.com/elektricM/amd-bc250-docs/pull/56)) and @Noahnoah55 ([#39](https://github.com/elektricM/amd-bc250-docs/issues/39)) reported the bug gone after updating Bazzite.
