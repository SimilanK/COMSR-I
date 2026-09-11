# Module layout notes — CUHAR tester flight computer

Per-module layout rules taken from the actual datasheets, checked against
`Main board.kicad_pcb`. Status recorded at time of writing.

Board: 50 x 75 mm, 4 layer (F.Cu / In1 GND / In2 POWER / B.Cu), 108 parts.
Density note: the reference board this derives from (CURSR-Vboard) is
79 x 79 mm for 128 parts = 48.8 mm2/part. This board is 34.7 mm2/part,
**29% denser**. Several datasheet keep-outs below are unachievable
because of that, and are recorded as accepted risks rather than fixed.

---

## 1. 5V buck — U2 TPS62901 (RPJ0009A)

Source: TI SLVSFS7A, Figure 7-85 "Layout Example", section 7.5.

Rules:
- VIN, VOUT and GND are **copper pours, not traces**. The figure shows
  planes, not routed connections.
- Input capacitor bridges the VIN and GND pads directly. This loop is the
  dominant EMI source on the board.
- Inductor sits at the SW pin. SW is the highest dV/dt node — keep the
  copper area small; area equals radiation.
- FB/VSET, SS/TR and M/S-CONF exit the opposite face from SW.
- EN and MODE, if tied to input supply, connect **at the input capacitor**,
  not at the IC pin.
- **Thermal vias on the VIN and GND pads.** The RPJ package has no thermal
  pad; it dissipates through its pins.

Status: rebuilt to this topology. U2 at 270 deg.
- input-cap loop 10.82 -> **3.36 mm** (was measuring centre-to-centre,
  which hid the real loop — C27 was oriented with both pads facing away)
- SW pad-to-pad 3.77 mm
- FB-to-SW 3.35 mm (rot 270 chosen over 0/90/180 for this reason)

Outstanding: thermal via array on VIN and GND pads, at routing time.

---

## 2. IMU — U9 ICM-45686 (LGA-14)

Source: TDK-InvenSense AN-000393, AN-000262.

Rules:
- Place where PCB mechanical stress is lowest; away from mounting holes
  and board flex points.
- **Avoid vias, traces and copper pour under the IMU on all layers.**
- Traces to pads as symmetrical as possible (helps self-alignment at reflow).
- NSMD pads preferred over SMD pads.
- Stencil opening 90% of land area.
- VDD decoupling as close as possible to the VDD pin, fanout trace >= 6 mil.
- 3 board anchor points preferred; 2 or >3 not recommended.

Status: U9 at (182.5, 95.0) on U1's east face — SPI1 exits there, so all
five bus nets run straight out with no wrap.
- C17 decoupling loop 2.95 mm, C18 3.85 mm — **worse than ideal**
- nearest neighbours: C10 0.41, R31 0.46, R33 0.44, R34 0.41, U1 1.57 mm

**ACCEPTED RISK:** TDK wants nothing within 2 mm. Enforcing that requires
evicting U1 and CN2, which the board cannot accommodate. Recorded, not fixed.

Actionable at routing time:
- no copper pour or via under the U9 footprint on any layer (zone keep-out)
- symmetrical fanout on the 14 pads
- ground via immediately at the GND pad

---

## 3. Barometer — U3 MS5607

Source: TE/MEAS MS5607 datasheet; TDK AN-000140 for general MEMS pressure
sensor practice (same package physics).

Rules:
- MEMS pressure sensors are affected by package stress. Keep away from
  mounting holes, connectors, and large through-hole parts.
- **No component pads or vias within 2 mm of the package land area** —
  ensures even cooling and minimal mechanical coupling.
- Do not route traces underneath on the layer it sits on.
- Keep away from heat sources (regulators, MOSFETs, high-current devices).
  Minimise thermal gradient across the package.
- Vias near the pad but **not in** the pad — plating causes uneven stress.
- Consider removing solder mask under the package (traps flux/debris).
- Hand soldering not recommended — uneven heat introduces offset bias.

Status: U3 at (156.67, 87.25), west of U1 on the SPI2 face.
- 30+ mm from the buck and from all pyro parts — thermal isolation good
- nearest neighbours: C19 0.48, R27 0.45, U7 0.63, R39 1.03 mm

**ACCEPTED RISK:** same 2 mm rule, same reason. Note the hand-soldering
warning conflicts with the self-assembly plan — this part in particular
wants hotplate reflow, not an iron.

---

## 4. USB + ESD — J5 USB-C, D1 USBLC6-2SC6

Rules:
- ESD device between the connector and everything else, as close to the
  connector as possible. Protection downstream of the thing it protects
  is useless.
- D+/D- routed as a differential pair, ~90 ohm, constant spacing.
- Keep the pair away from the SD bus and from switching nodes.
- Do not cross a plane split with the pair.

Status: D1 2.53 mm from J5 — correct.
- USB_DP 30.8 mm, USB_DN 30.9 mm, **mismatch 0.13 mm** — fine
- pair currently converges from 1.9 mm apart to 0.5 mm; use the
  differential pair router to hold constant spacing
- pair travels from U1's south face to the north edge; J5 is locked on the
  short edge by choice. At Full Speed (12 Mbps) this is acceptable.

---

## 5. Clocks — X1 (32 MHz HSE), Y1 (32.768 kHz LSE TCXO)

Rules:
- These are **oscillator modules, not crystals** — single-ended clock out,
  so OSC_OUT (PH1, PC15) is unused and free for GPIO.
