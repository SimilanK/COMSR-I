# COMSR-I Main Board — Design Review (2026-10-28)

Checked: `Main board.kicad_sch` + `untitled.kicad_sch`, `Main board.kicad_pcb`, `Mainboard.ioc`, against DTEG 2027 §6.9–6.17 and Appendix B.
Pin and protocol details are in `PIN_PROTOCOL_MAP.md`.
Assumption: the board's long axis (75 mm) runs along the rocket axis.

---

## A. Schematic problems

### Must fix (safety, or the board won't work)

| # | Problem | Where | Fix |
|---|---|---|---|
| S1 | No arm switch, no power switch and no fuse. Battery → PYRO+ directly, and the logic is on as soon as the battery is plugged in. | J6 → PYRO+ | Add an off-board, externally reachable switch on a locking connector, plus a fuse on PYRO+. DTEG 6.9.1, 6.14.4, 6.15.1–6.15.3 |
| S2 | No buzzer. Arming has to be verified by sound from >100 mm away in 76 dBA noise. LEDs inside the airframe don't count. | — | Add a buzzer + driver FET on a free GPIO. DTEG 6.16.1–6.16.2 |
| S3 | FIRE_C on PC13 and FIRE_D on PC15. These are backup-domain pins: ~3 mA, ≤2 MHz, ST says don't use them as current sources. Here they drive a 3.3k pull-down plus a gate. | U1 pins 7, 9 | Move to normal pins (PE7, PE8, PE10, PB1, PE12–PE15 are free) |
| S4 | JP1 is the only PYRO_GND↔GND link. Its footprint is a diode 0805 with no part number. Empty pad = no channel can fire. | JP1 | Use a real 0 Ω rated for the 5 A pulse (1206 or bigger), or a solid copper tie |
| S5 | SERVO1/SERVO2 go straight from MCU pins to the field connector. No buffer, no series R. CN1 pin 2 (SERVO1) sits next to pin 1 (V+, up to 8.4 V). One stray strand kills the MCU. | CN1/CN2, PE9/PE11 | Put U7 back on the board, or add series R + clamp. Don't put a signal pin next to V+ |
| S6 | BAT_LEVEL (PC3) is floating. The divider R27/R28/C30/D2 is set "not on board". | A1.2 | Put it back on the board, or remove the label and the ADC use |
| S7 | CN1 and CN2 pinouts are mirrored, and neither matches `MODULE_AND_PART_LIST.md`. Easy to land an e-match wrong in the field. | CN1, CN2 | Make both connectors identical. Update the doc |
| S8 | USART1 labels are swapped vs .ioc: net USART1_RX is on PA9 (= TX in .ioc), USART1_TX is on PA10. USART3 uses the opposite convention. | U1 pins 68/69, J7 | Pick one convention and fix the labels |
| S9 | J7 (USART1) has no GND pin. | J7 | Use a 3-pin connector with GND |

### Should fix

| # | Problem | Fix |
|---|---|---|
| S10 | No e-match continuity sense. R40–R43 pull each drain to PYRO+, but nothing reads them. | Divider from each drain to an ADC pin (PA0, PA1, PA4, PC0–PC2 are free) |
| S11 | SW3/SW4 (small slide switches) carry the servo supply or PYRO+ to CN pin "V+ select". | Check the switch current rating against servo stall current. Make sure an e-match can't be wired through it |
| S12 | PYRO+ has no reverse-polarity or TVS protection (D3 only protects the logic). J3 feeds VBAT+ straight, bypassing D3. | Decide what J3 is for. Add protection to J3 or remove it |
| S13 | X1 is a clipped-sine TCXO (KDS DSB211SDN). .ioc has HSE_VALUE = 16 MHz, but the part is 32 MHz. | Set HSE to 32 MHz, analog bypass. Check the ST datasheet for whether a DC-cut cap is needed |
| S14 | Connectors without a positive lock on safety-critical lines: J2 BEC (2.54 header), J8 (header), J7 (XH friction), CN1/CN2 (screw blocks). | DTEG B4.2: locking crimp connectors, or tie-wrap retention |
| S15 | J8 supplies +3V3 to the telemetry radio, shared with the 1 A LDO. | Check the radio's supply voltage and peak current |
| S16 | microSD has only C37 100 nF, ~8 mm away. | Add 10 µF + 100 nF at the Card1 VDD pin |
| S17 | D4 and D7 are both always-on +3V3 LEDs. | Repurpose D7 as a "PYRO+ live" indicator |
| S18 | ICM_CS (PB2) has no pull-up. The IMU CS floats during reset. MS5607_CS has R21. | Add a 47k pull-up |
| S19 | USB-C shield pads are unconnected. | Tie to GND (direct, or 1 MΩ ∥ 4.7 nF) |
| S20 | 8.4 V max = 2S lithium. LiPo is banned (DTEG 6.17.1.1). | Use 2S cylindrical Li-ion or LiFePO4 |
| F1 | Firmware: TIM1 CH2 (SERVO2) is not configured in .ioc. | Enable CH2 PWM |

---

## B. PCB flaws (current file)

