# motion-led-tp4056

A motion-activated light that runs off a single 18650 Li-ion cell. The cell is
charged by a TP4056 module, which is fed at 5 V by an LM2596 buck converter.
An AM312 PIR sensor watches for movement. Each time it sees some, it charges a
timing capacitor that holds an IRLZ44N MOSFET on, which powers a 5 V boost
module and **two 5 V COB LED bars** for about **40 s after the last movement**.

The MOSFET switches the boost module's ground, so between triggers the boost
and the bars are fully off. With no motion, the whole circuit draws about
**20 µA**, so standby alone would take years to drain a 2500 mAh cell.

The timer, sensor and MOSFET are soldered on a 5 × 7 cm perfboard; the charging
modules, the boost module and the bars connect to it by wire.

## Schematic

![Schematic: source → LM2596 (5.0 V) → TP4056 → 18650. TP4056 OUT feeds the perfboard (J1). On the perfboard, BAT+ feeds the AM312 and J2+; the PIR charges C1 through D2 and R2, R3 drains it, and C1 drives the IRLZ44N gate. Q1 switches J2− to ground. J2 feeds a 5 V 2 A boost module that drives two 5 V 3 W COB bars in parallel.](docs/schematic.svg)

The **OUT−** pad of the TP4056 is the common ground: the PIR's GND, C1, R3 and
Q1's source all go there, **never to B−**. Each motion pulse charges C1 through
D2 and R2. R3 then drains it slowly, keeping Q1 on for about 40 s. D2 stops C1
draining back into the sensor when the sensor's output goes low.

Q1 sits in the boost module's **ground** (IN−), and the bars hang off the
boost's output. When Q1 is off, the boost module is unpowered, so it draws
nothing on standby. Because of that, **the bars' − wires go only to the boost's
OUT−, never to the battery −**: that would bypass Q1 and leave the bars on.

## Perfboard layout

5 × 7 cm board with single round pads, 24 × 18 holes (columns 1–24, rows A–R).
The board carries the sensor socket, the timer and the MOSFET; two screw
terminals take the wires to the TP4056 (J1) and to the boost module (J2).

![Perfboard: component side and copper side. Parts: J1 BAT terminal E2/G2, AM312 socket J3 D5–F5, D2 E7–E11, R2 E12–E16, C1 H15(−)/H16(+), R3 J16–N16, Q1 J19 (G) J20 (D) J21 (S), J2 BOOST IN terminal E23/G23. Copper: BAT+ along row B, GND along row Q, gate down column 16 and along row J, switched − from J20 to G23.](docs/perfboard.svg)

> **Check your board first.** This layout is for a board whose pads are all
> separate. Some boards (RadioShack 276-150, for example) join pads into groups
> or have built-in power strips; soldering this layout onto one of those can
> short nets together. Compare the copper side with the drawing before
> soldering.

**Parts (component side):**

| Part | Pins | Holes |
|---|---|---|
| J1 screw terminal, 2-pin 5.08 mm (to TP4056 OUT) | + · − | E2 · G2 |
| J3 female header, 3-pin (AM312 plugs in) | VCC · OUT · GND | D5 · E5 · F5 |
| D2 1N4148 | plain end · striped end | E7 · E11 |
| R2 1 kΩ (brown·black·red) | either leg | E12 · E16 |
| C1 100 µF | + (long leg) · − (striped leg) | H16 · H15 |
| R3 1 MΩ (brown·black·green) | either leg | J16 · N16 |
| Q1 IRLZ44N, standing, printed side facing row R | G · D · S | J19 · J20 · J21 |
| J2 screw terminal, 2-pin 5.08 mm (to boost IN) | + · − | E23 · G23 |

**Copper side (solder bridges and wire on the back):**

| Net | Run | Wire |
|---|---|---|
| BAT+ | E2 → B2 → B23 → E23, plus B5 → D5 | **thick** (1.9 A) |
| GND | G2 → Q2 → Q21, plus F5 → Q5, H15 → Q15, N16 → Q16 | **thick** along row Q and J21 → Q21 |
| Sensor out | E5 → E7 (bridge across E6) | thin |
| D2 → R2 | E11 → E12 (bridge) | thin |
| Gate | E16 → H16 → J16 down column 16, then J16 → J19 along row J | thin |
| Switched − | J20 → G20 → G23 | **thick** (1.9 A) |

Thick = 20–22 AWG wire or bare tinned copper wire, laid along the pads and
soldered at each one. Thin runs can be solder bridges between neighbouring pads
or a short piece of component lead.

**Off the board:**

| From | To |
|---|---|
| TP4056 OUT+ / OUT− | J1 + / − |
| J2 + / − | boost module IN+ / IN− |
| boost OUT+ | both COB bars' + |
| boost OUT− | both COB bars' − (**not** the battery −) |

## Breadboard layout (rev 5 prototype)

