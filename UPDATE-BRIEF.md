# Docs Update Brief — March 2026

## Sources of Truth (in priority order)
1. **mothenjoyer69/bc250-documentation** (GitHub) — the original community docs, maintained by the OG devs
2. **BC250 Discord resource channels** — where technical discoveries happen first
3. **r/BC250Gaming subreddit** — user-facing questions and builds

## Critical Issues to Fix

### 1. Community is calling the docs "AI generated outdated garbage"
Discord quotes (March 5-7):
- "really wouldn't use that guide it's not only outdated but it's AI generated outdated garbage" 
- "don't trust anything except the section on BIOS flashing"
- "I keep vacillating between making an MR to fix the whole dang thing and not wanting to waste my time on correcting AI slop"
- "Problem is that it's SEO'd better than actually truthful resources"

### 2. Governor Landscape (MAJOR UPDATE NEEDED)

**Current state (March 2026):**
- **cyan-skillfish-governor-tt** — filippor's thermal throttling version. DEFAULT for Bazzite/Fedora via COPR `filippor/bazzite`. Service name: `cyan-skillfish-governor-tt`
- **cyan-skillfish-governor-smu** (NEW Jan 18 2026) — filippor's SMU variant. **Bypasses kernel patches entirely.** AUR: `cyan-skillfish-governor-smu`, COPR: `filippor/bazzite`
- **oberon-governor** — original by mothenjoyer69. COPR: `@exotic-soc/oberon-governor`. Still works, considered legacy but stable
- **NexGen3D setup script** — `github.com/NexGen-3D-Printing/SteamMachine` — installs cyan-skillfish-governor-tt + zram + CPU mitigations. De facto standard for Bazzite beginners

**filippor service name change (Dec 13 2025):**
- Renamed from `cyan-skillfish-governor` to `cyan-skillfish-governor-tt`
- Config folder moved from `/etc/cyan-skillfish-governor/` to `/etc/cyan-skillfish-governor-tt/`
- Migration: `sudo cp /etc/cyan-skillfish-governor/config.toml /etc/cyan-skillfish-governor-tt/config.toml`

**Minimum voltage bug:** Setting min voltage below 700mV locks GPU to 1500MHz. Must be ≥700mV.

### 3. Bazzite Kernel is Pre-Patched
- The GPU frequency range kernel patch is **already included in Bazzite's kernel**
- Manual kernel patching instructions on the overclocking page are **unnecessary for Bazzite users**
- The SMU governor also bypasses the need for kernel patching on ANY distro
- For CachyOS/Arch, either patch kernel or use SMU governor

### 4. ACPI Fix is Essential (not optional)
- Required for C-State support
- Without it, no proper power management
- GitHub: `bc250-collective/bc250-acpi-fix`

### 5. Mesa 25.1 Upstream Support
- Mesa 25.1 has upstream cyan-skillfish support
- mothenjoyer says: "should be shipped by most big distros at this point"

### 6. mothenjoyer's docs say:
- VRAM allocation: set to **512MB** for best APU experience (not 4/12 split)
- J1000 is standard 8-pin PCIe power (already fixed in our docs)
- ttm.pages_limit=3959290 and ttm.page_pool_size=3959290 kernel params needed for >8GB shared memory
- nct6683 driver for sensors (force=true needed), NOT nct6686 — check what our docs say
- "These boards more or less just work now" — install distro, install governor, done
- Windows: "No" — GPU not supported by any drivers
- Don't use Smokeless_UMAF — may cause permanent damage
- HW encode/decode does NOT work (VCN firmware blocked by Sony)

### 7. WiFi Drivers Issue (#10)
- User reported: installing performance mode killed wifi drivers
- Likely related to kernel swap or driver module changes during advanced setup
- Need warning on relevant page

## Specific Factual Errors Called Out by Community
1. **PCIe is 2.0 x2, NOT 3.0** — sapoperro flagged this Nov 22. mothenjoyer confirms 2.0
2. **"980 MHz is unstable across all boards"** — filippor questioned this Nov 22. Likely AI hallucination. Remove or caveat
3. **Case links were broken** — some fixed, verify all remain working
4. **Kernel 6.17.8 has a BC250 GPU bug** — fixed in 6.18, NOT backported. Important for CachyOS/Arch users. See https://gitlab.freedesktop.org/drm/amd/-/issues/4721
5. **nct6683 vs nct6686** — mothenjoyer says nct6683 with force=true. Check what our docs say
6. **HW encode/decode does NOT work** — VCN firmware blocked by Sony. If docs suggest it works, fix it
7. **ttm kernel params** — ttm.pages_limit=3959290 and ttm.page_pool_size=3959290 needed for >8GB shared memory
8. **Smokeless_UMAF warning** — "Don't try it, you may cause permanent damage" per mothenjoyer

