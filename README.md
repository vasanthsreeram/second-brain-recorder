# 🧠 Second-Brain Recorder

> An open-source, single-board, all-SMD **always-on audio recorder** that acts as a *second brain*.
> It records the sounds around you, stores them locally, and — when connected to your phone (iPhone or Android) —
> appears as a **drive**. A partner app then processes the audio in the cloud and turns it into
> structured **Obsidian** notes.

**Status:** 🟡 Design / analysis phase. No board has been fabricated yet. This repo currently
contains the architecture analysis, component selection, power budget, and a 3D layout mockup.

---

## The mission

Build a tiny, cheap, single PCB that:

1. **Listens continuously** and records when sound is detected (voice-activity-triggered).
2. **Compresses** the audio (ADPCM) and **stores it locally** on onboard flash.
3. **Connects to iPhone & Android** and presents itself as a **USB drive** (USB Mass Storage),
   so the recordings can be pulled off without any special driver.
4. A **partner app** uploads the audio for **cloud transcription**, then generates
   **Obsidian** markdown notes — your searchable second brain.

## Hardware at a glance (recommended design)

| Block | Part | Why |
|---|---|---|
| MCU | **RP2040** (QFN-56) | Cheap, native USB Mass Storage, PIO handles the mic — **no radio needed** |
| Storage | **W25N01GV** 128 MB SPI NAND | ~2.5–4.5 h of compressed audio |
| Boot flash | W25Qxx SPI NOR | RP2040 boots from external QSPI flash |
| Microphone | MEMS PDM mic (MSM261) | Digital audio in, all-SMD, no analog circuitry |
| Power | **TP4056** Li-Po charger + **ME6211** 3.3 V LDO | USB-C charge, simple linear rail |
| Connector | USB-C 16P SMD | Charge **and** data (the "drive") over one port |
| Board | ~**22 × 42 mm**, **2-layer** FR-4 | JLCPCB's cheapest standard stack-up |

Estimated parts cost: **~$5–8** per board in small quantities.

> There are **two viable architectures** — a cheaper **USB-only RP2040** board (recommended, above)
> and a **wireless ESP32-S3** board that adds Bluetooth sync. See
> [`docs/02-architecture.md`](docs/02-architecture.md) for the full trade-off.

## Repository layout

```
.
├── README.md                  ← you are here
├── LICENSE                    ← MIT
├── docs/
│   ├── 01-mission.md          ← product mission & end-to-end flow
│   ├── 02-architecture.md     ← RP2040-vs-ESP32-S3 decision, store-and-forward
│   ├── 03-hardware-bom.md     ← components, real dimensions, JLCPCB notes
│   ├── 04-power-battery.md    ← current draw & battery-life analysis
│   ├── 05-storage.md          ← SMD flash vs microSD, capacity & cost
│   ├── 06-connectivity.md     ← USB drive on iPhone/Android, BLE, MFi caveats
│   └── 07-manufacturing.md    ← what JLCPCB needs, next steps, open items
└── hardware/
    └── board-3d.html          ← three.js 3D mockup of the board (open in a browser)
```

## Viewing the 3D mockup

Open [`hardware/board-3d.html`](hardware/board-3d.html) in any modern browser. It renders the
board with all components placed to **real datasheet dimensions**. It is a **cosmetic mockup only** —
it does not simulate any electrical behavior.

## Honest scope & disclaimers

- This is a **design analysis**, not a finished, validated product. Nothing here has been fabricated or bench-tested.
- The schematic still needs to be drawn and the **PCB routed** in KiCad before it can go to a fab.
- "Recording all sounds around you" has **serious legal and privacy implications** — recording
  people without consent is illegal in many jurisdictions. See [`docs/01-mission.md`](docs/01-mission.md).

## License

[MIT](LICENSE) — open source.

## Contributors

- [@vasanthsreeram](https://github.com/vasanthsreeram)
- [@felixisaac](https://github.com/felixisaac)