- CubeMX must be set to **BYPASS Clock Source** for both, not
  "Crystal/Ceramic Resonator". Resonator mode will not start reliably.
- Short traces to the MCU. Ground guard either side.
- Keep switching nodes away.

Status: X1 and Y1 both ~13.4 mm from U1 centre, on U1's north face where
HSE_IN/LSE_IN are. Buck is 40+ mm away.
- HSE_IN span 6.3 mm, LSE_IN 5.1 mm — short, good

Actionable: ground guard traces at routing time.

---

## 6. 3.3V LDO — U10 TLV75733

Rules:
- Input and output capacitors close to their pins.
- Ground return short, straight to plane.

Status: C35 4.00 mm, C36 5.32 mm from U10. Both further than ideal.
C1 and C37 are +3V3 bulk and sit 63 mm away — acceptable, they are bulk,
not decoupling.

---

## 7. Pyro — Q1/Q2 SI7232DN, CN1/CN2, R40-R43, C31/C32

Rules:
- Gate resistor belongs **at the gate**, beside the FET, not at the MCU.
- Drain path (firing current) short and wide — 1.0 mm per net class.
- Bulk capacitor near the FET, not near the battery connector: the cap
  supplies the firing surge.
- PYRO_GND kept separate from signal ground; single tie point only.

Status:
- gate resistors now 4.7-7.6 mm from their FETs, on the U1-facing side
  so gate signals do not cross drain signals
- C31 5.9 mm from Q1, C32 8.6 mm from Q2 (were 40.6 and 38.9 mm — the
  bulk caps were stranded at the battery connector, a real defect)
- CN1/CN2 on the east edge, J6 battery below them

Outstanding: PYRO_GND deserves its own region on In2.Cu. The reference
board does this and we had not planned it.

---

## 8. MCU decoupling — U1 STM32U575

Rules:
- One 100 nF per VDD/VSS **pair**, straddling the pair so the loop
  VDD -> cap -> VSS is minimal. Not "near the chip" — at the pin pair.
- Own ground via at each cap.
- Analog supply (+3.3VA, pins 21/22) fed through FB1 ferrite; C12/C13
  belong at those pins specifically.
- VCAP (pin 48) cap is the internal regulator stability cap — ST wants it
  tight.

Status: nine 100 nF caps mapped one-to-one onto the nine VDD/VSS pairs,
each solved for minimum pad-to-pin distance over all four rotations.
- best 1.77 mm per pad, worst 4.11 mm (C34 on VCAP)
- C5, C9, C34 are worse because they share a ground pin with another cap

**Structural limit:** U1's courtyard extends 0.73 mm past the pad ring.
Plus half the cap body plus clearance, the floor is ~1.5 mm per pad.
Getting below that needs caps on the **back side** with a via through —
ruled out by the single-sided assembly decision. That trade was real.

---

## 9. SD card — Card1, R9-R13 pull-ups

Rules:
- Pull-ups (10k on DAT0-3 and CMD) belong near the **card**, not the MCU,
  to keep the stub short. CLK correctly has none.
- CLK is the aggressor: route first, keep away from DAT lines.
- Length matching not required at default SD speeds.

Status: R9-R13 moved to 7.2-9.0 mm from Card1. SD net spans 5.6-26.3 mm,
a 20 mm spread — irrelevant at <=50 MHz (0.13 ns skew vs 20-40 ns period).

Known issue: `SD x baro` is the largest remaining cross-group crossing
count (10). SPI2 is genuinely on U1's west face and Card1 needs an
accessible edge; every rearrangement tested traded these for worse
crossings elsewhere. Left as is deliberately.

---

## 10. Level shifter / servo — U7 SN74AHCT125

Rules:
- 3OE and 4OE are **active-low** enables. Unused channels must have OE
  tied high (disabled) or be driven properly.
- Servo power comes from +5V, not VBAT+. (Fixed earlier — J2/J3 pin 1 was
  originally on VBAT+, which on a 2S pack is 8.4 V into a 4.8-6 V servo.)

Status: PWM1/PWM2 exit U1's east face but U7 sits west near the servo
headers. Two nets wrap. Moving U7 east would make SERVO1/2 wrap instead —
zero net gain, so left alone deliberately.

---

## 11. Debug — J1 TagConnect TC2030-IDC-NL

Rules:
- Pogo-pin footprint: needs clear area for the plug body, not just the
  copper. The footprint models only pads.
- SWO unavailable — PB3 is used for SDMMC2_D2. Use a UART for printf.

Status: J1 has 3.6 mm clearance all round after being grid-searched
separately. Footprint includes the three 0.99 mm locating holes, so a
legged/retaining cable will fit the same pattern.

---

## Cross-cutting items outstanding

1. **Thermal vias** on U2's VIN and GND pads (TI requirement).
2. **PYRO_GND zone** on In2.Cu (copied from the reference board).
3. **Zone keep-out** under U9 and U3 on all layers.
4. **Ground guard** traces beside HSE_IN and LSE_IN.
5. **Differential pair router** for USB, constant spacing.
6. `VBAT-` is in the Battery net class, giving every ground via 0.9 mm
   diameter. The plane carries the current, not the vias — consider
   moving VBAT- to Default (0.6 mm via).
7. **U2 footprint pads are ~0.054 mm oversized** versus TI's land pattern
   (0.254 vs 0.20 for six pads, 0.305 vs 0.25 for three). Manufacturable,
   but it reduces gaps to 0.195-0.246 mm on a leadless part being hand
   soldered. Worth correcting.
