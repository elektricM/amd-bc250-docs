# Power Supply Requirements

Comprehensive guide to powering your BC-250 board safely and reliably.

## Power Requirements Overview

The BC-250 is a high-performance board that requires proper power delivery for stable operation.

### Basic Requirements

- **Voltage:** 12V DC
- **Connector:** PCIe 8-pin (6+2 pin)
- **TDP:** 220W (rated)
- **Actual Power Draw:** 70-235W depending on workload
- **Minimum PSU Recommendation:** 300W on 12V rail

!!!danger "Critical Warning"
    Always verify your PSU can deliver the required wattage on the 12V rail. Many budget PSUs cannot sustain their rated output, leading to system instability, crashes, or PSU failure.

## Measured Power Consumption

Real-world power measurements from community testing:

| Workload | Power Draw (Watts) | Notes |
|----------|-------------------|-------|
| Idle (no governor) | 85-105W | GPU locked at 1500 MHz |
| Idle (with governor) | 65-85W | GPU at 1000 MHz minimum |
| Desktop / Light Use | 70-90W | Web browsing, media playback |
| Gaming (Medium) | 120-150W | 1080p gaming, non-RT |
| Gaming (Heavy) | 160-200W | Demanding AAA titles |
| Cyberpunk 2077 (RT) | **235W** | Maximum observed (stock) |
| Gaming (with GPU freq patch) | **250W+** | Can exceed 235W with kernel patch |
| Furmark (Stock) | 250W | Stress test (unrealistic) |
| Furmark (OC to 2230 MHz) | **320W** | Extreme stress test |

!!!info "Power Savings with Governor"
    Installing the GPU governor can reduce idle power consumption by 20-30W by dynamically scaling GPU frequency.

[See GPU Governor Guide](../system/governor.md)

## Recommended PSU Options

!!!danger "Avoid Low-Wattage PSUs"
    Dell D220P-01 and D250AD-00 PSUs are **NOT RECOMMENDED** despite appearing cheap. At 220W/250W, they are insufficient for the BC-250's peak loads and have been reported to "cut out or even break" under gaming loads. Minimum 300W on the 12V rail is required for reliable operation.

### Option 1: Mean Well LOP-300-12

**Specifications:**
- **Model:** Mean Well LOP-300-12
- **Output:** 12V @ 25A (300W)
- **Form Factor:** Open frame
- **Pros:** High quality, medical-grade, reliable
- **Cons:** Requires custom mounting and wiring

**Features:**
- Over-current protection
- Over-voltage protection
- Thermal shutdown
- Medical-grade safety certification

!!!warning "Wiring Required"
    This PSU has bare terminals. You'll need to crimp your own PCIe 8-pin connector.

### Option 2: FlexATX PSU (500W)

**Specifications:**
- **Form Factor:** FlexATX (150mm x 81.5mm x 40.5mm)
- **Output:** 500W total, ~300W on 12V rail
- **Pros:** Standard ATX connectors, compact, modular options available
- **Cons:** Fan can be loud

**Recommended Models:**
- FSP FSP500-30AS (popular, 500W)
- Metalfish 500W FlexATX (new, modular)
- Enhance ENP-7660B (high quality, 600W)

!!!success "Plug and Play"
    FlexATX PSUs have standard PCIe 8-pin connectors, making installation straightforward.

### Option 3: Standard ATX PSU

**Specifications:**
- **Form Factor:** ATX (150mm x 140mm x 86mm)
- **Output:** 400W+ recommended
- **Pros:** Widely available, reliable, standard connectors
- **Cons:** Large, overkill for BC-250

**Minimum Recommendations:**
- 400W+ total output
- 20A+ on 12V rail (240W+)
- 80 Plus Bronze or better efficiency

!!!info "Using Existing PSU"
    If you have a spare ATX PSU, it will work fine. Use a standard PCIe 8-pin cable.

### Option 4: Server PSU

**Specifications:**
- **Form Factor:** Various (1U, 2U)
- **Output:** 700W-1200W typical
- **Pros:** Very cheap secondhand, high power, efficient
- **Cons:** **Extremely loud**, requires modifications

**Typical Models:**
- HP DPS-800GB
- Delta DPS-750RB
- Bitmain APW3++ (220W idle!)

!!!danger "Not Recommended for Desktop Use"
    Server PSUs use high-speed (10,000+ RPM) fans that sound like jet engines. Only suitable for rack-mounted or garage installations.

