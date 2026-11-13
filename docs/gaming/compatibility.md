# Game Compatibility and Performance

Community-tested games with performance data and known issues for the BC-250.

## Performance Overview

The BC-250 delivers solid 1080p gaming performance, comparable to an RX 6600 or GTX 1660 Ti.

**Typical Performance:**
- **1080p High Settings:** 60-100+ FPS in most titles
- **1080p Medium Settings:** 80-120+ FPS
- **1440p:** Playable in many titles with FSR
- **Ray Tracing:** Entry-level performance (30-60 FPS with compromises)

## Tested AAA Games

### Cyberpunk 2077

**Performance:**
- **1080p High + FSR:** 70-90 FPS (no RT)
- **1080p High + FSR + RT (lighting only):** 50-60 FPS
- **1080p Ultra + FSR3.1:** 100+ FPS

**Settings Recommendations:**
- Enable FSR 2.0/3.1 Quality mode
- Ray tracing: Lighting only (disable RT shadows)
- DLSS/FSR Frame Generation works well

**Known Issues:**
- Maximum power draw game (up to 235W)
- Requires good cooling

**Benchmarks:**
- Stock (2000 MHz @ 1000mV): 57.66 FPS
- OC (2230 MHz @ 1035mV): 60.82 FPS
- With `mitigations=off`: +18 FPS boost

### The Last of Us Part I

**Performance:**
- **1080p Medium-High:** 60 FPS (stable)

**Settings Recommendations:**
- Medium-High settings for 60 FPS
- FSR helps maintain frame rate

**Known Issues:**
- Heat: 90-100°C during shader compilation
- Some clicking in audio reported

### Control

**Performance:**
- **1080p + RT:** 40 FPS

**Settings Recommendations:**
- Ray tracing works but demanding
- Lower RT quality for better FPS

**Notes:**
- Good RT performance for entry-level RT hardware

### Detroit: Become Human

**Performance:**
- **1080p Medium:** 60 FPS (capped)

**Settings Recommendations:**
- Medium settings for best frame latency
- Caps at 60 FPS for cinematic feel

**Notes:**
- Runs smoothly, no issues reported

### Devil May Cry 5

**Performance:**
- **1080p High:** 100 FPS

**Settings Recommendations:**
- High settings easily achievable
- Lowest frame latency (10ms) of tested games

**Notes:**
- Excellent optimization, runs great

### Company of Heroes 3

**Performance:**
- **Playable**

**Settings Recommendations:**
- Use 4GB VRAM split (512MB causes artifacts/crashes)

**Notes:**
- VRAM-sensitive game, needs adequate allocation

### Red Dead Redemption 2

**Performance:**
- **Benchmark:** 45+ FPS minimum

**Settings Recommendations:**
- Use `-useMaximumSettings` launch flag
- May detect as software rendering - change adapter in graphics settings to match `vulkaninfo --summary` output

**Known Issues:**
- Can detect wrong graphics adapter

## Popular Games

### Fortnite

!!! failure "Does Not Work"
    Fortnite uses Easy Anti-Cheat which does not support Linux. **Cannot run on BC-250.**

### Apex Legends

**Status:** Not tested extensively
**Expected:** Good performance expected

### Valorant

**Status:** Anti-cheat may have issues on Linux
**Expected:** Technical challenges

### CS2 (Counter-Strike 2)

**Status:** Works well
**Expected:** 100+ FPS at 1080p

### Rocket League

**Status:** Works well
**Expected:** 120+ FPS at 1080p

### Elden Ring

**Status:** Playable
**Expected:** 60 FPS with some settings adjustments

## Emulation

### Ryujinx (Nintendo Switch)

**Game:** The Legend of Zelda: Tears of the Kingdom

**Performance:**
- **Consistent:** 20 FPS across multiple boards and distros
- **Appears to be board limitation** for this specific game
- Other users report 60+ FPS with specific configurations

**Notes:**
- Performance varies significantly by game
- Some games run at 60 FPS
- TOTK specifically seems problematic

### Other Emulators

**Status:** Generally good
**PCSX2 (PS2):** Excellent
**RPCS3 (PS3):** Good for lighter titles
**Dolphin (GameCube/Wii):** Excellent

## Ray Tracing Performance

### Games with RT Tested

| Game | Resolution | FPS | Notes |
|------|------------|-----|-------|
| Cyberpunk 2077 | 1080p | 50-60 | RT lighting only, FSR quality |
| Control | 1080p | 40 | Full RT |
| Portal 2 RTX | 720p | 40 | Software RT in Mesa 25.2+ |
| Half-Life 2 RTX | 720p | 20-30 | Very demanding |

!!!info "Real Hardware RT"
    BC-250 uses real RDNA 2 hardware RT, not software emulation (with Mesa 25.2+).

## FSR (FidelityFX Super Resolution)

**FSR 2.0/3.0 Support:** Excellent

**Performance Gains:**
- FSR Quality: +20-30% FPS
- FSR Balanced: +30-40% FPS
- FSR Performance: +40-60% FPS

