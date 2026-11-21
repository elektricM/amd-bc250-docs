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

## Power Supply Control

!!!danger "No Native ATX Power Control"
    The BC-250 does not include native ATX 24-pin or PS_ON control circuitry. Standard ATX power supplies require manual activation.

### Manual PS_ON Control

To use a standard ATX PSU, you must manually bridge the PS_ON signal (green wire, typically pin 16 on 24-pin ATX) to ground (black wire). This can be accomplished with:

- A momentary push button
- A latching button
- A relay circuit

Without this bridge, the PSU will not power on.

### PSU Fan Control Issue

Standard ATX PSUs without motherboard control will run fans at 100% speed continuously because the BC-250 cannot provide tachometer feedback. Solutions include:

- Use a PSU with dedicated control board (e.g., relay circuit tied to 3.3V or 12V rail)
- Accept full-speed fan operation

**Recommended BIOS Setting:** Set AUTO_PWRON1 to pins 1-2 (auto power-on) when using any external PSU control solution.

### TPMS1 3.3V Power Rail

The 3.3V pin on TPMS1 (pin 9) is active only when the board is powered on, making it suitable for relay or control circuit applications that need to detect system power state.

### J1000 (PCIe 8-pin)

Standard 8-pin PCIe power connector:

```
[ GND GND GND GND ]
[ GND 12V 12V 12V ]
```

**Recommended wire gauge:** 16AWG minimum for reliable power delivery.

**Current capacity:** Each MiniFit Jr contact can handle 9A (up to 13A for industrial-grade contacts). With three 12V pins, this connector can safely supply up to 324W (9A × 3 × 12V), or up to 468W with industrial-grade contacts.

For overclocking or high-power applications, consider supplementing J1000 with power from J2000/J2001 or soldering directly to the board.

### J2000 and J2001

Alternative power connectors compatible with Molex Micro-Fit BMI [444280801](https://www.molex.com/en-us/products/part-detail/444280801):

```
        J2000                J2001
   v                     v
[ LED1 12V 12V 12V ]  [ 12V 12V 12V PGD ]
[ LED2 GND GND GND ]  [ GND GND GND GND ]
```

| Pin | Board net | Purpose |
|-----|-----------|---------|
| `PGD` (J2001 pin 4) | `PWR_SW` | Power-enable input. Pulled up to 12 V on the board through 300 kΩ: left open, the board powers up. Pulled to GND, it turns off the board's 12 V input switch and the board loses power at once. In the rack chassis this pin was driven by the PSU's power-good. |
| `LED1` (J2000 pin 1) | `LED_M_G` | Green channel of the status LED, active-low: about 0 V while green is lit, about 3 V while it is off (red showing) |
| `LED2` (J2000 pin 5) | `LED_M_R` | Red channel of the status LED, active-low: about 0 V while red is lit, about 3 V while it is off (green showing) |

!!!note "Where this comes from"
    The nets and the driver circuit below are traced in the ASRock BC-250 boardview (r1.00). They match @tfabris's measurements for [#42](https://github.com/elektricM/amd-bc250-docs/issues/42): about 3 V on `LED1` and about 0 V on `LED2` while the status LED is red, and `LED2` going high when it turns green. The older wording "active-low, `LED1` mirrors green, `LED2` mirrors red" was correct.

    Tested by: @tfabris, one board (LED1/LED2 levels). Circuit: boardview.

**PGD is not a power button.** A momentary switch from `PGD` to GND acts like pulling the plug while it is pressed. To start the board, use the onboard power button or the `AUTO_PWRON1` jumper.

#### How LED1 and LED2 are driven

Both pins sit on the cathode side of the onboard status LED (`LED16`, the red/green side-view LED at the I/O edge between the LAN and USB 3.0 ports). A 2N7002 MOSFET pulls a pin to ground to light that colour:

```
+5V ──┬── LED16 green ── 100 Ω ──┬── LED1 (J2000 pin 1)
      │                          └── 2N7002 ── GND
      └── LED16 red ──── 150 Ω ──┬── LED2 (J2000 pin 5)
                                 └── 2N7002 ── GND
```

When its MOSFET is off, a pin floats at about 3 V: 5 V minus the forward voltage of the onboard LED.

!!!danger "Never connect an LED from LED1 or LED2 to GND"
    An LED from the pin to GND draws its current through the onboard LED of that colour. That is why the status LED turned orange in #42: the external LED lit the green half next to the red one. Two ways that work:

    - **Sink the current:** connect the external LED with a 220-330 Ω series resistor between +5 V and the pin. It lights together with the onboard colour. Do not use 12 V for this: while the pin is off, the onboard LED would be reverse-biased by several volts.
    - **Buffer it:** let the pin switch a small transistor or MOSFET, and power the front-panel LED from the PSU through that. @tfabris's [LED controller](https://github.com/tfabris/BC-250/tree/master/LED%20Controller) is a worked example: each pin drives a BC548 through a 10 kΩ base resistor, and the LEDs run from the PSU's 5 V.

#### What the status LED shows

The status LED shows the load on the memory rail, not a general power or activity state:

| LED16 | Meaning |
|-------|---------|
| Red | Low current on the GDDR6 supply rail (idle) |
| Green | High current on the GDDR6 supply rail (memory busy) |

It is never both at once. The circuit, from the boardview:

1. The memory VRM controller (Renesas ISL95712, `PUIO1`) reports its output current on `+VMEMIO_IMON` (pin 11). `+VMEMIO` feeds all eight GDDR6 chips and the APU's memory interface.
2. An LM358 (`U4503`) amplifies that signal by 10.1 and compares it with 3.49 V (5 V through 1.5 kΩ / 3.48 kΩ, 1 MΩ hysteresis).
3. The comparator drives the MOSFET pair above through a 10 kΩ / 1 µF filter (about 10-20 ms). One MOSFET lights green, a second one inverts that signal for red.

The current threshold in amps depends on the ISL95712's current-monitor gain, which the boardview does not show.

This fits two kinds of field reports, since each attempt to start the memory briefly loads the memory rail:

- The LED reportedly cycles red/green/red on boards whose GDDR6 no longer trains at 1750 MHz (brain_cylinder in the BC250 Discord, September 2026; fix discussed [on Reddit](https://www.reddit.com/r/BC250Gaming/comments/1wb2izn/)).
- Arieltune's BC-250 manual describes a regular red/green cycle as the heartbeat of a boot loop.

!!!note "Not yet confirmed by observation"
    The meaning above is derived from the circuit. To confirm it, watch the status LED at idle and during a memory-heavy GPU load such as `llama-bench`: it should change from red to green. Reports welcome.

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
