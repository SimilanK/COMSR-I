# CN1 / CN2 / J6 / J3 — orientation & placement plan (2026-10-28)

Board 50 × 75 mm, outline x 121.36–171.36, y 51.4–126.4. Pyro + power end = bottom short edge (y 126.4), facing the MCU South side.
Mounting-hole courtyards: H3 x ≤ 127.6, H4 x ≥ 165.2, both y ≥ 120.2.
Clearance used: connector courtyard 0.5 mm from the board edge and from other courtyards.

## Hard constraint
- PA001-6P courtyard = 30.2 × 8.4 mm.
- Space on the bottom edge between H3 and H4 = 37.6 mm.
- So only one PA001 fits on the bottom edge. Two need 60.4 mm.

## Rules for all four connectors
1. The wire or plug side faces the board edge. No cable crosses the board.
2. Every field connector stays in the bottom pyro/power zone, away from the MCU West side (clocks).
3. CN1 and CN2 get the same pinout. Each fire drain sits next to a PYRO+ pin. No signal pin (SERVO) next to V+.
4. Check the opening side in the KiCad 3D viewer. The footprint body is asymmetric: −3.17 / +5.21 mm. The numbers below assume the opening is on the local +Y side (5.21 mm). If it's on −Y, add 180° to the rotation for CN1, and swap −90 ↔ +90 for CN2.

---

## Option A — keep PA001 (one bottom, one side)

| Ref | Rotation | Centre (x, y) | Courtyard | Wire faces |
|---|---|---|---|---|
| CN1 | 0° | 146.2, 120.7 | x 131.1–161.3, y 117.5–125.9 | bottom edge (down the rocket axis) |
| CN2 | −90° | 127.1, 104.4 | x 121.9–130.3, y 89.3–119.5 | left long edge |

- CN1: move 1.5 mm right of where it is now (it currently touches where CN2 will go).
- CN2: move 10 mm down from the current y 94.5. Keep −90°.
- The pyro zone becomes y 89–126 (37 mm). The free area inside the L is x 130.3–171.4, y 89–117.5 for Q1, Q2, J6, SW3, SW4, C31/C32, JP1 and the fuse.
- The MCU has to move up: U1 centre at y ≤ 74, so the clock pins (8, 12) end up about 15 mm or more from the top of CN2. The top zone (y 51–64) gets tight for Card1 and J5. Check that they fit before committing.
- CN2's wires exit sideways. You need sled/airframe clearance on that side, at least 11 mm for the terminal height plus a wire bend.

## Option B — swap to Micro-Fit 3.0 2×3 right-angle (recommended)

- Each connector is about 12–13 mm wide. Check the exact footprint.
- Both go on the bottom edge between H3 and H4, for example centres x ≈ 137 and ≈ 156. Same rotation, plug mouth facing the bottom edge.
- Same orientation means the pinout is truly identical and the wiring harness is the same for both.
- The pyro zone shrinks to about 22 mm tall. U1 can stay near its current height.
- It's a crimp, locking connector, which is what DTEG B4.2 prefers (the PA001 wires need tie-wrap strain relief instead).

---

## J6 — XT30UPB-F vertical (battery in)

- Mates from the top, and the cable rises off the board. It can sit inside the zone, not on an edge.
- Keep 180° (pin 2 PYRO+ on the left, pin 1 VBAT− on the right).
- PYRO+ (pin 2) points toward the arm/fuse path and the CN PYRO+ pins.
- VBAT− (pin 1) points toward the Q1/Q2 sources and JP1. Put JP1 within 5 mm of J6 pin 1.
- Option A: centre ≈ (138, 110), courtyard x 132.0–143.2, y 106.9–113.1. SW3 and SW4 move to the right half of the zone.
- Option B: centre ≈ (140, 108), just above the two connectors.
- Leave about 20 mm of vertical clearance above J6 for the plug and the cable bend.

## J3 — XT30PW-F horizontal (VBAT+ in)

Problems with it now:
- It's on B.Cu, the only bottom-side part. That conflicts with single-sided assembly.
- It sits at the top-left, in what becomes the quiet end.
- It connects straight to VBAT+ with no diode. If a separate battery is plugged into J3 at a lower voltage than J6, J6 charges it through D3.

Decide what J3 is for first:
- **Not needed:** delete it. Logic runs from J6 through D3.
- **Separate logic battery:** add a series Schottky (or an ideal diode) from J3 to VBAT+. Move it to F.Cu.
  - Placement: mouth flush to an edge in the power zone, next to U2.
  - Option A: right long edge, y ≈ 95–105.
  - Option B: bottom edge beside the two Micro-Fits if they fit (about 35 of 37.6 mm used), otherwise the right edge.
  - The two locating pegs overhang the edge by 2.6 mm. That's correct for edge mounting, so keep the overhang.