| # | Flaw | Measured |
|---|---|---|
| P1 | **PYRO+ pour on In2.Cu covers x121–160, y74–126**: about 2/3 of the board, including under U1, X1, Y1, U9 and FB1. The firing pulse runs in a plane under the clocks and IMU. It is also on 0.5 oz inner copper, against your own rule. | 2 PYRO+ zones on In2 |
| P2 | No GND plane on In1. 0 vias, 45 track segments. The board is basically unrouted, so moving parts now is cheap. | — |
| P3 | Pyro is split over two edges: Q1/CN1 at the bottom, Q2/CN2 on the left edge mid-board. Not one isolated edge. | Q1 (152.5, 115.3), Q2 (132.7, 88.0) |
| P4 | JP1 (where all firing return current joins GND) sits mid-board, next to the clocks. | JP1–Y1 4.1 mm, JP1–X1 8.0 mm, JP1–U1 9.8 mm |
| P5 | Clocks are next to pyro parts. | X1–R42 6.5 mm, X1–Q2 6.6 mm, Y1–Q2 5.9 mm |
| P6 | The IMU sits between Q1 and U1. FIRE_A/FIRE_B traces run ~2.7 mm past it (x ≈ 144–146). | U9–Q1 10.3 mm, U9–R35 10.9 mm |
| P7 | Firing bulk caps C31/C32 are back at the battery connector, not at the FETs. This undoes the earlier fix. | C31–Q2 ~21 mm, C31–Q1 ~54 mm |
| P8 | Baro is close to the SD card (insertion stress) and the LDO (heat). | U3–Card1 5.7 mm, U3–U10 ~10 mm |
| P9 | D5, D6, R6, R14 are outside the board outline. | x ≈ 189–195 |
| P10 | J3 is the only bottom-side part, which conflicts with single-sided assembly. | J3 on B.Cu |
| P11 | SW4 at −90° puts its slide axis along the flight axis. SW3 (180°) is fine. | DTEG B4.3: perpendicular to flight |
| P12 | Manual pour names don't match their nets ("CN1-Pad2" zone = net CN1-Pad4, "CN1-Pad3" = CN1-Pad5). Left over from the pinout change. | Recheck all manual F.Cu pours |
| P13 | `MODULE_LAYOUT_NOTES.md` coordinates are stale (it says U9 is at 182.5, 95; it's now at 150.2, 103). | Don't trust those numbers |

---

## C. Placement changes — by nature and distance

Split the board along its length into zones. Noise and current stay at one end, quiet parts at the other, and the MCU sits in between.

```
 ┌──────────── short edge (top) ────────────┐
 │ QUIET:  X1, Y1, U3 baro, U9 IMU           │  data connectors on this end:
 │         (solid GND under, no traces)      │  USB-C, microSD, SWD, UARTs
 │------------------------------------------ │
 │ MCU:    U1 + decoupling, FB1/C12/C13      │
 │         on the VDDA side                  │
 │------------------------------------------ │
 │ POWER:  LDO U10, U11/U12, buck U2/L1      │
 │------------------------------------------ │
 │ PYRO + ENTRY: J6, fuse, arm connector,    │  all field wiring on this end:
 │   D3, JP1, Q1, Q2, CN1, CN2, SW3, SW4,    │  battery, arm, e-matches,
 │   R35–R43, C31, C32, J2 BEC, buzzer       │  servo/BEC
 └──────────── short edge (bottom) ─────────┘
```

| Part / group | Why (its nature) | Rule / target distance |
|---|---|---|
| Q1, Q2, CN1, CN2, C31, C32, R35–R43, SW3, SW4 | 5 A pulses, field wiring, ESD entry | All on one short edge. Drain → connector ≤ 5 mm. C31/C32 ≤ 5 mm from the FETs |
| J6, fuse, arm connector, D3 | Battery entry, highest current | Same edge as pyro, so the firing loop (J6 → fuse → arm → CN → match → FET → JP1 → J6) stays inside that strip |
| JP1 (PYRO_GND tie) | All return current joins GND here | At J6's negative pin (≤ 5 mm). Never mid-board |
| PYRO+ / PYRO_GND copper | Pulse current | Outer layers only, inside the pyro strip. Remove both In2 PYRO+ pours |
| U2, L1 (buck) | SW node, highest dV/dt | Next to the power entry. ≥ 15 mm from X1, Y1, U9 |
| U10, U11, U12 (LDO + OR diodes) | Heat | Between buck and MCU. ≥ 15 mm from U3 |
| X1, Y1 | Victims (high-impedance clock input) | ≤ 5 mm trace to OSC_IN. ≥ 20 mm from pyro copper, JP1 and the buck SW node. No PYRO/power copper under them on any layer |
| U9 IMU | Picks up magnetic field from the pulse loop and board stress | ≥ 20 mm from FETs and CN. ≥ 8 mm from mounting holes and big connectors. Keep-out on all layers under it |
| U3 baro | Sensitive to heat and stress | ≥ 15 mm from LDO, buck, FETs, D3. ≥ 10 mm from Card1 and connectors. No vias within 2 mm |
| FB1, C12, C13 (VDDA) | Analog reference, ADC accuracy | ≥ 15 mm from pyro and from JP1 |
| Card1 | CLK is an aggressor; insertion force | On a long edge at the data end, away from U3 |
| J5 USB, J1 SWD, J7, J8 | Data, low current | Data end. Keeps them physically apart from pyro so a plug can't go in the wrong place |
| Buzzer | Magnetic buzzer = magnet + coil | In the pyro/power strip, ≥ 20 mm from U9 |
| SW3, SW4 | Slide switches | Slide axis perpendicular to the flight axis |
| XT30, screw blocks, other parts > 7 g | Mass | Near a mounting hole, staked (DTEG B3.6) |

**Planes:**
- In1 = solid GND across the whole board.
- In2 = +3V3 under the MCU and quiet zones, plus a small +5V island at the power zone.
- No PYRO+ on any inner layer.
- Under X1, Y1, U9 and U3: GND only, no traces.