## Alternative Community Guides (reference, don't copy)
- **DeathStalker Grimoire**: https://github.com/DeathStalker471/bc250theGrimoire — community-approved step-by-step guide in grimdark fiction style. Covers heatsink mod, thermal paste, BIOS flash, governor setup
- **NexGen3D SteamMachine scripts**: https://github.com/NexGen-3D-Printing/SteamMachine — automated setup for Bazzite
- **filippor's instructions** (Dec 15 on Discord): `dnf copr enable filippor/bazzite` → `dnf install cyan-skillfish-governor-tt` → `systemctl status cyan-skillfish-governor-tt`

## What NOT to Change
- BIOS flashing section — was rewritten by deathstalkerjr, considered accurate by the community
- Site structure — don't reorganize
- Cases catalogue — recently updated (145 designs)

## Additional Community Criticism (from Discord flex-chat + help threads)

### Specific pages called out:
- **quick-start.md Step 5 "Remove nomodeset"** — user says editing grub directly is outdated on Bazzite, should use user.cfg instead (Jan 8, bearofberlin)
- **hardware/power.md** — "outdated infos saying vram can run very hot and needs cooling and power supply needs 8 pins because 6 pins not recommended" (Dec 17, akxfile)
- **hardware/cooling.md** — "ram cooling nonsense" flagged (Dec 17, akxfile)
- **bios/overclocking.md** — "AI generated outdated garbage", "don't bother with manual patching" (Mar 4-6)
- **linux/bazzite.md** — "has issues" (Nov 30, deathstalkerjr)
- **getting-started/quick-start.md** — user followed it, couldn't find pp_dpm_sclk (card0 vs card1), empty grub file (Feb 4, ruben2099)
- **General sentiment** (Feb 2, big_trov): "It did that because it is ai generated slop"
- **General sentiment** (Feb 10, pops1cl): "docs seem a little outdated"

### GPU governor crash bug (IMPORTANT for troubleshooting)
- User reports (Feb 26, nohanmv): cyan-skillfish-governor can cause black screen on GPU reset. If GPU crashes while governor is running, it can't reset properly → stuck black screen. Workaround: disable governor before playing crash-prone games. This NEEDS to be documented in troubleshooting.

## VRAM Allocation Controversy
- mothenjoyer says: 512MB best for general APU use
- akxfile (Feb 23): "idk why everyone say 512 is good for gaming" — needed 6-8GB to stop crashes in Expedition 33
- Community consensus seems to be: 512MB for most games, but some newer/heavier titles need 4-8GB
- Docs should present both options with use cases, not just one recommendation

## Overclocking Page Issues
- FPS numbers (Star Wars Battlefront 2: 80-85→120-130 FPS +50%) — looks AI-generated, unverifiable
- Presents kernel patch as essential when Bazzite already includes it
- Should lead with "install a governor" not "patch your kernel"
- SMU governor (Jan 2026) bypasses all kernel patching — this is the modern approach

## Quick-Start Grub Issue
- Page tells users to edit /etc/default/grub directly
- On Bazzite (immutable), this is wrong — should use user.cfg
- User bearofberlin (Jan 8) flagged this

## NexGen3D Setup Scripts (the "just works" approach)
- https://github.com/NexGen-3D-Printing/SteamMachine
- Setup-16GB.sh — GPU governor + zram + CPU mitigations
- Setup-CPU.sh — CPU overclock + ACPI fix
- SteamMachinePro/ — full pro setup guide
- Community adapts these for CachyOS too (amirseni, Mar 2)

## Cross-Reference Map
Discord channel → Doc page:
- `Bazzite OS for starters` → linux/bazzite.md
- `oberon-governor optimization` → system/governor.md  
- `Script to Install Governor and Fixes` → system/governor.md, linux/bazzite.md
- `CPU Overclocking & Undervolting Tools` → bios/overclocking.md
- `Increased GPU frequency range kernel patch` → linux/kernel.md
- `Making CachyOS work` → linux/cachyos.md
- `GPU Control for BC-250` → new tool, mention in governor.md
- `BIOS update/flasher program` → bios/flashing.md (DO NOT CHANGE - community approved)
- `zswap instead of zram` → could add to system/ 
- `Mem Timing Utility` → could add to bios/overclocking.md

Discord exports: ~/clawd/data/bc250-discord/ (35 resource channel files, ~8.8MB total)
