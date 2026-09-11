# CURSR-V Main Board — PCB Layout Runbook

Follows the tutorial's page-9 procedure (`references/PCBDesignTutorialRevA.pdf`).
Groups and part lists live in `MODULE_AND_PART_LIST.md`; per-module electrical
rules in `MODULE_LAYOUT_NOTES.md` (§).

**Working arrangement:** the `.kicad_pcb` is edited by the session that has file
access to it. Values, groupings and checks are produced here and handed over as
paste-ready instructions.

---

## STEP 0 — Board setup ✅ essentially closed

| Item | Status |
|---|---|
| Net classes (Pyro/Battery/Servo/Power/USB/Clock) | ✅ set + verified in `.kicad_pro` |
| Constraints (clearance 0.127, track 0.127, edge 0.3, hole 0.25) | ✅ set + verified |
| USB pair 0.25 / gap 0.15 → 90 Ω | ✅ set |
| Grids (2.54 / 1.27 / 0.635 / 0.254 / 0.127) + overrides | ✅ done |
| Board outline 50 × 75 mm | ✅ present |
| Mounting holes H1–H4 | ✅ present |
| Layer order F.Cu / In1=GND / In2=POWER / B.Cu, named | ✅ verified |
| Total thickness 1.6 mm | ✅ verified |
| Stackup dielectric → JLC 0.2104 / 1.065 / 0.2104, εr 4.6 | ⏳ last item, being applied |

**Not verifiable from the file:** copper weight. KiCad stores thickness only —
weight is chosen at order time on JLC's site. Assume **0.5 oz inner** and keep
high current on the outer layers.

---

## STEP 1 — Components on board ✅ done

All **110 footprints** imported and currently sitting outside the outline. That
is the correct starting state — tutorial step 2 is "throw all components onto the
board" before placing anything.

---

## STEP 2 — Stage by group ⏳ current step

Cluster all 110 parts into their functional groups, parked **outside** the
outline and separated from each other, so each block can be placed and routed as
a unit (tutorial steps 3 and 5).

Rule: ~16 parking cells, roughly 50 × 40 mm each, ≥15 mm between cells, ≥20 mm
clear of the outline. Tidiness inside a cell doesn't matter; separation between
cells does. Order the cells Tier A → B → C → D.

Group membership: see `MODULE_AND_PART_LIST.md`.

---

## STEP 3 — Place by tier

Bring groups in one at a time in build order. Rough-place **U1 first** as the
anchor — everything routes to it — even though it completes in Tier B.

- **Tier A** power chain along one edge; **Actuation on its own isolated edge**.
- **Tier B**: decoupling at the pins, clocks by HSE/LSE, sensors + microSD placed
  together to solve the SD-vs-baro crossing, PWM shifter by the PWM pins.
- **Tier C/D**: USB on the short edge, comms/LEDs/debug on remaining edges.

---

## STEP 4 — Route critical tracks first

1. Pour **In1 = GND**, **In2 = POWER**. No zones exist yet — they get created here.
2. **PYRO+ and firing drains** wide and short, on outer copper only; **PYRO_GND**
   its own region on In2.Cu, single tie at JP1.
3. Buck thermal vias; clock ground guards; USB pair over solid ground, no plane
   split crossing.
4. Then SPI, SDMMC, and everything else.

---

## STEP 5 — Finish

Refill zones + stitching vias · teardrops · thermal relief · **silkscreen the pyro
screw terminals** so field wiring can't be mis-landed · DRC to zero · net-by-net
cross-check against the schematic · external review.

---

## Blocking item, carried from the schematic

**`PYRO+` has no arming switch and no fuse.** It runs straight from J6 through D3
to the FETs and connectors. This is the safety interlock and must be added before
the board is built — it is not a layout task, it is a schematic gap.
