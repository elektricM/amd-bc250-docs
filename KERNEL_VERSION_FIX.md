# Documentation Updates Log

## November 21-22, 2025 - Community Corrections Applied

### Critical Corrections from Discord (DeathStalker, Astrocast)

#### Kernel Version Information (FIXED)
**Issue:** Documentation incorrectly stated "AVOID 6.15+" entirely

**Correct Information:**
- **6.15.0 - 6.15.6**: BROKEN (GPU initialization fails)
- **6.15.7 - 6.17.7**: WORKS (kernel support fixed)
- **6.17.8+**: BROKEN AGAIN (GPU driver issues return)
- **6.12 - 6.14 LTS**: Works (stable fallback)

**Recommended:** 6.15.7 - 6.17.7 for best compatibility

**Files Updated (15):** All kernel warnings across docs/linux/, docs/troubleshooting/, docs/getting-started/, docs/index.md, docs/reference/

---

#### Safety-Critical PSU Corrections (FIXED)

**Removed Unsafe PSU Recommendations:**
1. **Dell D220P-01 (220W)** - Too low power, reported to "cut out or break"
2. **Dell D250AD-00 (250W)** - Too low power, insufficient for peak loads
3. **LED PSUs** - Unreliable ripple current, too hit or miss to recommend

**Power Requirements Updated:**
- Minimum PSU: 300W on 12V rail (was 250W)
- Minimum power draw: 70W (not 50W - never observed that low)
- Maximum power: Can exceed 235W with GPU frequency patch

**Cable Safety:**
- **16 AWG minimum** required (18 AWG has caused melted cables!)
- Added safety warnings about proper wire gauge

---

#### Hardware Corrections (FIXED)

**PSU Model Numbers:**
- FSP500-30AS (not FSP500-50FGBBI)

**Board Dimensions:**
- 340mm / 310mm (depending on measurement method)
- Was incorrectly listed as ~200mm

**Cooling:**
- Arctic P12 Pro added (more readily available than Max)
- Corrected static pressure: 6.9 mm H2O (both Pro and Max)
- Added separate specs for each model

**Power Button:**
- Board has NO power button header
- Must solder to existing onboard button for external switch

**GPU Comparisons:**
- Standardized to GTX 1660 Ti (removed inconsistent 3060 Ti references)

---

#### VRAM Allocation Clarification (FIXED)

**512MB Setting Explained:**
- 512MB is DYNAMIC allocation, NOT a limit
- Can allocate nearly full 16GB to VRAM when needed (~14GB+ for GPU)
- Fixed confusing language suggesting 512MB was maximum

---

#### Software Updates (FIXED)

**Fedora 42/43:**
- nomodeset NO LONGER NEEDED with working kernels (6.15.7-6.17.7)
- Still provide fallback for black screen issues

**Governor Installation:**
- COPR repos available for Fedora/Bazzite (no compilation needed)
- Emphasized pre-built packages available

**Distribution Support:**
- Added SteamOS (Mesa now sufficiently updated)
- Added Ubuntu (works fine with Mesa PPA)

**Arch Linux Installation:**
- Removed archinstall recommendations
- Direct users to official Arch Wiki
- Better for understanding and troubleshooting

---

#### Documentation Cleanup (FIXED)

**Removed:**
- Energy cost calculations (as requested)
- Multi-board setup sections (unnecessary complexity)
- Outdated/incorrect recommendations

---

## Summary of Changes

**3 Major Commits:**
1. Kernel version corrections (15 files)
2. Community hardware corrections (10 files)
3. Critical safety corrections (6 files)

**Total Files Modified:** 25+ files

**Safety Impact:**
- Prevented potential fire hazards (cable gauge, PSU power)
- Removed recommendations for PSUs that fail under load
- Corrected power requirements to prevent system instability

**Credits:**
- Astrocast: Kernel version specifics (6.15.7, 6.17.8)
- DeathStalker: Hardware corrections, PSU safety, cable gauge, power measurements
- legodude: Initial positive feedback

## Files to Update
All files containing "6.15+" warnings:
- docs/linux/kernel.md
- docs/troubleshooting/boot.md
- docs/troubleshooting/stability.md
- docs/troubleshooting/performance.md
- docs/troubleshooting/display.md
- docs/linux/distributions.md
- docs/linux/fedora.md
- docs/linux/bazzite.md
- docs/linux/cachyos.md
- docs/linux/arch.md
- docs/linux/debian.md
- docs/getting-started/prerequisites.md
- docs/getting-started/quick-start.md
- docs/index.md
- docs/reference/quick-reference.md
- docs/gaming/compatibility.md
- docs/bios/overclocking.md

## New Warning Format

```markdown
!!!danger "Kernel Compatibility"
    - **AVOID:** 6.15.0-6.15.6 and 6.17.8+ (GPU initialization fails)
    - **RECOMMENDED:** 6.15.7+, 6.16.x, 6.17.0-6.17.7
    - **SAFE FALLBACK:** 6.12-6.14 LTS
```

## Credits
- DeathStalker: Identified 6.15.11 works, 6.17.8 broke again
- Astrocast: Confirmed 6.15.7 was the fix, 6.17.8 broke it
