# Contributing to BC250 Documentation

**Help us improve the BC250 documentation!** Instead of posting solutions in Discord where they get lost, add them here so everyone can find them.

## Why Contribute?

- **Searchable:** Documentation is indexed by search engines
- **Organized:** Information is categorized and easy to find
- **Permanent:** Won't get lost in chat history
- **Collaborative:** Everyone benefits from improvements

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

## What to Contribute

### High Priority

- **Tested hardware configurations** (power supplies, cooling solutions, displays)
- **Game compatibility** (FPS, settings, issues, workarounds)
- **Troubleshooting solutions** (especially if you solved something not documented)
- **Distribution-specific setup steps** (if you got a distro working)
- **BIOS settings** that work well
- **Benchmark results** with your configuration

### Also Welcome

- **Fixing typos or unclear explanations**
- **Adding missing commands or steps**
- **Updating outdated information**
- **Adding screenshots or diagrams**
- **Improving organization**

### Don't Contribute

- ❌ Speculation or unverified claims
- ❌ Proprietary/copyrighted BIOS files
- ❌ Instructions for illegal activities
- ❌ Personal opinions without technical basis

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

# Commit with clear message
git commit -m "Fix Fedora setup instructions for Mesa 25.1"

# Push to your fork
git push origin fix-something

# Create pull request on GitHub
```

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

1. **Submit pull request** (PR)
2. **Automated checks** verify documentation builds
3. **Maintainers review** for accuracy and clarity
4. **Feedback/changes** may be requested
5. **Merged** once approved
6. **Published** automatically to https://elektricM.github.io/amd-bc250-docs/

## Questions?

- **Documentation issues:** Open an issue on GitHub
- **General BC250 discussion:** Join Discord (but please document solutions here!)
- **Pull request help:** Ask in the PR comments

## License

By contributing, you agree your contributions will be licensed under:
- **Documentation:** CC BY-SA 4.0
- **Code examples:** MIT

---

**Thank you for helping make BC250 documentation better for everyone!**

Instead of answering the same questions in Discord, let's build a knowledge base that helps everyone.
