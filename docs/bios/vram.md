# VRAM Configuration Guide

The BC-250 shares the same 16GB of memory between the CPU and GPU. This is called 
"Unified Memory Architecture" or UMA.

However, all PC software still expects CPU and GPU to have separate pools of memory, therefore
the system splits usage of the memory into RAM and VRAM.

---

# Changing the VRAM Split

The stock 3.00 BIOS splits the memory into 8GB RAM and 8GB VRAM.

Control of the RAM/VRAM split is ordinarily not exposed to the user.
However, there are ways to modify it.


## Memcfg Utility

You can set the VRAM split from a running Linux system by running a program: [fanoush/bc250_memcfg](https://github.com/fanoush/bc250_memcfg).

This is the preferred option because it does not involve reflashing BIOS, and therefore it is safer
for novice users.

```bash
git clone https://github.com/fanoush/bc250_memcfg
cd bc250_memcfg
make
sudo ./bc250memcfg UMA_SIZE 512
```
Replace '512' with the desired VRAM size in MB.
- 256 = 256 MB
- 512 = 512 MB
- 1024 = 1 GB
- 3072 = 3 GB
- 4096 = 4 GB
- 6144 = 6 GB
- 8192 = 8 GB
- 10240 = 10 GB
- 12288 = 12 GB
- **Note:** Linux won't boot with 2GB VRAM split for some reason. All other values seem OK.

Reboot to apply the change.

Even though this is a Linux program, the change is stored into CMOS, so it only needs to be ran 
once and not at every boot.

## A note on modified BIOS

You may have read about needing to flash the BIOS to unlock the VRAM split option in the BIOS
setup. This was previously required to get the option, but it is now obsolete.

Because of the above utility, there is no longer any need to flash BIOS to change VRAM settings.

---

# Dynamic VRAM

It is commonly stated that 512MB VRAM split is a "dynamic allocation" mode, where the system
will automatically reassign RAM to the VRAM pool as needed, but then will return that VRAM back to
RAM when the game is closed.

In truth, **ALL VRAM split settings are dynamic!** There is nothing special about the 512MB mode
except that it has a very small MINIMUM VRAM allocation. 

What the VRAM split setting above is actually doing is setting the MINIMUM amount of memory
that is used as VRAM. This means that with a 512MB VRAM split, when a game is not running, almost 
the whole 16GB of system memory is available as regular RAM, while with an 8GB split nearly half
of the system memory will never be usable as regular RAM. But, both modes will dynamically use more
than 512MB and 8GB of VRAM respectively.

So what is the maximum VRAM?

By default, Linux will use up to half of the RAM capacity (that is not being dedicated to VRAM)
as dynamic VRAM. This is used in addition to the VRAM split selected above. When this total is
exceeded, the display driver will crash.

With this knowledge, here is the default maximum VRAM usage before crash for each VRAM split:
- 512 MB split means 8.25 GB max VRAM
- 4 GB split means 10 GB max VRAM
- 6 GB split means 11 GB max VRAM
- 8 GB split means 12 GB max VRAM

## Solving Crashes with 512MB VRAM split

Because the default max VRAM usage with 512MB VRAM split is 8.25 GB, the system can tip over the
limit when the game is set up to use 8GB or more of VRAM. This is the primary reason for games
crashing when using the 512MB VRAM split, while working fine with an 8GB VRAM split.

To solve this while staying with the 512MB VRAM split, there is a Linux kernel boot parameter that
will override the dynamic VRAM limit with any amount.

- `ttm.pages_limit=value`
	- where `value` is replaced by a number using the following formula:
	- ttm.pages_limit = `(Desired max dynamic VRAM in GB) * 1024 * 1024 / 4`
	
For example, to extend the max dynamic VRAM to 12GB, use `ttm.pages_limit=3014656`.
This has the same maximum VRAM usage as the 8GB VRAM split, while returning nearly all of the VRAM
back to regular RAM when the game is closed.

### Applying the kernel parameter

Applying the kernel boot param differs depending on your distro:
- On Bazzite:
	- `sudo rpm-ostree kargs --delete=ttm.pages_limit --append=ttm.pages_limit=3014656 --reboot`
- On CachyOS:
	- CachyOS can have one of several different bootloaders depending on your installation settings.
		- See [CachyOS Boot Manager   Configuration](https://wiki.cachyos.org/configuration/boot_manager_configuration/#kernel-command-configuration)
- On others:
	- Determine which bootloader your distro uses and consult its documentation.

---

# What VRAM Split is Best?

Using 256MB or 512MB with the above `ttm.pages_limit` kernel parameter ensures that your system
has as much RAM available as possible while avoiding crashes in games, while still being able to
allocate as much VRAM as needed.

However, there are reports that higher VRAM splits perform a little bit better due to not needing
to reallocate memory as often. There is not conclusive data on this so it is up to you to 
experiment.

No matter what your VRAM split is, it is a good idea to set up your game settings to keep total VRAM
usage on the lower side, such as 8GB or less. 

With a low VRAM split, a game using less than 8GB of VRAM gives memory back to the regular RAM pool,
which lets a game needing lots of RAM stay out of swap more. This helps with performance and
stuttering.

With an 8GB VRAM split, memory is wasted if your game uses less than 8GB of VRAM, because the 8GB
split makes that VRAM unusable as regular RAM.
