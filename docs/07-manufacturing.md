# 07 · Manufacturing & Next Steps

## What JLCPCB actually needs

To order from JLCPCB you upload a specific set of files — **not** a schematic:

| Service | Files required |
|---|---|
| **Bare PCB** | **Gerbers** (copper, soldermask, silkscreen) + **drill file** (usually one ZIP) |
| **Assembly (PCBA)** | also a **BOM** (with LCSC part numbers) + **CPL / pick-and-place** file |

These are **exported from KiCad** *after* the board is laid out and routed.

## Current state of this project

- ✅ Architecture, component selection, power budget, storage, connectivity — **analyzed** (this repo).
- ✅ 3D layout mockup — `hardware/board-3d.html` (cosmetic only).
- ⬜ Schematic — **not yet drawn**.
- ⬜ PCB layout / routing — **not yet done**.
- ⬜ Gerber / BOM / CPL export — **not yet generated**.
- ⬜ Firmware — **not yet written**.

## The realistic build path

1. **Prototype firmware first** on dev boards (RP2040 + I²S/PDM mic breakout, ~$13) or in **Wokwi**
   (browser ESP32/RP2040 simulator). Prove: VAD → ADPCM → write to flash → USB Mass Storage.
   *This is the real "simulation" — SPICE cannot test firmware, USB, or RF.*
2. **Draw the schematic** — via the `circuit-synth` skill (Python → KiCad) or directly in KiCad.
3. **Route the PCB** in KiCad (the one step that genuinely needs a human): 2-layer, ground pour,
   short USB/flash traces, mic acoustic port, decoupling placement.
4. **Run ERC + DRC** in KiCad — the checks that actually prevent dead boards (use JLCPCB design rules).
5. **Export** Gerbers + drill (+ BOM + CPL for PCBA) and upload to JLCPCB.

## Tooling

- **`circuit-synth`** skill — installed; defines circuits in Python and generates a KiCad project
  (schematic + netlist). Requires KiCad's symbol libraries (`brew install --cask kicad`).
- **KiCad** — for routing, ERC/DRC, and the Gerber/CPL export.
- **Wokwi** / dev boards — for firmware validation before fabrication.

## Open decisions

- [ ] **RP2040 (USB-only) vs ESP32-S3 (wireless)** — recommended RP2040; confirm if wireless sync is needed.
- [ ] **128 MB vs 256 MB NAND** — capacity vs ~$1.
- [ ] **Battery size** — 1000 mAh (~1.5–2.5 days) vs 1500–2000 mAh for margin.
- [ ] **Audio quality** — 8 kHz (more hours) vs 16 kHz (better transcription).
- [ ] Firmware codec — ADPCM (cheap) vs Opus (smaller files, more CPU).
