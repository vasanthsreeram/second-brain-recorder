# 04 · Power & Battery-Life Analysis

## Usable capacity

A "1000 mAh" Li-Po does not deliver 1000 mAh of useful energy:

- Don't discharge below ~3.3 V → lose ~10–15%.
- Linear LDO loss (battery ~3.7 V → 3.3 V) ≈ **89% efficient**.

**Plan on ~800 mAh usable** from a 1000 mAh cell.

## Current draw by mode

(RP2040 design; no radio. ESP32-S3 BLE sync is higher but only during sync.)

| Mode | What it's doing | Avg current |
|---|---|---|
| Deep sleep | waiting on button, mic off | ~0.05–0.3 mA |
| **Listening (VAD)** | mic on, analyzing for sound | ~12–20 mA |
| **Recording to flash** | capture + ADPCM + NAND write (radio off) | ~35–55 mA |
| USB offload | enumerated as drive, transferring | ~40–70 mA (USB-powered anyway) |
| *(ESP32 only)* BLE sync | wireless transfer | ~70–100 mA, brief |

NAND flash writes draw only a few mA — far less than a microSD card's 50–100 mA write bursts, which
is one reason soldered flash is the better fit.

## Runtime estimates (800 mAh usable)

| Usage pattern | Runtime |
|---|---|
| Continuous recording to flash | **~15–20 hours** |
| Voice-activated, active ~10–20% of the time (realistic) | **~1.5–2.5 days** |
| Mostly deep sleep, button-to-record only | weeks |

**The battery is the limiting factor for daily use, not storage.**

## Recording capacity vs battery (they're independent)

| Flash | 8 kHz voice (~8 KB/s) | 16 kHz voice (~16 KB/s) |
|---|---|---|
| 128 MB (W25N01) | ~4.5 h | ~2.3 h |
| 256 MB (W25N02) | ~9 h | ~4.5 h |

So a typical loop: record a few hours of speech → battery lasts ~1.5–2.5 days → plug in to offload
and clear the flash → repeat.

## Levers to extend runtime

1. **Firmware sleep behavior** — the single biggest lever (10× more impact than hardware choices).
   Aggressive deep sleep between sound events turns 8 hours into 2 days.
2. **Bigger battery** — 1500–2000 mAh barely changes the board (just the connector/wire).
3. **Buck/boost instead of LDO** — ~92–95% vs ~89%; modest gain, more cost. Not worth it for "cheap."

## Charging from a phone

iPhone 15+/Android USB-C can host-power the device, but supply is limited (~4.5 W on iPhone).
Set the **TP4056 charge current to ~300–500 mA** so you don't exceed it. A wall charger should be
the primary charging method; phone-charging is a convenience, not the plan.
