# 02 · Architecture & The Big Decision

## Store-and-forward, not live-stream

Early thinking was "stream audio live over Bluetooth." We moved to **store-and-forward**: record to
local flash, then bulk-transfer when connected. This is strictly better here:

| | Live streaming | **Store-and-forward (chosen)** |
|---|---|---|
| Radio/power | radio on constantly = battery killer | radio off while recording → big battery win |
| Reliability | drop connection = lost audio | audio safe on flash, retransmit anytime |
| Range | must stay near phone | record anywhere, sync later |
| Capacity | limited by connection time | hours of audio on cheap flash |

## The MCU decision: RP2040 (USB-only) vs ESP32-S3 (wireless)

The single biggest cost/complexity driver is **whether the device needs a radio**. Because the
device offloads by **appearing as a USB drive**, Bluetooth is *optional*.

| | **RP2040 (recommended)** | ESP32-S3 |
|---|---|---|
| Wireless | ❌ none — must plug in | ✅ BLE sync + USB |
| USB Mass Storage | ✅ native (TinyUSB) | ✅ native |
| Audio capture | ✅ PIO does PDM/I²S; ADPCM easy | ✅ I²S + lots of compute |
| RF / antenna design | none (no radio) | handled by certified module |
| Unit cost (silicon) | ~$0.80–1.00 | ~$2.50–3.50 (module) |
| Board complexity | simpler, smaller (~22×42 mm) | larger (module dominates) |
| Best when… | offloading by cable is fine | you want wireless sync too |

### Decision

**Recommended: RP2040, USB-only.** The mission says the device *"connects to iPhone/Android and
puts it in the drive"* — that's a USB-Mass-Storage workflow, which works on both platforms without a
radio. RP2040 is ~1/3 the silicon cost, simpler to route (no RF), and smaller.

**Keep ESP32-S3 if** wireless background sync (no cable) becomes a hard requirement later. The
firmware concepts are the same; only the offload transport differs.

## Block diagram (RP2040 design)

```
USB-C ──┬─► CC resistors (sink) ──► VBUS
        │
        ├─► TP4056 Li-Po charger ──► battery ──┐
        │                                      ├─► ME6211 3.3V LDO ─► 3V3 rail
        │                          (battery) ──┘
        │
        └─► USB D+/D- ──► RP2040 ──(QSPI)── boot NOR flash
                            │
                            ├──(SPI)──► W25N01 128MB NAND  (audio storage)
                            ├──(PIO/I²S)──► MEMS PDM mic
                            ├──► status LED
                            └──► buttons (reset / user)

  RP2040 enumerates over USB as MASS STORAGE → phone sees a drive.
```

## Firmware responsibilities (same for either MCU)

1. **VAD** — analyze incoming mic samples; start recording above a sound threshold, stop after silence.
2. **Compress** — ADPCM encode (cheap, ~4:1).
3. **Filesystem** — LittleFS on NAND with wear leveling; one file per recording, timestamped.
4. **USB MSC** — when plugged in, expose the recordings as a FAT/USB drive.
5. **Power management** — sleep aggressively between sound events; this is the #1 battery lever.

> Note: prototype the firmware on **off-the-shelf dev boards** (RP2040 + I²S mic breakout, ~$13 total),
> or in the browser via **Wokwi**, *before* committing to a custom PCB. That is the realistic
> "simulation" — SPICE cannot test firmware/USB/RF behavior.
