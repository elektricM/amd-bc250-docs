# Contributing to BC250 Documentation

Solutions posted only in Discord get lost. Adding them here makes them searchable, versioned, and easier for the next person hitting the same problem to find.

## Quick Start (No Git Knowledge Required)

### Method 1: Edit Directly on GitHub (Easiest)

1. **Browse to the page you want to edit** on GitHub: https://github.com/elektricM/amd-bc250-docs/tree/main/docs
2. **Click the pencil icon** (✏️) in the top right
3. **Make your changes** using the editor
4. **Scroll down** and click "Propose changes"
5. **Click "Create pull request"**

Done! We'll review and merge your contribution.

### Method 2: Report Issues

Found incorrect information or missing content?

1. Go to https://github.com/elektricM/amd-bc250-docs/issues
2. Click "New Issue"
3. Describe what's wrong or what's missing
4. Submit

We'll fix it or add the missing information.

## What's Most Useful

Anything that helps the next person fix a problem faster:

- Tested hardware (power supplies, cooling, displays, adapters)
- Game compatibility (FPS, settings, issues, workarounds)
- Troubleshooting solutions you've actually used
- Distribution-specific setup steps
- BIOS settings for specific use cases
- Benchmark results with your configuration
- Typo fixes, clearer explanations, missing commands
- Screenshots, diagrams, dmesg or sensor captures

### Please don't

- Add unverified claims or speculation. If you didn't test it, say "reportedly" and cite the source.
- Copy proprietary BIOS files or copyrighted content.
- Mix unrelated changes in one PR.

## Documentation Structure

```
docs/
├── getting-started/     # First-time setup
├── hardware/           # Specs, power, cooling, displays
├── bios/              # BIOS flashing, VRAM, overclocking
├── linux/             # Distribution setup guides
├── drivers/           # RADV, environment variables
├── system/            # Governor, sensors, power management
├── gaming/            # Game compatibility
├── troubleshooting/   # Common problems and solutions
└── reference/         # Quick references
```

## For Advanced Contributors (Using Git)

### Setup Local Environment

```bash
# Clone the repository
git clone https://github.com/elektricM/amd-bc250-docs.git
cd amd-bc250-docs

# Install dependencies
pip install mkdocs-material

# Preview your changes locally
mkdocs serve
# Open http://127.0.0.1:8000/ in your browser
```

### Making Changes

```bash
# Create a new branch
git checkout -b fix-something

# Edit files in docs/ directory
# Preview changes with: mkdocs serve

# Stage your changes
git add docs/

# Commit with a clear message (see Pull Request Guidelines below for the prefix)
git commit -m "fix: correct Fedora setup instructions for Mesa 25.1"

# Push to your fork
git push origin fix-something

# Create pull request on GitHub
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

### Writing Guidelines

**Style:**
- Use clear, concise language
- Write for beginners but include advanced details
- Use code blocks for commands
- Add warnings for risky operations

**Markdown Format:**
```markdown
# Page Title

Brief introduction explaining what this page covers.

## Section Heading

Content here.

### Subsection

More specific content.

!!!warning "Important Warning"
    Use admonitions for critical information.

!!!tip "Pro Tip"
    Helpful hints for users.

```bash
# Code blocks with syntax highlighting
sudo dnf install mesa-vulkan-drivers
```

**Links:**
```markdown
[Relative link to another page](../bios/flashing.md)
[External link](https://example.com/)
```
```

**Testing:**
- Test all commands you document
- Verify links work
- Check formatting with `mkdocs serve`

## Contribution Examples

### Example 1: Adding Game Compatibility

Edit `docs/gaming/compatibility.md`:

```markdown
### Cyberpunk 2077

**Status:** ✅ Playable
**Performance:** 45-60 FPS @ 1080p Medium
**VRAM:** 512MB dynamic or 8GB/8GB split
**Settings:**
- RT: Off (too demanding)
- FSR: Quality mode
- RADV_DEBUG=nocompute %command%

**Issues:**
- Occasional stuttering in crowded areas
- Fixed with kernel 6.13+
```

### Example 2: Adding Troubleshooting Solution

Edit `docs/troubleshooting/boot.md`:

```markdown
### Black Screen After Kernel Update

**Symptoms:**
- System boots but no display output
- Fans running normally

**Solution:**
1. Boot into recovery kernel from GRUB
2. Downgrade to previous kernel version:
   ```bash
   sudo dnf downgrade kernel
   ```
3. Report kernel version that failed on GitHub Issues
```

### Example 3: Adding Distribution Guide

Create new file `docs/linux/gentoo.md`:

```markdown
# Gentoo Setup

Step-by-step guide for setting up BC250 on Gentoo.

## Prerequisites
- Gentoo installation media
- Basic Gentoo knowledge

## Installation
[...]
```

Then add to `mkdocs.yml`:
```yaml
- Linux Setup:
    - Gentoo Setup: linux/gentoo.md
```

## Review Process

1. Submit your PR
2. Automated checks verify the site builds
3. Maintainer review for accuracy and clarity
4. Feedback if changes are needed
5. Merge, automatic publish to https://elektricM.github.io/amd-bc250-docs/

## Questions?

- **Documentation issues:** Open an issue on GitHub
- **General BC250 discussion:** Join Discord (but please document solutions here!)
- **Pull request help:** Ask in the PR comments

## License

By contributing, you agree your contributions will be licensed under CC BY-SA 4.0 for documentation and MIT for code examples.
