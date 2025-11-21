# AMD BC250 Documentation

Comprehensive community-driven documentation for the AMD BC250 board - a powerful ex-mining board featuring a PlayStation 5 APU.

## About the BC250

The AMD BC250 is a compact motherboard built around AMD's "Cyan Skillfish" APU (6x Zen 2 cores, 24 RDNA2 CUs, 16GB GDDR6) originally designed for cryptocurrency mining. The community has transformed this hardware into a capable Linux gaming and desktop system.

## Documentation

Visit the documentation site: [https://elektricM.github.io/amd-bc250-docs/](https://elektricM.github.io/amd-bc250-docs/)

## Key Features

- **Linux Setup Guides** - Step-by-step instructions for Fedora, Bazzite, Arch, CachyOS, and Debian
- **BIOS Configuration** - Flashing procedures, VRAM allocation, overclocking guides
- **Hardware Reference** - Specifications, power requirements, cooling solutions, pinouts
- **Troubleshooting** - Common issues and verified solutions from 1000+ community members
- **Gaming Compatibility** - 30+ tested games with performance data

## Project Structure

```
amd-bc250-docs/
├── docs/                   # MkDocs documentation source (20 pages)
├── parsed-content/         # Extracted Discord discussions (550KB, 13 files)
├── discord-export/         # Discord server exports and analysis
├── mkdocs.yml             # MkDocs configuration
└── scripts/               # Utility scripts
```

## Building Locally

```bash
# Install MkDocs and Material theme
pip install mkdocs-material

# Serve locally (with live reload)
mkdocs serve

# Build static site
mkdocs build
```

The site will be available at http://127.0.0.1:8000/

## Content Sources

This documentation is built from:

- **Primary:** BC250 Discord community (9,716 technical messages analyzed from 1000+ members)
- **Secondary:** [BC250 GitHub repository](https://github.com/mothenjoyer69/bc250-documentation)
- **Last Updated:** November 21, 2025

## Critical Requirements

- **Kernel:** 6.12.x - 6.14.x LTS (⚠️ AVOID 6.15+)
- **Mesa:** 25.1.3+ minimum, 25.1.5+ recommended
- **BIOS:** P3.00 with 512MB dynamic VRAM allocation
- **Governor:** Required for optimal GPU performance

## Contributing

This documentation is community-driven. If you find errors or have improvements:

1. Join the [BC250 Discord](https://discord.com/invite/uDvkhNpxRQ)
2. Submit issues or pull requests to this repository
3. Share your testing results and solutions

## Deployment

This site is automatically deployed to GitHub Pages via GitHub Actions on every push to the main branch.

## License

Documentation content: CC BY-SA 4.0
Code and configuration: MIT

---

**Built with:** [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
