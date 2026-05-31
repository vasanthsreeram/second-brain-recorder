# 03 · Hardware & Bill of Materials

All components are **SMD** and chosen to be cheap and **JLCPCB-stockable** (so JLCPCB PCBA
assembly is possible). Dimensions are **real datasheet values**.

## Recommended design (RP2040, USB-only)

| Ref | Part | Function | Package | Body L×W×H (mm) | ~Unit cost |
|---|---|---|---|---|---|
| U1 | **RP2040** | MCU (USB, PIO mic, ADPCM) | QFN-56 | 7.0 × 7.0 × 0.9 | ~$0.90 |
| U2 | **W25N01GV** | 128 MB SPI NAND — audio storage | SOIC-8 | 5.3 × 5.3 × 1.5 | ~$1.80 |
| U3 | **W25Q16** (or 2 MB) | RP2040 boot/QSPI flash | SOIC-8 | 5.3 × 4.0 × 1.5 | ~$0.30 |
| MIC1 | **MSM261** PDM MEMS mic | digital audio in | bottom-port | 3.5 × 2.65 × 0.98 | ~$0.50 |
| U4 | **TP4056** | Li-Po charge management | ESOP-8 | 3.9 × 4.9 × 1.5 | ~$0.20 |
| U5 | **ME6211C33** | 3.3 V LDO | SOT-23-5 | 1.6 × 2.9 × 1.1 | ~$0.10 |
| Y1 | 12 MHz crystal | RP2040 clock | 3225 SMD | 3.2 × 2.5 × 0.8 | ~$0.10 |
| J1 | **USB-C 16P** (e.g. C165948) | charge + USB drive | SMD | 8.94 × 7.25 × 3.26 | ~$0.30 |
| SW1/2 | tactile switches | reset / user | SMD 3×4 | 3.9 × 2.9 × 1.7 | ~$0.05 |
| D1 | LED | status / recording indicator | 0805 | 2.0 × 1.25 × 0.7 | ~$0.02 |
| R/C | resistors + caps | decoupling, pull-ups, CC, charge-set | 0805 | 2.0 × 1.25 × 0.7 | cents |

**Estimated total parts cost: ~$5–8** in small quantities (the NAND + RP2040 dominate).

## Alternative design (ESP32-S3, wireless)

Swap U1/U3/Y1 for a single module:

| Ref | Part | Function | Package | Body (mm) | ~Unit cost |
|---|---|---|---|---|---|
| U1 | **ESP32-S3-WROOM-1** | MCU + BLE + RAM + flash (incl. antenna) | module | 18.0 × 25.5 × 3.1 | ~$2.50–3.50 |

The module includes its own clock and boot flash, so the crystal and boot-flash chip are removed.
Everything else (NAND, mic, charger, LDO, USB-C, buttons, LED, passives) is identical.

## Board

- **Size:** ~22 × 42 mm (RP2040) / ~22 × 50 mm (ESP32-S3, module is bigger).
- **Layers:** **2** — sufficient because there is no external antenna and no fast parallel bus.
  Keep a solid ground pour on the bottom layer under the MCU and decoupling caps.
- **Stack-up:** 1.6 mm FR-4, 1 oz copper — JLCPCB's cheapest standard tier.

## Layout cautions (for the routing step)

- Decoupling caps **right next to** each power pin.
- The MEMS mic needs an **acoustic port** (a hole in the PCB for bottom-port mics) and a small
  acoustic seal; keep it away from noisy switching nodes.
- SD/NAND and USB lines kept short; USB D+/D− routed as a pair.
- USB-C **CC1/CC2 each need a 5.1 kΩ pull-down** so the device is recognized as a powered sink.
