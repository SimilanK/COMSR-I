# CURSR-V Main Board — Placement Group Spec (for placement agent)

Derived from `MODULE_AND_PART_LIST.md` + measured PCB geometry. This is the
input contract for whatever agent does the actual rough-placement pass —
it turns the schematic's *build-order* tiers into a *spatial* plan
(zones, adjacency, keepouts) without assuming placement order = build order.

**Do not confuse Tier with placement priority.** The source doc says Tiers
are "the order blocks are completed, not placed" — MCU (U1, Tier B) is
explicitly called out as an early anchor despite completing later. Treat
Tier as a routing/verification sequence, separate from the zone-carving
order below.

---

## 1. Fixed geometry (not up for negotiation)

- Board outline: rectangle, **50 mm × 75 mm**. Corners at
  (121.361218, 51.4) / (171.361218, 126.4) in the current file's coordinate
  space (re-derive if the file is re-saved with a different origin — don't
  hardcode this if the outline moves).
- 4 mounting holes, one per corner: **H1–H4** (group A0). These are fixed
  points, not placeable — everything else works around them.
- **Keepout radius around each mounting hole: TBD** — no washer/hardware
  clearance spec was in the part list. Placeholder **3 mm** radius is
  reasonable for M3 hardware but must be confirmed before finalizing, not
  just assumed.
- Copper-to-edge clearance: **0.3 mm** (from net-class constraints — this
  governs copper, not component body clearance; component-to-edge margin
  should be set separately, typically ≥1 mm for JLCPCB tolerance).

## 2. Functional zones and edge affinity

Each row is one placement zone. "Edge affinity" = which side of the board
(if any) the zone should be biased toward, driven by mechanical/connector
needs, not by area-fitting alone.

| Zone | Parts (count) | Edge affinity | Why | Adjacency requirement |
|---|---|---|---|---|
| **A1 Battery input** (3) | J6 D3 JP1 | Board edge (connector) | J6 is a physical connector | Feeds A2 + A5; keep short path to both |
| **A1.2 Battery sense** (4) | R27 R28 C30 D2 | near A1 | Senses battery net directly off J6 | Adjacent to A1, not A2 |
| **A2 5V buck** (10) | U2 L1 C27 C28 C29 R24 R25 R26 C25 C26 | interior, near A1 | Input-cap loop must stay tight (source note §1) | Input side faces A1; output side faces A3/A5 |
| **A3 3V3 LDO path** (7) | U10 U11 U12 C35 C36 C1 C37 | interior, downstream of A2 | Regulator chain | Between A2 and B1 (MCU is the main 3V3 load) |
| **A4 BEC interface** (2) | J2 J3 | board edge (connector) | Physical connectors | Near A2/A5 depending on which rail they tap |
| **A5 Actuation** (20) | Q1 Q2 CN1 CN2 SW3 SW4, R35-38, R2/R32-34, R40-43, C31 C32 | **isolated board edge** | Source explicitly calls for "isolated edge, wide short drain paths" (§7); this is the pyro/high-current group | Own ground region on In2.Cu, single tie point at JP1 — keep physically separable from everything else |
| **B1 MCU core** (14) | U1 C3-C9 C34 FB1 C12-C14 R19 | interior, center-ish | Anchor component — placed early, completes late | Hub: touches B2, B3, B4, B5, C1, C2, D1, D2 |
| **B2 Clocks** (4) | X1 C10 Y1 C11 | tight to U1 | "short with ground guards" (§5) | Directly adjacent to B1, nothing routed between |
| **B3 Sensors** (9) | U9 C17 C19 R31, U3 C18 R14, R15 R16 | interior, quiet corner | No copper/vias under U9/U3 (§2, §3) — needs clean ground beneath | Near B1 (SPI2), away from A5 (noise) and A2 (switching) |
| **B4 microSD** (6) | Card1 R9-R13 | board edge (card slot) | Physical card access | **Route in same pass as B3** — SDMMC CLK is the aggressor, SD×baro is worst crossing (§9). Placement should keep this trace path short, not just the group boundary |
| **B5 PWM level shift** (4) | U7 C15 R5 R8 | between B1 and A5 | Bridges MCU logic to actuation | Sits on the signal path from B1 to A5 |
| **C1 USB** (4) | J5 D1 R22 R23 | board edge (connector) | Physical connector; D1 ESD **must be at the connector**, not just in the group | Diff pair routed after placement — keep J5→D1→U1 path straight |
| **C2 Comms** (2) | J7 J8 | board edge (connector) | Physical connectors | Near B1 (UART pins) |
| **D1 LEDs & buttons** (13) | D4-D7, R21 R29 R39 R44, SW1 SW2, R1 C2 R7 | user-facing edge/top | Human-operated | Low routing priority, fill remaining space |
| **D2 Debug** (4) | J1 R3 R4 R20 | board edge, **reserve early** | J1 TagConnect needs 3.6 mm clear for pogo body (§11) — physical space must be reserved even though routing is trivial | Reserve footprint-plus-clearance zone before other interior groups claim that area |

## 3. Placement pass order (not the same as Tier)

1. Lock A0 (mounting holes) and edge-cuts — these are the immovable frame.
2. Reserve D2 (J1 TagConnect) clearance early — explicit instruction in
   source doc, easy to accidentally box in later.
3. Place A5 (Actuation) on its isolated edge first — it has the strongest
   "must not touch anything else" constraint (own ground region, high
   current).
4. Place A1 → A2 → A3 as a chain (battery in → buck → LDO), since each
   feeds the next.
5. Anchor B1 (MCU) — even though it's a Tier B completion item, place it
   early since nearly everything else (B2, B3, B4, B5, C1, C2) is defined
   relative to it.
6. Place B2 immediately adjacent to B1 (tight coupling, no negotiation).
7. Place B3 + B4 together, in a quiet interior corner away from A2/A5 —
   they're routed as one pass per §9.
8. Place remaining connector groups (A4, C1, C2) against their required
   edges.
9. Fill D1 (LEDs/buttons) in whatever space remains near the user-facing
   side.

Area-proportional zone sizing (not equal sectioning) should still apply
within this order — A5 (20 parts) and B1 (14 parts) need meaningfully more
area than A4 (2 parts) or C2 (2 parts); don't carve equal cells and force
groups into them.

## 4. Open items — resolve before (or flag during) layout

These are from the source doc's "Open items" section, restated as
placement-relevant flags:

- **No arm switch / fuse in PYRO+** — the safety interlock doesn't exist
  in the schematic yet. This isn't a placement problem, but the A5 zone
  should leave physical room for it in case it's added before board
  finalization. Flag, don't silently ignore.
- **C25 / C26** are provisionally in A2 (buck). If they actually belong at
  U7 (+5V, in B5) or at USB VBUS (C1), they should move zones. Don't place
  them as permanently-A2 without checking net association first.
- **Naming collision**: capacitor **C2** (in D1, LEDs & buttons) is not the
  same thing as group code "C2" (Comms). Any agent consuming this spec
  should refer to groups by name ("Comms", "LEDs & buttons"), never by
  code, to avoid exactly this ambiguity.

## 5. What this spec deliberately leaves to the placement agent

- Exact X/Y coordinates within each zone (local placement).
- Rotation of individual footprints.
- Final zone boundary polygons (this doc gives adjacency/affinity intent,
  not pixel-perfect rectangles).
- Actual courtyard-area sums per group (should be computed from the real
  footprints before sizing zones, not estimated from part count alone).
