# CURSR-V SRAD Flight Computer — Module & Part List

**110 parts.** Board 50 × 75 mm, 4-layer (F.Cu / In1=GND / In2=POWER / B.Cu).
Page 1 = `Main board.kicad_sch`, page 2 (Sensors) = `untitled.kicad_sch`.
Group codes here match the block titles in the schematic.

> **Primary reference:** `references/PCBDesignTutorialRevA.pdf` — consulted before
> any internet source for layout/placement/routing decisions.

**Verified against the netlist, not from memory.** Every part below was assigned
by tracing actual nets. All 110 assign cleanly; nothing is unallocated.

---

## Open items (read before layout)

1. **No arming switch and no fuse in `PYRO+`.** The net runs straight from J6
   through D3 to the FETs and output connectors. The arm break we designed for
   does not exist in the schematic. This is the safety interlock — it must be
   added before the board is built.
2. **C25 / C26** are 5 V-rail bulk, provisionally grouped with the buck. If
   either sits at U7 (+5 V) or at the USB VBUS pin, move it to that group.
3. **Naming collision:** capacitor **C2** vs group code **C2 (Comms)**. Use group
   *names*, not codes, in instructions.

## Settled decisions

- **Fab:** JLCPCB. Stackup = standard 4-layer 1.6 mm — prepreg **0.2104 mm**,
  core **1.065 mm**, prepreg **0.2104 mm**, εr **4.6**.
- **Inner copper is 0.5 oz** (outer 1 oz) → all PYRO+, firing drains and battery
  current stay on **F.Cu / B.Cu**, never an inner layer.
- **Pyro is not current-limited, by design.** Firing current = V_batt / R_ematch.
  Size the path for a **~5 A peak** (8.4 V into a 1.6 Ω match), not the ~2 A
  nominal. Protection against a dead short is the arm fuse, not a series resistor.
- **Actuation mode select is SW3 / SW4 (SPDT).** The old J4 / J9 jumper headers
  are gone. Set mode only when disarmed; never switch under an armed PYRO+.
- **CN1 / CN2 pinout:** 1 = PYRO+ · 2 = GND (servo return + guard) · 3 = fire
  (upper ch) · 4 = mode-selected V+ · 5 = SERVOx · 6 = fire (lower ch).

## Net classes & constraints (set and verified in `Main board.kicad_pro`)

| Class | Track | Via |
|---|---|---|
| Pyro | 1.5 mm | 1.0 / 0.5 |
| Battery | 1.0 mm | 0.9 / 0.45 |
| Servo | 0.8 mm | 0.8 / 0.4 |
| Power | 0.5 mm | 0.7 / 0.35 |
| USB | 0.25 mm, diff pair 0.25 / gap 0.15 (90 Ω) | 0.6 / 0.3 |
| HS_Clock, Default | 0.2 mm | 0.6 / 0.3 |

Constraints: min clearance **0.127**, min track **0.127**, copper-to-edge **0.3**,
hole-to-hole **0.25**, zone clearance **0.25**.

---

# Groups — build order

Tier = the order blocks are **completed**, not placed. Everything is rough-placed
first (tutorial step 2); the MCU is placed early as the anchor even though it
completes in Tier B.

## TIER A — power & high-energy (complete first)

| Group | Parts |
|---|---|
| **A0 Mechanical** (4) | H1 H2 H3 H4 |
| **A1 Battery input** (3) | J6 D3 JP1 |
| **A1.2 Battery sense** (4) | R27 R28 C30 D2 |
| **A2 5 V buck** (10) | U2 L1 C27 C28 C29 R24 R25 R26 C25 C26 |
| **A3 3V3 LDO + path** (7) | U10 U11 U12 C35 C36 C1 C37 |
| **A4 BEC interface** (2) | J2 J3 |
| **A5 Actuation** (20) | Q1 Q2 CN1 CN2 SW3 SW4 · R35 R36 R37 R38 (gate series) · R2 R32 R33 R34 (gate pulldowns) · R40 R41 R42 R43 (continuity bleed) · C31 C32 (firing bulk) |

Layout notes: buck input-cap loop tight, SW node small (§1). Actuation on an
isolated edge, wide short drain paths, bulk cap at the FET, `PYRO_GND` its own
region on In2.Cu with a single tie at JP1 (§7).

## TIER B — MCU & signal

| Group | Parts |
|---|---|
| **B1 MCU core** (14) | U1 C3 C4 C5 C6 C7 C8 C9 C34 FB1 C12 C13 C14 R19 |
| **B2 Clocks** (4) | X1 C10 Y1 C11 |
| **B3 Sensors** (9) *(page 2 + SPI2 series)* | U9 C17 C19 R31 · U3 C18 R14 · R15 R16 |
| **B4 microSD** (6) | Card1 R9 R10 R11 R12 R13 |
| **B5 PWM level shift** (4) | U7 C15 R5 R8 |

Layout notes: one 100 nF per VDD/VSS pair at the pin (§8); C34 VCAP tight.
Clocks short with ground guards (§5). No copper or vias under U9/U3 (§2, §3).
**Route microSD in the same pass as the sensors** — SDMMC CLK is the aggressor
and `SD × baro` is the worst crossing (§9).

## TIER C — standard

| Group | Parts |
|---|---|
| **C1 USB** (4) | J5 D1 R22 R23 |
| **C2 Comms** (2) | J7 (USART1) J8 (USART3) |

USB is Full Speed (12 Mbps) — the pair is forgiving. D1 ESD **at the connector**.

## TIER D — finishing

| Group | Parts |
|---|---|
| **D1 LEDs & buttons** (13) | D4 D5 D6 D7 R21 R29 R39 R44 SW1 SW2 R1 C2 R7 |
| **D2 Debug** (4) | J1 R3 R4 R20 |

J1 TagConnect needs 3.6 mm clear for the pogo body (§11) — reserve the space
early even though its routing is trivial.

---

## Cross-cutting, still outstanding

Arm switch + fuse in PYRO+ · thermal vias on U2 VIN/GND · `PYRO_GND` zone on
In2.Cu · zone keep-out under U9 & U3 · ground guards at HSE_IN/LSE_IN · diff-pair
router on USB · correct U2 oversized pads · silkscreen the pyro screws.
