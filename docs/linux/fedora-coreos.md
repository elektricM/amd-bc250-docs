# Fedora CoreOS Setup

<img src="https://fedoraproject.org/assets/images/coreos-logo-light.png" alt="Fedora CoreOS Logo"/>

Fedora CoreOS Works great on the BC-250.  As a container optimized OS, use cockpit, podman, and quadlets to manage a single machine.  I use docker swarm to manage multiple nodes.

I won't go into [ignition setup](https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/), but I can at least guide post installation setup for the BC-250 and my performance customizations.

**Status:** Fully Working

**Difficulty:** Specialized Setup (its not a destkop, its a headless server platform)

**Operating System:** This is the base that Bazzite is built upon, so one can easily 'rebase' to Bazzite in the future.

**Kernel and Software Versions:** As of this writing, [6.19.14 is being used](https://fedoraproject.org/coreos/release-notes/?arch=x86_64&stream=stable).  Next version will use Kernel 7.0.8, more info.

---

## Why Choose Fedora CoreOS

### Advantages

- **Rolling release** - Rolling Releases and Managed Upgrades
- **Low RAM usage** - Less than 1GB typical for a freshly provisioned machine.
- **Immutable Base** - All the rpm-ostree advantages.
- **Build for Container Optimized Workloads** - Some of my uses are; Games on Whales, Frigate, Home Assistant, *Claw, and llama-server

---

## BIOS Requirements

**REQUIRED before installing:**
1. Flash modified BIOS (P3.00 recommended)
2. Set VRAM allocation (512MB dynamic recommended)
3. **Disable IOMMU** (IOMMU is broken - MUST disable)

See [BIOS Flashing Guide](../bios/flashing.md).

---

## Fedora CoreOS Installation

### Prerequisites

- USB drive (4GB+)
- Fedora CoreOS Live DVD ISO [fedoraproject.org](https://fedoraproject.org/coreos/download/)
- Familiarity with Linux command line

### Installation Steps

!!!info "Follow the Fedora CoreOS Installation Guides"
    Fedora CoreOS installation should be done following the official [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide).  There is no specific deviations or customizations needed to install CoreOS on a BC-250.  All customization is post-installation.

---

## Post Installation Steps

### Enable ACPI C-States and P-States

This will ensure the C-States and P-States settings will persist across CoreOS upgrades.  Run these as the root user.

1. Obtain the two .aml files from the [bc250-acpi-fix github repository](https://github.com/bc250-collective/bc250-acpi-fix).
``` 
git clone https://github.com/bc250-collective/bc250-acpi-fix.git
```
2. Make the dracut directory containing the AML files
```
mkdir -p /etc/dracut.conf.d/acpi/
```
3. copy the *.aml files into /etc/dracut.conf.d/acpi/
```
cp bc250-acpi-fix/*.aml /etc/dracut/conf.d/acpi/
```
4. create a new file /etc/dracut.conf.d/99-acpi-override.conf with the following content
```
acpi_override="yes"
acpi_table_dir="/etc/dracut.conf.d/acpi"
```
5.  Enable InitramFS Generation
```
rpm-ostree initramfs --enable
```
6. Reboot
```
systemctl reboot
```
7. Verify the ACPI Table Overrides are applied
```
root@localhost:~# dmesg | grep SSDT-.ST
[    0.005283] ACPI: SSDT ACPI table found in initrd [kernel/firmware/acpi/SSDT-CST.aml][0x30e]
[    0.005286] ACPI: SSDT ACPI table found in initrd [kernel/firmware/acpi/SSDT-PST.aml][0x39e]
```

### Enable Cyan Skillfish Governor

This will allow you to throttle the GPU.  350mhz <-> 2200mhz was tested on my setup.  Highly dependant on your cooling setup.
Beware of setting this above 2000 and enabling all GPU Compute Units.

1. Download and place the COPR filippo/bazzite repo into your local system.
   ```
   curl --output-dir /etc/yum.repos.d/ -O https://copr.fedorainfracloud.org/coprs/filippor/bazzite/repo/fedora-44/filippor-bazzite-fedora-44.repo
   ```
2. Install the cyan-skillfish-govenor-smu package
   ```
   rpm-ostree install cyan-skillfish-governor-smu
   ```
3. Reboot
   ```
   systemctl reboot
   ```
4. Edit the configuration to your preferences
   ```
   nano /etc/cyan-skillfish-governor-smu/config.toml 
   ```
5. Enable the cyan-skillfish-governor-smu service
   ```
   systemctl enable cyan-skillfish-governor-smu.service
   systemctl restart cyan-skillfish-governor-smu.service
   ```
6. Verify the service is running and working
   ```
   systemctl status cyan-skillfish-governor-smu.service
   ```
### Enable Hardware Monitor

This allows the read-only hardware monitor functions to work.

1. Create the configuration file to load the module.
```
echo 'nct6683' > /etc/modules-load.d/nct6683.conf
```
2. Create the configuration file for the options to load with the module.
```
echo 'options nct6683 force=1' > /etc/modprobe.d/nct6683.conf
```
3. Reboot
```
systemctl reboot
```
4. Verify the module was loaded
```
root@localhost:~# dmesg | grep nct6683
[    3.811815] nct6683: Found NCT6686D or compatible chip at 0x2e:0xa20
[    3.812657] nct6683 nct6683.2592: NCT6686D EC firmware version 1.0 build 07/28/21
```
### Enable Additional Core Unlock

As mentioned before **CARE** is needed when enabling the 40 Compute Units and cooling/power.  This *WILL* cause issues if you're unprepared.  It is recommeneded you keep the govenor between 350mhz and 1500mhz if you enable this.  This will need to be loaded once per reboot.

1. Install the AMD User Mode Register (UMR) Utility
```
rpm-ostree install umr
```
2. Reboot
```
systemctl reboot
```
3. Create a new file for systemd with the following contents
```
cat <<EOF > /etc/systemd/system/gpu-unlock.service
[Unit]
Description=Unlock GPU Compute Units at Boot
After=multi-user.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStartPre=/usr/bin/sleep 10s
ExecStart=/usr/bin/umr -w *.gfx1013.mmRLC_PG_ALWAYS_ON_WGP_MASK 0x1f
ExecStart=/usr/bin/umr -w *.gfx1013.mmCC_GC_SHADER_ARRAY_CONFIG 0x0
ExecStart=/usr/bin/umr -w *.gfx1013.mmCC_GC_SHADER_ARRAY_CONFIG 0x0 -b 1 0 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmCC_GC_SHADER_ARRAY_CONFIG 0x0 -b 1 1 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmCC_GC_SHADER_ARRAY_CONFIG 0x0 -b 0 1 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmCC_GC_SHADER_ARRAY_CONFIG 0x0 -b 0 0 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmSPI_PG_ENABLE_STATIC_WGP_MASK 0x1f -b 0 0 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmSPI_PG_ENABLE_STATIC_WGP_MASK 0x1f -b 0 1 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmSPI_PG_ENABLE_STATIC_WGP_MASK 0x1f -b 1 0 0xffffffff
ExecStart=/usr/bin/umr -w *.gfx1013.mmSPI_PG_ENABLE_STATIC_WGP_MASK 0x1f -b 1 1 0xffffffff

[Install]
WantedBy=multi-user.target
EOF
```
4. Register, Enable and Start the Service
```
systemctl daemon-reload
systemctl enable gpu-unlock.service
systemctl restart gpu-unlock.service
```
### Enable 14.75GB Allocation of VRAM

This will allow you to allocate 14.75GB of RAM to the GPU for services like LLMs.

1. Create the modprobe config with the following
```
cat <<EOF > /etc/modprobe.d/bc250-vram.conf
options ttm pages_limit=3959290 page_pool_size=3959290
options amdgpu gttsize=14750
EOF
```
2. Reboot
```
systemctl reboot
```

### Turn off Spectre Mitigations

This allows for some CPU speed-ups.

1.  Configure kernel commandline settings
```
rpm-ostree kargs --replace=mitigiatons=auto,nosmc=off
```
2. reboot
```
systemctl reboot
```

---

## Podman Expose GPU to Containers

I often expose the gpu for compute uses (e.g. llms) leveraging vulkan.  In order to do so, in your podman command line use the following
```
--device /dev/dri --device /dev/kfd
```

**Related Guides:**
- [Fedora Setup](fedora.md)
- [Bazzite Setup](bazzite.md)
- [CachyOS Setup](cachyos.md)
- [GPU Governor](../system/governor.md)
