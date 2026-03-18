# VRAM Configuration Guide

One of the BC-250's unique features is configurable memory allocation between CPU RAM and GPU VRAM. Understanding how to configure this properly is critical for optimal performance.

## Understanding UMA (Unified Memory Architecture)

The BC-250 uses **unified memory** - a single 16GB pool of GDDR6 RAM shared between CPU and GPU. The BIOS setting "UMA Frame Buffer Size" controls how this memory is divided.

!!!info "Key Concept"
    Unlike traditional systems with separate RAM and VRAM, the BC-250's memory is dynamically sharable (with 512MB setting) or statically partitioned (with fixed allocations).

---

## Configuration Options

### Option 1: 512MB Dynamic

**BIOS Setting:** UMA Frame Buffer Size = 512MB

**Important:** This "512MB" setting is NOT a limit - it enables **dynamic VRAM allocation** where the GPU can access nearly the full 16GB as needed.

**How it works:**
- System starts with ~15.5GB CPU RAM, ~512MB minimum GPU VRAM
- When GPU needs more VRAM, it automatically claims from system RAM
- When GPU load drops, memory returns to system pool
- Can allocate nearly full 16GB to VRAM when needed (up to ~14GB+ for GPU)

**Pros:**
- Most flexible
- Best for varied workloads
- No need to choose allocation manually

**Cons:**
- May conflict with ZRAM in some games (RDR2, Company of Heroes 3)
- Some games incorrectly report available VRAM
- Some titles (Expedition 33, Mafia) crash unless 4-8GB is statically allocated

**Best for:**
- General use, mixed gaming, productivity
- Users who don't want to tweak settings

### Option 2: Fixed 10GB RAM / 6GB VRAM

**BIOS Setting:** UMA Frame Buffer Size = 6144MB

Statically allocates 6GB to GPU, 10GB to CPU.

**Pros:**
- Fixes ZRAM conflicts
- More predictable performance
- Games properly detect VRAM amount
- Stable for AAA titles

**Cons:**
- Less flexible
- May waste VRAM if not fully used
- Can run out of system RAM in extreme cases

**Best for:**
- AAA gaming (RDR2, Cyberpunk, Control)
- Users experiencing crashes with 512MB dynamic
- Systems using ZRAM for swap

### Option 3: Fixed 8GB RAM / 8GB VRAM

**BIOS Setting:** UMA Frame Buffer Size = 8192MB

Balanced 50/50 split.

**Pros:**
- Balanced for most use cases
- Simple to reason about
- Good for compute workloads

**Cons:**
- May waste VRAM if unused
- Less system RAM than 512MB dynamic typically provides

**Best for:**
- AI/LLM inference
- Compute workloads needing large VRAM
- Users wanting simple balanced split

### Option 4: Fixed 12GB RAM / 4GB VRAM

**BIOS Setting:** UMA Frame Buffer Size = 4096MB

CPU-favoring split.

**Pros:**
- Maximum system RAM
- Low idle power (less VRAM to keep refreshed)
- Good for non-gaming use

**Cons:**
- Limited VRAM for modern games
- May struggle with high-res textures
- Not enough for 4K gaming

**Best for:**
- Light gaming (esports titles, older games)
- Desktop/productivity use
- Low-power optimization

---

## Changing VRAM Allocation

### In BIOS

1. Boot into BIOS (press **Del** during startup)
2. Navigate to **Chipset Configuration** or **Advanced** menu
3. Find **UMA Frame Buffer Size** setting
4. Select desired value:
   - 512MB (dynamic)
   - 4096MB (12GB/4GB)
   - 6144MB (10GB/6GB)
   - 8192MB (8GB/8GB)
5. Save and exit (F10)
6. System will reboot with new allocation

!!!warning "Takes Effect Immediately"
    The new allocation applies on reboot. No need to reflash BIOS or reinstall OS.

### Verification in Linux

Check current allocation:

```bash
# Check system RAM
free -h
# Should show ~10-15GB depending on allocation

# Check VRAM
cat /sys/class/drm/card0/device/mem_info_vram_total
# Shows GPU memory in bytes

# Check both
neofetch
# Or
inxi -Fxxxz
```

---

---

## Known Issues

### ZRAM Conflicts with 512MB Dynamic

**Symptoms:**
- RDR2 crashes when loading new areas
- Company of Heroes 3 artifacts then crashes
- Out-of-memory errors despite available RAM

**Cause:**
ZRAM compressed swap can confuse the dynamic allocator, causing memory management failures.

**Solutions:**
1. **Disable ZRAM:**
   ```bash
   sudo systemctl disable zram-swap
   sudo reboot
   ```

2. **Switch to fixed 10GB/6GB allocation** (better solution)

3. **Reduce ZRAM size:**
   ```bash
   # Edit /etc/systemd/zram-generator.conf
   [zram0]
   zram-size = 4096  # Reduce from default 8GB
   ```

### Games Misreporting VRAM

**Symptoms:**
- Game settings show wrong VRAM amount
- Ultra textures disabled despite having VRAM
- Performance warnings despite good performance

**Cause:**
Games query BIOS-reported VRAM (512MB or fixed amount) and don't understand dynamic allocation.

**Solution:**
Ignore the warning. Performance is what matters. The game will use what it needs.

**Workaround (if game refuses to run):**
Switch to fixed allocation that matches game's requirements.

### Vulkan vs OpenGL VRAM Reporting

**Issue:**
Vulkan sees full dynamic VRAM (~10-12GB), OpenGL only sees BIOS-allocated amount (512MB).

**Impact:**
- OpenGL games may refuse to run on "512MB VRAM"
- Vulkan/Proton games work fine

**Solution:**
Most modern games use Vulkan via Proton. If game needs OpenGL and complains, use fixed allocation.

---

## Advanced: Kernel Parameters for More VRAM

You can override VRAM limits via kernel parameters to access up to ~14.75GB VRAM.

!!!warning "Experimental"
    This is for advanced users doing AI inference or compute. Not needed for gaming.

Add to GRUB command line:

```bash
sudo nano /etc/default/grub

# Add to GRUB_CMDLINE_LINUX_DEFAULT:
amdgpu.gttsize=14750 ttm.pages_limit=3959290 ttm.page_pool_size=3959290

# Update GRUB
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot
```

**What this does:**
- `amdgpu.gttsize`: Sets GTT (Graphics Translation Table) size in MB
- `ttm`: Increases memory manager limits

**Usage:**
When running LLM inference or other compute tasks, limit memory allocation to avoid crashes:

```bash
llama.cpp --mem 14500  # Slightly less than 14.75GB max
```

---

---

## Testing Your Configuration

After changing allocation, verify it works:

```bash
# 1. Check allocation took effect
free -h
cat /sys/class/drm/card0/device/mem_info_vram_total

# 2. Run stress test
vkmark  # Vulkan benchmark
glmark2  # OpenGL benchmark

# 3. Test actual games
steam  # Launch and test a few titles

# 4. Monitor for crashes/OOM
journalctl -f  # Watch for memory errors
```

---

---

**Related Pages:**
- [BIOS Flashing Guide](flashing.md)
- [Governor Configuration](../system/governor.md)
- [Gaming Performance](../gaming/compatibility.md)