!!!danger "LED PSUs Not Recommended"
    12V LED/Industrial power supplies are **NOT RECOMMENDED** for the BC-250. They have unreliable ripple current and quality varies too widely to be safe. The risk of instability or component damage is too high.

## PSU Safety and Requirements

### Calculating Required Wattage

**Formula:**
```
Required Wattage = Max Power Draw * Safety Margin
Required Wattage = 235W * 1.2 = 282W
```

**Minimum:** 250W on 12V rail
**Recommended:** 300W+ on 12V rail for overclocking headroom

!!!tip "Check the 12V Rail"
    Many PSUs split 12V output across multiple rails. Ensure a single rail can provide at least 220W, or use a PSU with a single 12V rail.

### Connector Requirements

**J1000 - PCIe 8-pin (6+2 pin) Pinout:**

```
[ GND GND GND GND ]
[ 12V 12V 12V GND ]
```

| Pin | Function |
|-----|----------|
| 1-3 | 12V |
| 4-6 | Ground |
| 7-8 | Ground (sense pins) |

This is the standard power connector and is perfectly suitable for powering the BC-250.

!!!info "Using 6-pin Connector"
    Some users report success with 6-pin connectors (missing pins 7-8), but this is **not recommended**. The missing sense pins can cause compatibility issues.

**J2000 and J2001 - Alternative Power Connectors:**