**FSR Frame Generation:**
- Works in supported games
- Can double frame rate
- Adds slight latency

**FSR 4 with Optiscaler:**
- Community reports Balance mode better than FSR 3.1.5 Quality
- Worth testing in supported games

## Known Game Issues

### Magic: The Gathering Arena

**Issue:** Crashes/freezes specifically on Fedora
**Workaround:** Works better on Manjaro or Bazzite
**Possible Fix:** Try different Proton versions

### Final Fantasy VII Rebirth

**Issue:** "DX12 is not supported on your system"
**Cause:** Game checks for specific GPU compatibility
**Status:** No fix for BC-250 yet
**Workaround:** None currently

### Black Myth Wukong

**Issue:** Cleaned files version - "CreateProcess() returned 2" error
**Cause:** Anti-tamper detection
**Workaround:** Use unmodified game files

### Star Wars Zero Company

**Issue:** Intermittent GPU page faults, usually about 50 s after launch. The game drops to `CrashReportClient`, or the whole board hangs and needs a power cycle (hard hangs may leave no fault lines in the journal). Check `journalctl -k -b` (or `-b -1` after a hang) for an amdgpu `[gfxhub] page fault` with:

- `Process GameThread`, thread `vkd3d_queue`
- `Faulty UTCL2 client ID: TCP (0x8)`
- `GCVM_L2_PROTECTION_FAULT_STATUS: 0x00641041` (or `0x00741041`), i.e. a write to a read-only page

The Proton log from a faulting run contains `VK_ERROR_DEVICE_LOST`.

