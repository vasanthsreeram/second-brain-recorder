# 05 · Storage: SMD Flash vs microSD

## Decision: soldered SMD NAND flash

The mission requires **all-SMD, single-board**. A microSD socket is a bulky **mechanical connector**;
a soldered flash chip is a tiny IC and a better fit.

| | **SMD flash (chosen)** | microSD socket |
|---|---|---|
| Matches "all SMD" | ✅ perfectly | ⚠️ it's a connector |
| Size / cost | ✅ smaller, cheaper | bigger (~15 × 14.5 mm) |
| Reliability | ✅ no moving parts/contacts | contacts wear |
| Capacity | fixed (≤256 MB practical) | ✅ huge & swappable (GBs) |
| Get data out | only via USB/BLE | ✅ pull card, read on PC |
| Firmware | flash FS (LittleFS) | FAT on card |

Since the workflow is "offload via the USB drive," removability isn't needed — soldered flash wins.

## Options & cost

| Chip | Capacity | Package | ~Price | Compressed audio |
|---|---|---|---|---|
| W25Q128 (NOR) | 16 MB | SOIC-8 | ~$0.80–1.20 | ~15–30 min |
| **W25N01GV (NAND)** ⭐ | 128 MB | SOIC-8 | ~$1.50–2.50 | ~2.5–4.5 h |
| W25N02KV (NAND) | 256 MB | SOIC-8 | ~$2.50–3.50 | ~5–9 h |
| eMMC | 4–8 GB | BGA-153 | ~$3–8 | days (but BGA = harder/costlier assembly) |

## "Storage in the chip itself"?

Using the MCU's own built-in flash:

- **RP2040** has *no* internal flash — it always needs an external QSPI chip (so storage is external regardless).
- **ESP32-S3** modules include 8–16 MB flash. Bumping 8→16 MB costs ~$0.30–0.50, but **16 MB ≈ only
  15–30 min** of audio and is shared with firmware — too small.

So real storage needs a dedicated chip either way. **Adding it costs ~$1.50–$2.50** (the 128 MB NAND).

## Chip lifespan

NAND endures ~100k erase/program cycles per block. With wear leveling (LittleFS) and a few hours of
audio per day, that is **many years** — not a practical limit.

## Recommendation

**W25N01GV, 128 MB SPI NAND (~$2):** best balance of capacity, cost, and easy SOIC-8 assembly.
Step up to 256 MB (~$1 more) if you want longer recording before a forced offload.