These connectors are compatible with 8-pin Molex Micro-Fit connectors ([444280801](https://www.molex.com/en-us/products/part-detail/444280801)):

```
   v                     v
[ LED1 12V 12V 12V ]  [ 12V 12V 12V GND ]
[ LED2 GND GND GND ]  [ GND GND GND PGD ]
```

| Pin | Purpose |
|-----|---------|
| `PGD` | `PGOOD` - 5V when PSU2 is connected to rack chassis |
| `LED1` | Active-low LED output - mirrors green backplane LED |
| `LED2` | Active-low LED output - mirrors red backplane LED |

If powering the BC-250 from these connectors, use both J2000 and J2001 for redundancy.

### Cable Quality

- **Use 16 AWG minimum** wire for high current capacity (18 AWG has caused melted cables)
- **Avoid adapters** (SATA-to-PCIe, Molex-to-PCIe) - these are fire hazards
- **Check cable temperature** under load - warm cables indicate resistance issues
- **Crimp properly** if making custom cables - poor crimps create hot spots

!!!danger "Adapter Fire Hazard"
    SATA connectors are rated for 54W. Using SATA-to-PCIe adapters with a 220W board is a **fire risk**. Two Molex connectors (156W combined) are also insufficient.

## Power-On Control

### Method 1: Auto Power-On

The BC-250 starts automatically when 12V power is applied.

**Setup:**
1. Connect PSU to BC-250
2. Turn on PSU
3. Board powers on immediately

**Use Case:** Simple setups where PSU has an on/off switch

### Method 2: Power Button (Soldering Required)

The BC-250 does **NOT have a power button header**. To add an external power button, you must solder directly to the existing onboard power button.

**Setup:**
1. Identify the onboard power button on the rear of the board
2. Solder wires to both sides of the power button
3. Connect to external momentary switch
4. Short to turn on, short again for soft shutdown
5. Hold for 5+ seconds for hard power-off

**Use Case:** Advanced builds with custom case integration

!!!warning "Soldering Required"
    This modification requires soldering skills. There is no power button header on the BC-250.

### Method 3: ATX PSU Control

For ATX PSUs, the 24-pin connector includes a power-on signal.

**Permanent On (Jumper Method):**
1. Short pin 16 (green, PS_ON) to any ground pin (black)
2. PSU runs whenever plugged in

**Soft Power Control:**
1. Leave PS_ON pin unconnected
2. Use external switch connected to PSU PS_ON signal
3. Switch bridges PS_ON to ground when pressed

!!!info "Remote Power On"
    Some users have successfully implemented Wake-on-LAN for remote power control.

## Power Supply Issues and Troubleshooting

### System Crashes Under Load

**Symptoms:**
- System shuts off during gaming or benchmarks
- Random reboots
- PSU makes clicking sound before shutdown

**Cause:** PSU over-current protection triggered

**Solutions:**
1. **Verify PSU wattage** - must support 220W+ on 12V rail
2. **Check cable connections** - loose connections create resistance
3. **Reduce GPU voltage** - lower max voltage in governor config
4. **Upgrade PSU** - use higher wattage unit

!!!warning "Insufficient PSU Power"
    A 180W PSU **will not work** for gaming. A 220W PSU is marginal and may trip protection during demanding workloads.

### PSU Fan Noise

**Symptoms:**
- PSU fan makes rattling or buzzing sound
- Fan speed fluctuates
- High-pitched coil whine

**Causes:**
- Cheap bearing (sleeve bearing)
- Coil whine from transformers
- Fan hitting PSU housing

**Solutions:**
1. **Replace PSU fan** - upgrade to quality fan (Noctua, Arctic)
2. **Accept the noise** - some PSUs are just noisy
3. **Upgrade PSU** - higher quality units are quieter

### Coil Whine

**Symptoms:**
- High-pitched whine from PSU
- Worse at idle or low load
- Varies with GPU frequency

**Cause:** Transformer coils vibrating at audible frequencies

**Solutions:**
1. **Apply load** - some PSUs only whine at low loads
2. **Damping material** - hot glue on transformer (risky!)
3. **Replace PSU** - no reliable fix for coil whine

### PSU Overheating

**Symptoms:**
- PSU shuts down after 10-30 minutes
- PSU fan runs at max speed
- PSU housing is very hot to touch

**Causes:**
- Inadequate PSU cooling
- PSU loaded beyond rating
- High ambient temperature

**Solutions:**
1. **Improve PSU airflow** - ensure PSU fan intake is clear
2. **Add case fan** - exhaust hot air from PSU area
3. **Reduce load** - lower GPU max frequency/voltage
4. **Upgrade PSU** - use higher wattage unit with better cooling

## DIY Power Supply Modifications

### Making Custom PCIe Cables

**Required Tools:**
- Wire crimpers
- PCIe 8-pin connector housing
- 16 AWG wire minimum (silicone insulation recommended)
- Pin removal tool (optional)

**Steps:**
1. Cut 8 wires to appropriate length (~30cm)
2. Strip 5mm of insulation from each end
3. Crimp terminals onto wire ends
4. Insert pins into PCIe connector (3x 12V, 5x GND)
5. Verify continuity with multimeter
6. Test with low load before full gaming

!!!danger "DIY Safety"
    Poor crimps can cause fire. Test cables under load and monitor temperature. If cables get warm, they have high resistance and should be redone.

### Shorting ATX Connector for Always-On

**24-Pin ATX Pinout:**
- Pin 16 (Green): PS_ON signal
- Pins 15, 17 (Black): Ground

**Method:**
1. Use paperclip or jumper wire
2. Bridge pin 16 to pin 15 or 17
3. PSU turns on when plugged in

**Use Case:** External PSU that powers BC-250 only

### Adding Power Switch

**Components:**
- Momentary push button switch
- 2-conductor wire

**Wiring:**
1. Connect switch between PSU PS_ON and GND
2. Test that short press powers on/off
3. For BC-250 control, must solder to onboard button (no header available)

## PSU Efficiency

**80 Plus Certification:**
- **80 Plus Bronze:** 82-85% efficient at 50% load
- **80 Plus Silver:** 85-88% efficient at 50% load
- **80 Plus Gold:** 87-90% efficient at 50% load
- **80 Plus Platinum:** 90-92% efficient at 50% load

**Efficiency Impact:**
- Bronze PSU at 180W load: ~212W from wall
- Platinum PSU at 180W load: ~197W from wall
- Savings: ~15W (varies with load)

!!!info "PSU Efficiency Matters"
    Higher efficiency PSUs waste less power as heat, reducing cooling requirements and long-term operating costs.

## Recommended PSU Summary

| Use Case | Recommended PSU |
|----------|-----------------|
| **Budget Build** | FlexATX 500W (secondhand) |
| **Compact Build** | FlexATX 500W |
| **Quality Build** | Mean Well LOP-300-12 |
| **Reuse Existing** | ATX 400W+ |
| **Multi-Board** | Server PSU + breakout |

## See Also

- [Hardware Specifications](specifications.md)
- [Cooling Solutions](cooling.md)
- [GPU Governor Setup](../system/governor.md)
- [Troubleshooting Display Issues](../troubleshooting/display.md)
