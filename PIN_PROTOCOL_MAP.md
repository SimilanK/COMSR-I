# COMSR-I Main Board — Pin, Function & Protocol Map

Source: `Main board.kicad_pcb` pad nets (synced to schematic 2026-10-20) + `Mainboard.ioc` (2026-10-10).
MCU: **STM32U575VGT6, LQFP-100**. SYSCLK 160 MHz (PLL from MSI in .ioc).
Ground net in the files is named `VBAT-` (all GND symbols resolve to it).

## 1. MCU pins in use

| Pin | Port | Net | Function | Protocol / setting |
|---|---|---|---|---|
| 7 | PC13 | FIRE_C | Pyro ch C gate (Q2-G1 via R37 1k, pulldown R34 3.3k) | GPIO out — **low-drive pin, see review S4** |
| 8 | PC14 | LSE_IN | 32.768 kHz from Y1 (ECS-327TXO TCXO) | RCC LSE **bypass** |
| 9 | PC15 | FIRE_D | Pyro ch D gate (Q2-G2 via R38 1k, pulldown R33 3.3k) | GPIO out — **low-drive pin, see review S4** |
| 12 | PH0 | HSE_IN | 32 MHz from X1 (KDS DSB211SDN, clipped-sine TCXO) | RCC HSE **analog bypass**; .ioc still says 16 MHz |
| 14 | NRST | nRST | Reset: R1 10k pull-up, C2 100nF, SW1 | — |
| 18 | PC3 | BAT_LEVEL | Battery sense | ADC1_IN4 — **divider not on board** |
| 25 | PA2 | FIRE_B | Pyro ch B gate (Q1-G2 via R36 1k, pulldown R2 3.3k) | GPIO out |
| 26 | PA3 | FIRE_A | Pyro ch A gate (Q1-G1 via R35 1k, pulldown R32 3.3k) | GPIO out |
| 30 | PA5 | SPI1_SCLK | IMU U9 | SPI1 master, mode 3 (CPOL 1, CPHA 2-edge), 8-bit, 2.5 Mbit/s |
| 31 | PA6 | SPI1_MISO | IMU U9 AP_SDO | SPI1 |
| 32 | PA7 | SPI1_MOSI | IMU U9 AP_SDIO | SPI1 |
| 33 | PC4 | USART3_TX | J8 pin 3 (telemetry) | USART3 async |
| 34 | PC5 | USART3_RX | J8 pin 2 | USART3 async |
| 35 | PB0 | ICM_INT_1 | IMU INT1 | EXTI0 |
| 37 | PB2 | ICM_CS | IMU chip select | GPIO out, init high, high speed |
| 40 | PE9 | SERVO1 | CN1 pin 2 | TIM1_CH1 PWM |
| 42 | PE11 | SERVO2 | CN2 pin 5 | TIM1_CH2 PWM (**CH2 not configured in .ioc**) |
| 53 | PB14 | DAT0 | microSD | SDMMC2 4-bit |
| 54 | PB15 | DAT1 | microSD | SDMMC2 |
| 67 | PA8 | MS5607_CS | Baro chip select (R21 47k pull-up) | GPIO out, init high |
| 68 | PA9 | USART1_RX (net) | J7 pin 2 | .ioc = **USART1_TX** → label mismatch |
| 69 | PA10 | USART1_TX (net) | J7 pin 1 | .ioc = **USART1_RX** → label mismatch |
| 70 | PA11 | USB_DN | USB-C via D1 | USB_OTG_FS device |
| 71 | PA12 | USB_DP | USB-C via D1 | USB_OTG_FS device |
| 72 | PA13 | SWDIO | J1 via R4 22Ω | SWD |
| 76 | PA14 | SWCLK | J1 via R3 22Ω | SWD |
| 77 | PA15 | PA15 | LED D5 via R6 2k | GPIO out (LED1) |
| 82 | PD1 | SPI2_SCLK (via R15 22Ω) | Baro U3 | SPI2 master, mode 0, 8-bit, 2.5 Mbit/s |
| 84 | PD3 | SPI2_MISO | Baro U3 SDO | SPI2 |
| 85 | PD4 | SPI2_MOSI (via R16 22Ω) | Baro U3 SDI | SPI2 |
| 87 | PD6 | SDMMC_CLK | microSD | SDMMC2_CK |
| 88 | PD7 | SDMMC_CMD | microSD (R13 10k pull-up) | SDMMC2_CMD |
| 89 | PB3 | DAT2 | microSD | SDMMC2 (SWO lost) |
| 90 | PB4 | DAT3 | microSD | SDMMC2 |
| 92 | PB6 | PB6 | LED D6 via R14 2k | GPIO out (LED2) |
| 94 | PH3-BOOT0 | /Boot0 | R19 10k pull-down, SW2 to 3V3 | — |

Power pins: VBAT(6), VDD(11, 28, 50, 75, 100), VDDUSB(73) = +3V3 · VDDA(22) + VREF+(21) = +3.3VA via FB1 · VSS(10, 27, 49, 74, 99), VSSA(19), VREF-(20) = GND · VCAP(48) = C34 4.7 µF.