This is the earlier breadboard prototype with a single 5 mm LED and a 68 Ω
resistor in place of the boost module and COB bars. The timer part is the same
as on the perfboard.

![Half-size breadboard layout of the circuit, with each part in its hole.](docs/breadboard.svg)

Holes a–e in one numbered column are connected to each other, and so are f–j.
The top red **+** row is the OUT+ rail and the blue **−** row is the OUT− rail.

| Part | Pin / leg | Goes in |
|---|---|---|
| TP4056 | OUT+ / OUT− | + rail · − rail |
| AM312 | VCC · GND · OUT | + rail · − rail · c3 |
| D2 1N4148 | plain end · striped end | d3 · d7 |
| R2 1 kΩ | either leg | c7 ↔ c11 |
| C1 100 µF | long leg (+) · striped leg (−) | a11 · − rail |
| R3 1 MΩ | either leg | b11 ↔ − rail |
| Q1 IRLZ44N | G · D · S (printed side toward you) | e11 · e12 · e13 |
| Jumper | source to ground | c13 → − rail |
| LED | short leg (−) · long leg (+) | d12 · d15 |
| R1 68 Ω | either leg | c15 ↔ c19 |
| Jumper | to supply | a19 → + rail |

For a second LED, put its short leg in any free hole in column 12, its long leg
in a new column, and give it its own 68 Ω resistor to a column jumpered to the
+ rail.

**Check two polarities before powering it.** The IRLZ44N's legs are G-D-S from
left to right when you look at its printed side, with the metal tab facing away
from you. C1's striped leg is negative and goes into the − rail. An
electrolytic capacitor wired backwards can overheat and burst. Also check the
AM312's pin labels on its own board, because the order varies between sellers.

## Parts

