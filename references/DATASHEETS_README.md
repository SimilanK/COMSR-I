# CURSR-V Main Board — Datasheet Links

Corrected against the **current 110-part design** (netlist-verified). The earlier
version of this list was built from an old BOM and contained parts that are not
on this board.

> `datasheets.html` in this folder is the clickable version but is **stale** —
> it still lists the removed parts below. This file is the correct list.

## Removed — not on this board
`MMC5983MA` magnetometer · `ADXL375` high-g accelerometer · `SN74LVC1G32` OR gate ·
`PS1440P02BT` buzzer. Also: **J6 is the battery connector** (Molex 436500201),
not a microSD socket as previously written.

## Added since
`PA001-6P` actuation terminals · `ECS-327TXO-33-TR` LSE oscillator ·
SPDT mode switches · AMASS XT30 for the BEC interface.

---

## MCU & sensors
| Ref | Part | Link |
|---|---|---|
| U1 | STM32U575VGTx | https://www.st.com/resource/en/datasheet/stm32u575ag.pdf |
| U9 | ICM-45686 (IMU, SPI1) | https://www.lcsc.com/datasheet/lcsc_datasheet_2411220643_TDK-InvenSense-ICM-45686_C22459454.pdf |
| U3 | MS5607-02BA03 (baro, SPI2) | https://www.mouser.com/datasheet/2/418/6/ENG_DS_MS5607_02BA03_B3-1134558.pdf |

## Power
| Ref | Part | Link |
|---|---|---|
| U2 | TPS62901RPJR buck | https://www.ti.com/lit/ds/symlink/tps62901.pdf |
| U10 | TLV75733PDBV LDO | https://www.ti.com/lit/ds/symlink/tlv757p.pdf |
| U11, U12 | NRVB120VLSFT1G Schottky (5 V OR-ing) | https://www.onsemi.com/pdf/datasheet/mbr120vlsft1-d.pdf |
| L1 | CKCS4018-1UH/N inductor | https://www.lcsc.com/datasheet/C354573.pdf |
| D3 | SS34 550 mV @ 3 A | https://www.lcsc.com/datasheet/C466502.pdf |
| D2 | MMSZ4683T1G Zener (ADC clamp) | https://www.onsemi.com/pdf/datasheet/mmsz4678t1-d.pdf |

## Actuation
| Ref | Part | Link |
|---|---|---|
| Q1, Q2 | SI7232DN-T1-GE3 dual N-FET | https://www.lcsc.com/datasheet/C144854.pdf |
| CN1, CN2 | **PA001-6P** screw terminal — LCSC **C173288** | https://jlcpcb.com/partdetail/HIWA-PA0016P/C173288 |
| SW3, SW4 | SPDT mode switch — SS-12D00-G3, LCSC **C22355741** | https://www.lcsc.com/product-detail/C22355741.html |
| U7 | SN74AHCT125DR level shifter | https://www.ti.com/lit/ds/symlink/sn74ahct125.pdf |

Rate the mode switch for the **servo** continuous/stall current — the pyro pulse
is carry-only through static contacts.

## Clocks, USB, storage, I/O
| Ref | Part | Link |
|---|---|---|
| X1 | 1XXD32000MBA 32 MHz HSE | https://www.lcsc.com/datasheet/C253694.pdf |
| Y1 | ECS-327TXO-33-TR 32.768 kHz LSE | https://ecsxtal.com/store/pdf/ECS-327TXO.pdf |
| J5 | USB Type-C 16P | https://www.lcsc.com/datasheet/C2927039.pdf |
| D1 | USBLC6-2SC6 ESD | https://www.st.com/resource/en/datasheet/usblc6-2.pdf |
| Card1 | TF-015 microSD socket | https://www.lcsc.com/datasheet/C113206.pdf |
| J6 | Molex 436500201 — **battery connector** | https://docs.rs-online.com/cb53/0900766b815e9ebe.pdf |
| SW1, SW2 | SKRKAEE020 tactile | https://datasheet.lcsc.com/szlcsc/ALPS-Electric-SKRKAEE020_C115357.pdf |
| J1 | TagConnect TC2030-NL (SWD) | needs 3.6 mm pogo clearance — see `MODULE_LAYOUT_NOTES.md` §11 |

## Fab
JLCPCB, 4-layer 1.6 mm, stackup 0.2104 / 1.065 / 0.2104, εr 4.6.
**Inner copper 0.5 oz** — keep PYRO+, firing drains and battery on outer layers.
