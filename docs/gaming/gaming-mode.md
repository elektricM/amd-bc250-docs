# Gaming Mode Setup

Run a Steam Deck-style gaming mode on your BC-250 using [gamescope-session-plus](https://github.com/elektricM/bc250-gaming-mode). This installs a "Steam Big Picture" Wayland session you can pick from your login screen (SDDM/GDM), giving you the full Deck UI — resolution switching, MangoHud overlay, VRR, and FSR/NIS upscaling.

It works alongside your desktop — KDE Plasma, GNOME, whatever. Just pick which session you want at login.

!!! info "Tested Configuration"
    Fedora 43 + KDE Plasma + SDDM + BC-250. Other setups may work but haven't been verified.

## What You Get

- Full Steam Deck UI (Big Picture mode as the session)
- Gamescope compositor with resolution switching
- MangoHud performance overlay
- VRR (Variable Refresh Rate) support
- FSR/NIS upscaling built in
- Switch between gaming mode and desktop at the login screen

## Installation

The setup is packaged in a repo that handles everything:

```bash
git clone https://github.com/elektricM/bc250-gaming-mode.git
cd bc250-gaming-mode
```

Follow the README in the repo for the install steps — it's based on ChimeraOS's gamescope-session and adapted for the BC-250.

After installation, log out. On the SDDM/GDM login screen, look for a session called "Steam Big Picture" in the session picker (usually bottom-left corner on SDDM). Select it and log in.

## BC-250 Specific Notes

The BC-250 has some quirks you should know about:

**GPU device:** The GPU shows up as `card1`, not `card0`. The gamescope session config accounts for this, but if you're troubleshooting, keep it in mind.

**Display output:** The BC-250 outputs on `DP-1`. If gamescope can't find your display, check that your output matches:

```bash
# Verify your output name
cat /sys/class/drm/card1-DP-1/status
```

**GPU governor:** The [GPU governor service](../system/governor.md) keeps running in gaming mode. This is fine — it handles frequency scaling the same way whether you're in desktop or gaming mode.

**Audio:** Audio over DisplayPort works normally in gaming mode. No extra setup needed.

## Optional: Decky Loader

[Decky Loader](https://decky.xyz/) lets you install Steam Deck plugins in gaming mode. Things like PowerTools, ProtonDB Badges, CSS Loader, etc.

!!! tip
    Decky was designed for the Steam Deck but works in any gamescope session. Most plugins work, but hardware-specific ones (like fan control) won't do anything useful on the BC-250.

## Switching Sessions

To go back to your desktop:

1. In gaming mode, press the **Steam** button
2. Go to **Power** → **Switch to Desktop**
3. This logs you out and takes you back to SDDM/GDM
4. Pick your desktop session (KDE Plasma, GNOME, etc.) and log in

To go back to gaming mode, just select "Steam Big Picture" at login again.

## Troubleshooting

**Black screen after selecting gaming mode session:**

- Check that Steam is installed and has been launched at least once from the desktop
- Verify gamescope is installed: `which gamescope`
- Check the journal for errors: `journalctl --user -u gamescope-session -b`

**Wrong resolution or display not detected:**

- Make sure you're on `DP-1` — the BC-250 doesn't use HDMI for gaming mode
- Check that your monitor is connected before logging in

**Performance feels worse than desktop:**

- Gamescope adds a compositor layer, but it shouldn't hurt performance in practice
- Make sure the GPU governor is running: `systemctl status cyan-skillfish-governor-tt`
- Check MangoHud overlay (++f12++ by default) to verify clocks and temps

## See Also

- [Game Compatibility](compatibility.md)
- [GPU Governor](../system/governor.md)
- [Performance Troubleshooting](../troubleshooting/performance.md)
