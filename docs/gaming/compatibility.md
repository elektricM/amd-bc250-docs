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

**Status:** Not tested/reported
**Expected:** Should run well at 1080p

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
amdgpu.gttsize=14750 ttm.pages_limit=3776000
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

### vs. PlayStation 5

| Aspect | BC-250 | PS5 |
|--------|--------|-----|
| CPU Performance | 75% | 100% |
| GPU Performance | 67% | 100% |
| Real-World Gaming | 70-80% | 100% |

**Summary:** Expect slightly lower performance than PS5, but still very capable for 1080p gaming.

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
8. **Optimize Kernel:** Use 6.15.7-6.17.7 or 6.12-6.14 LTS, avoid 6.15.0-6.15.6 and 6.17.8+

## See Also

- [System Configuration](../system/governor.md)
- [BIOS VRAM Configuration](../bios/vram.md)
- [Linux Distribution Recommendations](../linux/distributions.md)
- [Performance Troubleshooting](../troubleshooting/display.md)
