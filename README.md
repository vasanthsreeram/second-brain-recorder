# 🏷️ lintbadge

> A **badge-form, employee-ready AI voice recorder** — clip-on/lanyard, retro-futuristic polish.
> Bluetooth call recording, dual studio mics, 8+ hours battery, seamless sync to Lint app/ERPNext.
> Zero-subscription, private AI processing via LintLabs backend.

**Status:** 🟡 Pivot to v1.0 — full development & production plan. Firmware skeleton + companion app skeleton ready.

---

## The mission

Build a **badge-form wearable** that employees clip on or wear on a lanyard in meetings/calls:

1. **Records** via Bluetooth HFP (both sides of a call), dual room mics (group discussions), or whisper mode (long-press).
2. **Stores locally** on SD/eMMC (1000+ hours of audio).
3. **Pairs via BLE** to the Lint Voice companion app (iOS/Android/Mac) for live status + quick sync.
4. **Syncs via Wi-Fi** to your self-hosted LintLabs backend (Whisper transcription → summary → ERPNext/Slack).
5. **Zero monthly fees** — all core AI runs on your own servers.

## Hardware at a glance (Lint Badge v1.0)

| Block | Part | Why |
|---|---|---|
| MCU | **ESP32-C6-MINI-1** (4MB Flash) | Wi-Fi 6 + BLE 5.3 + 802.15.4; HFP sink for calls |
| Audio | **Dual TDK ICS-43434** I2S MEMS + **ES8311** codec | Studio-grade, noise reduction ready |
| Storage | **MicroSD slot + 128 GB U3** (or 64 GB eMMC) | 1000+ hours recording |
| Battery | **3.7 V 1200 mAh** LiPo (JST-PH, PCM protected) | 8–10 hrs continuous |
| PMU | **TP4056** + **MAX17048** fuel gauge + **TPS63101** buck-boost | Smart charging + battery % to app |
| Connectivity | BLE (pairing) + Wi-Fi (bulk sync) | Dual-mode, no cable needed |
| Form factor | **55 × 35 × 12 mm** badge (shirt-clip + lanyard + magnets) | Retro-futuristic Lint branding |
| Extras | WS2812B RGB LED + 3 tactile buttons (PWR/Mode/Whisper) | Status + controls |

Estimated parts cost: **~$48–62** per unit (100 pcs: ~$48/unit, prototype ~$80).

## Repository layout

```
.
├── README.md                  ← you are here
├── LICENSE                    ← MIT
├── docs/
│   ├── 01-mission.md          ← product mission & end-to-end flow
│   ├── 02-architecture.md     ← ESP32-C6 architecture (BLE + Wi-Fi + HFP)
│   ├── 03-hardware-bom.md     ← full Lint Badge BOM (14 components)
│   ├── 04-power-battery.md    ← power analysis for ESP32-C6 + badge form
│   ├── 05-storage.md          ← SD card / eMMC storage (128 GB+)
│   ├── 06-connectivity.md     ← BLE pairing + Wi-Fi sync (replaces USB-only)
│   └── 07-manufacturing.md    ← PCBWay assembly, 3D resin case plan
├── firmware/                  ← ESP-IDF firmware skeleton
│   └── main/                  ← audio, BT, BLE, Wi-Fi, storage, power, buttons, modes
├── app/                       ← Flutter companion app skeleton
│   └── lib/                   ← BLE pairing, sync, meeting notes
└── hardware/
    └── board-3d.html          ← three.js 3D mockup (badge form factor)
```

## Viewing the 3D mockup

Open [`hardware/board-3d.html`](hardware/board-3d.html) in any modern browser. It renders the
badge with all components placed to **real datasheet dimensions**. It is a **cosmetic mockup only** —
it does not simulate any electrical behavior.

## Honest scope & disclaimers

- This is a **design analysis + skeleton code**, not a finished, validated product. Nothing here has been fabricated or bench-tested.
- The schematic still needs to be drawn and the **PCB routed** in KiCad before it can go to a fab.
- "Recording audio" has **serious legal and privacy implications** — recording
  people without consent is illegal in many jurisdictions. See [`docs/01-mission.md`](docs/01-mission.md).
- Bluetooth HFP call recording quality depends on phone/BT stack compatibility — test across devices.

## License

[MIT](LICENSE) — open source.

## Contributors

- [@vasanthsreeram](https://github.com/vasanthsreeram)
- [@felixisaac](https://github.com/felixisaac)