| Qty | Part | Value / model | What it does |
|---|---|---|---|
| 1 | PIR sensor | AM312 mini PIR (2.7–12 V) | Detects motion. Its output is 3.3 V while it sees movement. It draws about 15 µA on standby, so it runs straight off the battery. |
| 1 | Q1 MOSFET | IRLZ44N logic-level N-channel (TO-220) | Switches the boost module's ground (~1.9 A when on, no heatsink needed). Its gate draws almost no current, so it doesn't drain the timing capacitor. |
| 1 | D2 | 1N4148 (or 1N4001–1N4007) | Lets the sensor charge C1 but stops C1 draining back into the sensor. |
| 1 | C1 | 100 µF, 16 V electrolytic | Stores the charge that keeps the light on. |
| 1 | R2 | 1 kΩ, ¼ W | Limits the current that charges C1. |
| 1 | R3 | 1 MΩ, ¼ W | Drains C1 slowly, which sets the on-time. It also keeps the LED off at power-up. |
| 2 | COB LED bars | 5 V input, 3 W, 80 × 7.5 mm ([AliExpress listing](https://es.aliexpress.com/item/1005005025060319.html)) | The light. In parallel they draw ~1.2 A at 5 V. |
| 1 | Boost module | 5 V output, **2 A or more**, 3–5 V input (FP6277- or FP6298-based "3.7 V → 5 V 2 A" boards) | Makes 5 V for the bars from the 3.0–4.2 V cell. An MT3608 can't supply 1.2 A. |
| 1 | Perfboard | 5 × 7 cm, single pads, 24 × 18 holes | Carries the sensor socket, timer and MOSFET. |
| 2 | J1, J2 | 2-pin screw terminal, 5.08 mm pitch | Wires to the TP4056 and the boost module. |
| 1 | J3 | 3-pin female header, 2.54 mm | Socket for the AM312. |
| — | Wire | 20–22 AWG or bare tinned copper for the 1.9 A paths; thinner for the rest | |
| — | Charging side | LM2596 buck module, TP4056 module **with protection** (6 pads), 18650 cell | Already part of the build. |

Optional extras:
- An SPDT slide switch in series with OUT+, as a master off switch.
- A 1 MΩ trimmer in series with 100 kΩ, in place of R3, to set the on-time
  anywhere from about 7 s to 75 s.

## Values

**Current and battery life.** Two 3 W bars at 5 V draw ~1.2 A. Through a boost
module at ~85 % efficiency, that is about **1.9 A from the cell** (6 W / 0.85 /
3.7 V). The TP4056's protection trips at roughly 3 A, so this is within range,
and the IRLZ44N drops only ~0.2–0.4 W at that current with ~2.8 V on its gate.

A 2500 mAh cell holds ~9 Wh: about **1.2 h of total light**, or roughly 100
triggers of 40 s per charge.

**On-time (C1 × R3).**

```
R3 × C1 = 1 MΩ × 100 µF = 100 s
Full brightness ≈ 0.4 × 100 s ≈ 40 s
Bars switch off ≈ 0.4–0.6 × 100 s ≈ 40–60 s
```

| C1 | R3 | Full brightness | Fully off |
|---|---|---|---|
| 100 µF | 470 kΩ | ~20 s | ~30 s |
| **100 µF** | **1 MΩ** | **~40 s** | **~70 s** |
| 220 µF | 1 MΩ | ~1.5 min | ~2.5 min |

These times are approximate. Each MOSFET switches at a slightly different gate
voltage (1–2 V for the IRLZ44N), and electrolytic capacitors vary by ±20%, so
tune R3 for your parts. New motion restarts the timer.

With the boost module, the light **switches off rather than fading**: as the
gate voltage falls, the boost module's input sags until it cuts out. A brief
flicker in the last second is normal.

**Standby.** About 15 µA (PIR) + 3 µA (TP4056 protection chip); the boost module
draws nothing because Q1 cuts its ground.

## Choosing D2

The diode has to block C1's charge with almost no leakage, because R3 drains
only about 3 µA.

| Diode | Use it? | Why |
|---|---|---|
| 1N4148 / 1N914 | Best | Almost no leakage at 3 V. |
| 1N4001 to 1N4007 | Good | Same behaviour, physically bigger. |
| BAT43, BAT46, BAT85 | OK | Small Schottky diodes, low leakage at room temperature. |
| 1N5817, 1N5818, 1N5819 | Only with R3 ≤ 470 kΩ | Power Schottky leakage (µA, and more when warm) drains C1 and shortens the on-time. |
| An LED used as a diode | No | It drops 1.8 V or more, so C1 charges to only about 1.5 V, too low to turn Q1 on. |
| Zener below 5.1 V | No | It conducts backwards at this voltage. |

You can't skip D2. Without it, C1 drains back into the sensor as soon as the
sensor's output goes low, and the light goes back to turning off after about 2 s.

## Build and test

1. With the LM2596 **not connected** to the TP4056, power it and turn its
   trimmer until a multimeter reads **5.0 V** at OUT.
2. Wire LM2596 OUT to TP4056 IN and the cell to B+ / B−. The charger's red LED
   should light while it charges.
3. Solder the perfboard from the tables above, low parts first: wire links and
   bridges, resistors and D2, then C1, the header, the terminals and Q1. Check
   the MOSFET's pin order and the capacitor's polarity.
4. **Before connecting the bars**, power the boost module from a bench supply or
   the cell and check its output reads **5.0 V** (adjust it if it has a trimmer).
5. Connect J1 to TP4056 OUT+ / OUT−, J2 to the boost IN, and the bars to the
   boost OUT. Wait **about 60 s** for the PIR to settle, because it can trigger
   on its own during warm-up.
6. Wave a hand in front of it. Both bars should light for about 40 s after the
   movement stops, then switch off. Feel Q1 after a few triggers: warm is fine,
   too hot to touch means it isn't fully on (check C1's voltage at the gate).

## Before you connect it

- **Load ground goes to OUT−, not B−.** The protection chip switches OUT−, so
  wiring to B− skips over-discharge protection and can drain the cell below 2.5 V.
- **Set the LM2596 before connecting it.** These modules come with the trimmer at
  a random setting, and above 8 V the TP4056 is destroyed.
- **Your source must supply 1 A or more.** The TP4056 charges at 1 A by default
  (R<sub>PROG</sub> = 1.2 kΩ).
- **The bars' − wires go only to the boost module's OUT−.** Tying them to the
  battery − bypasses Q1, and the bars stay on.
- **Check the boost output before connecting the bars.** Some modules are
  adjustable and ship at a random voltage; more than 5.5 V can damage 5 V bars.
- **Use thick wire for the 1.9 A paths** (BAT+ row B, GND row Q, J20 → G23, and
  the wires to the boost module). Thin wire there heats up and drops voltage.

## Revision history

| Rev | Change | Why |
|---|---|---|
| 1 | AM312 + 2N2222 NPN switch, R1 56 Ω, R2 1 kΩ, R3 100 kΩ | First design. Light stayed on about 2 s after motion (the AM312's fixed hold time). |
| 2 | R1 56 Ω → **68 Ω** | Matches the resistor on hand. LED current drops from ~18 mA to ~15 mA on a full battery. |
| 3 | Added the breadboard layout | Hole-by-hole placement for prototyping. |
| 4 | 2N2222 → **IRLZ44N**; added **C1 100 µF** and **D2**; R3 100 kΩ → **1 MΩ** | Capacitor timer extends the on-time to ~40 s. A MOSFET gate doesn't drain the capacitor the way a transistor base would. Parts moved to new holes. |
| 5 | D2 1N5819 → **1N4148** | Power Schottky leakage is comparable to R3's 3 µA drain and would shorten the on-time. |
| 6 | 5 mm LED + 68 Ω → **two 5 V 3 W COB bars** driven by a **5 V 2 A boost module**; Q1 now switches the boost module's ground; **perfboard layout** added | Much more light. Switching the boost module's ground keeps standby at ~20 µA, where a boost left powered would drain the cell in about 2 months. |