**Cause (A/B tested, see source below):** vkd3d-proton descriptor-table change. Proton Experimental `experimental-11.0-20261105c` ships vkd3d-proton build [563eb6ca](https://github.com/HansKristian-Work/vkd3d-proton/commit/563eb6caaa8fb1386cbc629e7527c6522f5bfde8), which includes [03a508fb](https://github.com/HansKristian-Work/vkd3d-proton/commit/03a508fb) ("Clear descriptor table offsets to 0 on heap rebind"). Upstream made that behavior opt-in in [1a46e2f7](https://github.com/HansKristian-Work/vkd3d-proton/commit/1a46e2f7).
**Status:** Upstream vkd3d-proton master no longer faulted in testing; waiting for Proton Experimental to ship it. Proton 11.0 didn't fault in 3 runs, but the title screen ignored input.
**Workaround:** Run the game on a copy of Proton Experimental with the vkd3d-proton [v3.1](https://github.com/HansKristian-Work/vkd3d-proton/releases/tag/v3.1) DLLs swapped in.

Check whether your Proton Experimental includes the upstream change (`0` = not yet, `1` = included; this only identifies the build):
```bash
grep -c clear_table_offsets_on_rebind \
  ~/.local/share/Steam/steamapps/common/"Proton - Experimental"/files/lib/wine/vkd3d-proton/x86_64-windows/d3d12core.dll
```

Create the custom compatibility tool:
```bash
T=~/.local/share/Steam/compatibilitytools.d/ExpV31
mkdir -p "${T%/*}"
cp -a --reflink=auto ~/.local/share/Steam/steamapps/common/"Proton - Experimental" "$T"  # independent copy
cd /tmp
curl -sSLO https://github.com/HansKristian-Work/vkd3d-proton/releases/download/v3.1/vkd3d-proton-3.1.tar.zst
tar --zstd -xf vkd3d-proton-3.1.tar.zst
for a in x64:x86_64 x86:i386; do
  for f in d3d12.dll d3d12core.dll; do
    cp -f "vkd3d-proton-3.1/${a%:*}/$f" "$T/files/lib/wine/vkd3d-proton/${a#*:}-windows/"
  done
done
echo "1 ExpV31" > "$T/version"
cat > "$T/compatibilitytool.vdf" <<'VDF'
"compatibilitytools"
{
  "compat_tools"
  {
    "ExpV31"
    {
      "install_path" "."
      "display_name" "Proton Experimental (vkd3d 3.1)"
      "from_oslist" "windows"
      "to_oslist" "linux"
    }
  }
}
VDF
```
Restart Steam, then pick **Proton Experimental (vkd3d 3.1)** in the game's Properties → Compatibility. When the `grep` check prints `1` after an update, test stock Proton Experimental again before removing the workaround.

!!!warning "Hard hangs"
    If the fault hangs the whole board instead of crashing the game, see [Black Screen on GPU Reset](../troubleshooting/stability.md#black-screen-on-gpu-reset-governor-running).

*Source: own testing, Nov 2026. BC-250 on Bazzite (kernel 7.2.15, Mesa 26.2.6, cyan-skillfish-governor-smu, 8-core/40-CU unlocked). Stock Proton Experimental: 11 launches (some with diagnostic env vars) gave 5 GPU faults, 1 hard hang, 1 early exit and 4 runs still alive after 150–430 s. ExpV31 had 5/5 clean 5-minute runs. vkd3d-proton master (2576c0b4) was also 5/5 clean, and 2/2 hung with `VKD3D_CONFIG=clear_table_offsets_on_rebind`, which re-enables the regressed behavior.*

## Proton Compatibility

**Recommended Proton Versions:**
- **Proton Experimental:** Latest features, some instability
- **Proton GE:** Community-maintained, better compatibility
- **Proton 8.0/9.0:** Stable versions

**Per-Game Proton Selection:**
- Some games work better with specific Proton versions
- Test different versions if game doesn't work

**Install Proton GE:**
```bash
# Install ProtonUp-Qt
sudo dnf install protonup-qt  # Fedora
sudo pacman -S protonup-qt    # Arch

# Launch and install Proton-GE
protonup-qt
```

## Performance Optimization

### Steam Launch Options

**MangoHud (FPS overlay):**
```
mangohud %command%
```

**Force RADV and fix glitches:**
```
RADV_DEBUG=nohiz %command%
```

**GameMode (CPU optimization):**
```
gamemoderun %command%
```

**Combined:**
```
RADV_DEBUG=nohiz mangohud gamemoderun %command%
```

### In-Game Settings

**Graphics Priority:**
1. **Resolution:** 1080p native or with FSR
2. **Texture Quality:** High (plenty of VRAM)
3. **Shadows:** Medium-High
4. **Effects:** Medium
5. **Post-Processing:** Medium
6. **Ray Tracing:** Selective (lighting only, or off)

**V-Sync:**
- Disable for lowest latency
- Enable if screen tearing bothers you
- Use FreeSync/G-Sync if monitor supports it

### System Tweaks

**CPU Governor:**
```bash
# Set to performance mode for gaming
sudo cpupower frequency-set -g performance
```

**Disable Compositing (X11):**
- Reduces latency
- Automatic in most games with fullscreen

## VRAM Requirements by Game

| VRAM Usage | Games | Recommended Split |
|------------|-------|-------------------|
| < 2GB | Esports, indie games | 512MB |
| 2-4GB | Most games | 4GB |
| 4-6GB | AAA titles | 4GB + mem params |
| 6GB+ | Highest settings, RT | 4GB + mem params |

**Memory Parameters for More VRAM:**
```bash
# Add to kernel parameters (see Kernel guide)
amdgpu.gttsize=14750 ttm.pages_limit=3959290 ttm.page_pool_size=3959290
```

## Benchmark Scores

### Unigine Superposition (1080p Extreme)

| Configuration | Score | GPU Temp |
|---------------|-------|----------|
| Stock (2000 MHz, 1000mV) | 3888 | 76°C |
| Patched (2230 MHz, 1035mV) | 4118 | 86°C |

### Furmark (Stress Test)

**Warning:** Unrealistic load, not representative of gaming

| Configuration | Power Draw |
|---------------|------------|
| Stock (2000 MHz, 1000mV) | 250W |
| OC (2230 MHz, 1085mV) | 320W |

!!!danger "Furmark Power Draw"
    Furmark draws significantly more power than any game. Use game benchmarks for realistic testing.

## Game-Specific Tweaks

### Improve Stuttering

**Check shader caching:**
- Steam pre-compiles shaders
- Wait for shader compilation before playing
- Can cause initial stuttering

**Increase shader cache size:**
```bash
# Add to /etc/environment
__GL_SHADER_DISK_CACHE_SIZE=10737418240  # 10GB
```

### Fix Audio Clicking

**Some games have audio issues:**
- Try different Proton versions
- Use Proton-GE
- Check game-specific ProtonDB reports

## Resources

### ProtonDB

Check game compatibility: [protondb.com](https://www.protondb.com)

- Community reports on Linux compatibility
- Specific BC-250 performance may vary
- Look for similar AMD GPU reports

### Are We Anti-Cheat Yet

Check anti-cheat compatibility: [areweanticheatyet.com](https://areweanticheatyet.com)

- Some games with kernel-level anti-cheat don't work on Linux
- Situation improving over time

## Performance Expectations

### vs. Desktop GPUs

**Approximate Equivalent:**
- Rasterization: Between RX 6600 and RX 6600 XT
- Ray Tracing: Similar to RX 6600
- Memory: 16GB total (split), vs 8GB dedicated

## Tips for Best Experience

1. **Use FSR:** Free performance boost in supported games
2. **Install Governor:** Essential for dynamic frequency scaling
3. **Update Mesa:** Always use Mesa 25.1.3+ for best compatibility
4. **Try Proton-GE:** Better compatibility than stock Proton
5. **Monitor Temps:** Keep GPU < 85°C for stability
6. **Adequate VRAM:** Use 4GB split for AAA games
7. **Disable Mitigations:** `mitigations=off` for +10-15% FPS
8. **Optimize Kernel:** Use 6.18.18 LTS (recommended), 6.19.x stable, or 6.17.11+, avoid 6.15.0-6.15.6 and 6.17.8-6.17.10

## See Also

- [System Configuration](../system/governor.md)
- [BIOS VRAM Configuration](../bios/vram.md)
- [Linux Distribution Recommendations](../linux/distributions.md)
- [Performance Troubleshooting](../troubleshooting/display.md)
