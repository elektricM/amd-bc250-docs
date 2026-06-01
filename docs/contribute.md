# How to Contribute

Solutions posted only in Discord get lost. Adding them here makes them searchable, versioned, and easier for the next person hitting the same problem to find.

## Easy Ways to Contribute

### 1. Edit on GitHub (no Git knowledge needed)

1. Click the **edit icon** (✏️) in the top right of any page
2. Make your changes in the GitHub editor
3. Click "Propose changes" → "Create pull request"

We'll review and merge it.

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

## What's Most Useful

Anything that helps the next person fix a problem faster:

- Game compatibility reports (FPS, settings, known issues)
- Troubleshooting solutions you've actually used
- Hardware that works (PSUs, coolers, displays, adapters)
- Distribution guides for distros not yet documented
- BIOS settings for specific use cases
- Benchmark results with your configuration
- Typo fixes, clearer explanations, missing commands
- Screenshots, diagrams, dmesg or sensor captures

### Please don't

- Add unverified claims or speculation. If you didn't test it, say "reportedly" and cite the source.
- Copy proprietary BIOS files or copyrighted content.
- Mix unrelated changes in one PR.

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

## Pull Request Guidelines

A few conventions that keep the docs reviewable and the commit history readable.

### Commit messages

Short single-line subject prefixed by type, blank line, then a body explaining what changed and why.

Allowed prefixes, lowercase, no subscope:

- `docs:` documentation add or edit
- `fix:` correcting an error or outdated information
- `feat:` new site feature (theme override, plugin, nav change)
- `chore:` refactor, dependency bumps, non-functional mkdocs.yml cleanup

Examples:

- `docs: add Alpine Linux setup guide`
- `fix: correct VRAM pages_limit value in Fedora CoreOS guide`
- `chore: bump mkdocs-material to 9.5`

### One change per PR

Keep each PR focused on one logical change. Typo fix and new distro guide should be two PRs. Mixed PRs are slower to review and harder to revert.

### Hardware-verified claims

If you add a command, parameter, kernel option, or BIOS setting, run it on a BC-250 first and confirm the outcome. Real output (sensors, dmesg, benchmark scores) is better than paraphrase.

### Cite the source

For non-obvious claims (performance numbers, kernel-version requirements, "stable up to X MHz"), say where the information came from: your own bench, a Discord thread, an upstream kernel commit. "Reportedly" is fine for unconfirmed claims as long as it's flagged.

### Cross-reference, don't duplicate

If a step is already documented elsewhere (governor setup, BIOS flashing, kernel choice), link to that page instead of copying the steps. Keeps the site maintainable.

### PR title matches the commit subject

Use the same line for the PR title and the commit subject. If the PR has multiple commits and we squash-merge, that subject becomes the merged commit.

## Review Process

1. Submit your PR
2. Automated checks verify the site builds
3. Maintainer review (usually within 48 hours)
4. Feedback if changes are needed
5. Merge, automatic publish to https://elektricM.github.io/amd-bc250-docs/

## Questions?

- **How do I...?** Read [CONTRIBUTING.md](https://github.com/elektricM/amd-bc250-docs/blob/main/CONTRIBUTING.md)
- **Found a bug?** [Open an issue](https://github.com/elektricM/amd-bc250-docs/issues)
- **Need help?** Ask in Discord, but add the answer here when solved!

---

[Start contributing →](https://github.com/elektricM/amd-bc250-docs)