**Free pins, normal drive:** PE2–PE8, PE10, PE12–PE15, PE0, PE1, PB1, PB5, PB7–PB10, PB12, PB13, PC0–PC2, PC6–PC12, PD0, PD2, PD5, PD8–PD15, PA0, PA1, PA4, PH1.
**Free ADC-capable (for battery / continuity sense):** PA0, PA1, PA4, PC0, PC1, PC2.

## 2. Peripherals on the board

| Ref | Part | Bus | Notes |
|---|---|---|---|
| U9 | ICM-45686 IMU | SPI1 (mode 3), CS PB2, INT1 → PB0 | VDD + VDDIO = 3V3; INT2/FSYNC pulled up by R31 10k; AUX pins NC |
| U3 | MS5607-02BA03 baro | SPI2 (mode 0), CS PA8 | PS pin → GND = SPI mode |
| Card1 | microSD TF-015 | SDMMC2 4-bit | 10k pull-ups on DAT0–3 and CMD; CD pin not connected |
| J5 | USB-C | USB FS device | CC1/CC2 5.1k (R22/R23) = sink; ESD D1 USBLC6-2; shield NC |
| J1 | TagConnect TC2030-NL | SWD | VCC via R20 100Ω; SWO NC |
| J7 | JST-XH 2-pin | USART1 "companion" | pin 1 USART1_TX net, pin 2 USART1_RX net — **no GND pin** |
| J8 | 1×4 2.54 header | USART3 "telemetry" | 1 +3V3 · 2 RX · 3 TX · 4 GND |
| X1 | KDS 1XXD32000MBA (DSB211SDN) | HSE | 32 MHz clipped-sine TCXO, pins 1/2 GND, 3 out, 4 VCC |
| Y1 | ECS-327TXO-33 | LSE | 32.768 kHz TCXO, pin 1 enable tied high |

## 3. Power tree

```
J6 XT30 (2S, 8.4 V max) ─┬─ PYRO+ (raw battery, no switch, no fuse) ─→ CN1/CN2, SW3/SW4, R40–R43, C31/C32
                         └─ D3 SS34 ─→ VBAT+ ─→ U2 TPS62901 buck (VSET 76.8k, MODE 26.1k) ─→ +5V
J3 XT30PW (B side) ─────────→ VBAT+ (bypasses D3; purpose unknown)
+5V ─ U12 Schottky ─┐
VBUS ─ U11 Schottky ┴─→ U10 TLV75733 LDO (1 A) ─→ +3V3 ─ FB1 ─→ +3.3VA
J2 (2.54 header) ─→ BEC+ ─→ SW3/SW4 pin 1 (servo supply option)
PYRO_GND (FET sources) ─ JP1 ─→ GND   (only link)
```
The +5V rail only feeds the LDO now (U7 and C25 are excluded from the board).

## 4. Pyro / actuation

| Channel | MCU pin | FET | Connector pin (drain) | E-match high side |
|---|---|---|---|---|
| A | PA3 | Q1 ch1 (G1, D1 pins 7/8) | CN1 pin 4 | CN1 pin 6 = PYRO+ |
| B | PA2 | Q1 ch2 (G2, D2 pins 5/6) | CN1 pin 5 | CN1 pin 6 = PYRO+ |
| C | PC13 | Q2 ch1 | CN2 pin 3 | CN2 pin 1 = PYRO+ |
| D | PC15 | Q2 ch2 | CN2 pin 2 | CN2 pin 1 = PYRO+ |

- FETs: SI7232DN dual N-ch, low-side switch, sources on PYRO_GND.
- Gate: 1k series at the FET (R35–R38), 3.3k pull-down on the MCU side (R2, R32–R34).
- R40–R43 1 MΩ from PYRO+ to each drain. Nothing reads them (no continuity sense).
- Firing current = V_batt / R_match, design for ~5 A peak.

**Actual connector pinouts (from PCB) — the two are not the same:**

| Pin | CN1 | CN2 |
|---|---|---|
| 1 | V+ select (SW3: BEC+ or PYRO+) | PYRO+ |
| 2 | SERVO1 (PE9 direct) | FIRE_D drain |
| 3 | GND | FIRE_C drain |
| 4 | FIRE_A drain | GND |
| 5 | FIRE_B drain | SERVO2 (PE11 direct) |
| 6 | PYRO+ | V+ select (SW4) |

`MODULE_AND_PART_LIST.md` lists a different pinout (1 PYRO+, 2 GND, 3 fire, 4 V+, 5 SERVO, 6 fire). It matches neither connector.

## 5. Parts in the schematic but set "not on board"

U7 SN74AHCT125 + C15, R5, R8 (servo level shift) · R27, R28, C30, D2 (battery divider 47k/27k → 3.065 V at 8.4 V) · C25.
That leaves 101 footprints on the PCB out of 110 symbols.
