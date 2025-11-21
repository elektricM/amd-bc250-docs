# How to Contribute

**Help us make BC250 documentation better!** Instead of posting solutions only in Discord, add them here where everyone can find them.

## Why Contribute to the Docs?

| Discord Posts | Documentation |
|--------------|---------------|
| Gets buried in chat history | Searchable forever |
| Hard to find later | Organized by topic |
| Same questions repeated | One permanent answer |
| Limited formatting | Code blocks, images, tables |
| No version control | Track changes, revert mistakes |

## Easy Ways to Contribute

### 1. Edit This Page (Easiest!)

1. Click the **edit icon** (✏️) in the top right of any page
2. Make your changes in the GitHub editor
3. Click "Propose changes" → "Create pull request"
4. Done! We'll review and merge it

**No Git knowledge required!**

### 2. Report Issues

Found incorrect or missing information?

**[Open an issue →](https://github.com/elektricM/amd-bc250-docs/issues)**

Tell us:
- What's wrong or missing
- What should be there instead
- Any relevant links or sources

### 3. Share Your Configuration

Got BC250 working well? Share your setup:

**Hardware:**
- What PSU works for you?
- Which cooler keeps temps good?
- Display adapter compatibility?

**Software:**
- Your distro setup steps
- Kernel version that works
- BIOS settings you use
- Games you've tested

**Add it to the relevant page!**

## What We Need Most

### 🔥 High Priority

- **Game compatibility reports** (FPS, settings, issues)
- **Troubleshooting solutions** you've discovered
- **Hardware testing** (PSUs, coolers, displays that work)
- **Distribution guides** for distros not yet documented
- **BIOS settings** that work well for different use cases

### ✅ Also Welcome

- Fixing typos and unclear explanations
- Adding missing commands or configuration steps
- Updating outdated information
- Screenshots or diagrams
- Benchmark results

### ❌ Please Don't

- Post speculation without testing
- Copy proprietary content
- Add unverified claims
- Include personal opinions without technical basis

## Contribution Examples

### Example: Add Game Compatibility

Edit `docs/gaming/compatibility.md`:

```markdown
### Elden Ring

**Status:** ✅ Playable
**Performance:** 55-60 FPS @ 1080p High
**VRAM:** 512MB dynamic recommended
**Settings:**
- Graphics: High
- Anti-aliasing: Medium
- RADV_DEBUG=nocompute %command%

**Notes:**
- Stuttering in some areas with 8GB/8GB split
- Smooth with 512MB dynamic VRAM
```

### Example: Add Troubleshooting Solution

Edit `docs/troubleshooting/boot.md`:

```markdown
### Black Screen After Mesa Update

**Symptoms:**
- System boots but no display
- SSH still works

**Solution:**
Downgrade Mesa to last working version:

```bash
sudo dnf downgrade mesa-vulkan-drivers mesa-dri-drivers
```

Then report the issue to Mesa GitLab.
```

### Example: Add Hardware Compatibility

Edit `docs/hardware/power.md`:

```markdown
### Tested PSUs

**Corsair SF450 (450W SFX)**
- ✅ Works perfectly
- Tested with RTX 3060 Ti (TDP mod)
- No stability issues
- Tested by: @username
```

## Advanced: Local Development

Want to preview changes locally?

```bash
# Clone repository
git clone https://github.com/elektricM/amd-bc250-docs.git
cd amd-bc250-docs

# Install MkDocs
pip install mkdocs-material

# Preview locally
mkdocs serve
# Open http://127.0.0.1:8000/
```

**Full guide:** [CONTRIBUTING.md on GitHub](https://github.com/elektricM/amd-bc250-docs/blob/main/CONTRIBUTING.md)

## Documentation Structure

Know where to add your contribution:

```
docs/
├── getting-started/     # First-time setup, prerequisites
├── hardware/           # Specs, power, cooling, displays
├── bios/              # BIOS flashing, VRAM, overclocking
├── linux/             # Distribution-specific guides
├── drivers/           # RADV driver, environment variables
├── system/            # GPU governor, sensors, power
├── gaming/            # Game compatibility, performance
├── troubleshooting/   # Common problems and solutions
└── reference/         # Quick references, cheatsheets
```

## Review Process

1. **Submit** your contribution
2. **Automated checks** verify it builds correctly
3. **Review** by maintainers (usually within 48 hours)
4. **Feedback** if changes needed
5. **Merged** and automatically published
6. **Live** at https://elektricM.github.io/amd-bc250-docs/

## Questions?

- **How do I...?** Read [CONTRIBUTING.md](https://github.com/elektricM/amd-bc250-docs/blob/main/CONTRIBUTING.md)
- **Found a bug?** [Open an issue](https://github.com/elektricM/amd-bc250-docs/issues)
- **Need help?** Ask in Discord, but add the answer here when solved!

---

## Thank You!

Every contribution helps someone avoid hours of troubleshooting.

**Instead of answering the same questions in Discord, let's build a knowledge base that helps everyone.**

[Start Contributing →](https://github.com/elektricM/amd-bc250-docs)
