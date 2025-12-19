# Hardware Pinouts

Detailed connector pinouts and chip identification for the BC-250 board.

!!!info "Source"
    This information is based on documentation from [mothenjoyer69's bc250-documentation](https://github.com/mothenjoyer69/bc250-documentation) repository. Credit to mothenjoyer69, Segfault, neggles, and yeyus for the reverse engineering work.

## Connector Overview

Connectors are listed clockwise from the M.2 header. Pin 1 is generally indicated on the PCB by a white silkscreen triangle (shown as `>` or `^` below).

## Storage

### M2_1

M-keyed M.2 slot supporting:

- Two lanes of PCIe 2.0
- SATA III connection

## Jumpers

### AUTO_PWRON1

```
> [ 1 2 3 ]
```

| Jumper Position | Behavior |
|-----------------|----------|
| Pins 1-2 | Auto power-on when 12V applied (default) |
| Pins 2-3 | Wait for power button press |

### CLRCMOS1

```
> [ 1 2 3 ]
```

| Jumper Position | Behavior |
|-----------------|----------|
| Pins 1-2 | Power CMOS from CR2032 battery (default) |
| Pins 2-3 | Clear CMOS settings |

## I2C and Debug Headers

### I2C_HEADER1

```
> [ SCL SDA GND ]
```

The SCL pin is on the "lower" side of the board, closer to the power connectors.

This exposes an I2C interface which hosts PMBUS communications to the Intersil PMICs.

### TPMS1 (LPC Header)

18-pin 2.0mm pitch header for boot-time monitoring:

```
 PCICLK -- [  1   2 ] -- GND
  FRAME -- [  3   4 ] -- SMB_CLK_MAIN
PCIRST# -- [  5   6 ] -- SMB_DATA_MAIN
   LAD3 -- [  7   8 ] -- LAD2
     3V -- [  9  10 ] -- LAD1
   LAD0 -- [ 11  12 ] -- GND
           [     14 ] -- S_PWRDWN#
   3VSB -- [ 15  16 ] -- SERIRQ#
    GND -- [ 17  18 ] -- GND
```

LPC is clocked relative to `PCICLK` at 33MHz.

**Minimal connections for LPC monitoring:**

```
  [ GND    -     - LAD2 LAD1 -    - - - ]
> [ PCICLK FRAME - LAD3 -    LAD0   - - ]
```

### J2 (JTAG Debug)

Unpopulated 20-pin 1.27mm pitch footprint on the bottom of the board. This is an AMD HDT+ debug connector for JTAG debugging.

```
 VDDIO -- [  1   2 ] -- TCK
   GND -- [  3   4 ] -- TMS
   GND -- [  5   6 ] -- TDI
   GND -- [  7   8 ] -- TDO
TRST_L -- [  9  10 ] -- PWROK_BUF
DBRDY3 -- [ 11  12 ] -- RESET_L
DBRDY2 -- [ 13  14 ] -- DBRDY0
DBRDY1 -- [ 15  16 ] -- DBREQ_L
   GND -- [ 17  18 ] -- TEST19
 VDDIO -- [ 19  20 ] -- TEST18
```

Note: Pins `TEST18`, `TEST19`, `DBRDY0` are left floating on this PCB.

## Fan Headers

### CPU_FAN1

Standard 4-pin PWM-capable fan header:

```
[ PWM Tach 12V GND ]
                ^
```

### J4003 (Multi-Fan Header)

2.54mm-pitch connector for controlling five 80mm fans (designed for rack chassis):

```
[ GND F1T F2T F3T F4T F5T DET     ]
[ GND F1P F2P F3P F4P F5P GND GND ]
   ^
```

| Pin | Purpose |
|-----|---------|
| `F1T` | Fan 1 (`CPU_FAN1`) Tachometer signal |
| `F1P` | Fan 1 PWM control input |
| `FnT` | Fan `n` Tachometer |
| `FnP` | Fan `n` PWM control |
| `DET` | Grounded if connected to power distribution board |
| `GND` | Ground |

Fan 1 signals correspond to `CPU_FAN1` tachometer and PWM pins.

**Fan Numbering (BIOS vs Linux):**

| BIOS Fan | Linux Fan (NCT6686) |
|----------|---------------------|
| 1 | 2 |
| 2 | 3 |
| 3 | 4 |
| 4 | 5 |
| 5 | 1 |

## Power Connectors

### J1000 (PCIe 8-pin)

Standard 8-pin PCIe power connector:

```
[ GND GND GND GND ]
[ 12V 12V 12V GND ]
```

### J2000 and J2001

Alternative power connectors compatible with Molex Micro-Fit BMI [444280801](https://www.molex.com/en-us/products/part-detail/444280801):

```
   v                     v
[ LED1 12V 12V 12V ]  [ 12V 12V 12V GND ]
[ LED2 GND GND GND ]  [ GND GND GND PGD ]
```

| Pin | Purpose |
|-----|---------|
| `PGD` | `PGOOD` - 5V when PSU2 connected to rack chassis |
| `LED1` | Active-low LED output - mirrors green backplane LED |
| `LED2` | Active-low LED output - mirrors red backplane LED |

Use both J2000 and J2001 for redundancy when powering from these connectors.

## SPI Flash Header

### J4004

2.54mm header for reflashing the BIOS SPI flash chip:

```
[ GND SCLK MOSI UNK ]
[ VCC  CS  MISO     ]
   ^
```

| Pin | Function |
|-----|----------|
| VCC | 3.3V |
| GND | Ground |
| CS | Chip Select |
| SCLK | Serial Clock |
| MOSI | Master Out, Slave In |
| MISO | Master In, Slave Out |
| UNK | Unknown (tied to ground via 10kOhm resistor) |

## Auxiliary Chip Identification

| # | Designator | Chip | Description |
|---|------------|------|-------------|
| 1 | `M2U2` | NXP CBTL04083B | 2:1 PCIe x4 Multiplexer |
| 2 | `PUIO1` | Intersil ISL95712 | Core supply PMIC |
| 3 | `PUA11`, etc | Intersil ISL99360 | Smart Power Stage (phase controller) |
| 4 | `PUA1` | Intersil ISL69247 | Main PMIC |
| 5 | `U30` | Realtek RTL8111H | Ethernet NIC (PCIe x1) |
| 6 | `BIOS_A1` | Winbond 25Q128JVSQ | 16MiB SPI flash (BIOS) |
| 7 | `SU1` | AMD 218-0844029 | A68H Bolton-D2H FCH chipset |
| 8 | `UIO1` | Nuvoton NCT6686D | SuperIO controller |
| 9 | `SIO1_R` | Macronix MX25L4006E | 512KiB SPI flash (SuperIO program) |

!!!danger "Two Flash Chips"
    The board has two SPI flash chips. When flashing BIOS:

    - **Target:** `BIOS_A1` (16MB) - Winbond or Macronix
    - **Avoid:** `SIO1_R` (512KB) - Flashing this will brick the SuperIO controller

## See Also

- [Hardware Specifications](specifications.md)
- [Power Requirements](power.md)
- [BIOS Flashing Guide](../bios/flashing.md)
- [Sensors & Monitoring](../system/sensors.md)
