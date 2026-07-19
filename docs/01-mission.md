# 01 · Mission & End-to-End Flow (Lint Badge v1.0)

## Vision

A **badge-form wearable** that passively captures audio from meetings and calls, then turns it into
structured, searchable notes — all processed privately through your LintLabs backend. Employees clip it
on, wear it on a lanyard, or mount it magnetically on a laptop/whiteboard.

## End-to-end flow

```
  ┌──────────────┐   sound    ┌───────────────┐   BLE          ┌──────────────┐   Wi-Fi    ┌────────────┐
  │  Environment │ ─────────► │  Lint Badge   │ ────────────► │  Lint Voice  │ ─────────► │ Transcribe │
  │  (meeting)   │  detected  │  records +    │  (pairing)    │  Companion   │  (upload)  │  + LLM     │
  └──────────────┘            │  compresses + │               │  (iOS/Android│            └─────┬──────┘
                              │  stores audio │               │  /Mac)     │                  │
                              └───────────────┘               └──────┬──────┘                  ▼
                                                                  │                       ┌──────────────────────┐
                                                                  ▼                       │  Summary + To-Dos    │
                                                            ┌──────────────────┐            │  (ERPNext / Slack /  │
                                                            │ Generate Notes   │ ◄──────────│  Dashboard / Obsidian)│
                                                            │ (frontmatter,    │            └──────────────────────┘
                                                            │  links, tags)    │
                                                            └──────────────────┘
```

### Recording modes

| Mode | Trigger | Audio path | Best for |
|---|---|---|---|
| **Call mode** | BLE HFP pair to phone/earbuds | Both sides captured via earbud mic | 1-on-1 calls, remote meetings |
| **Room mode** | Dual I2S MEMS mics (default) | Stereo, directional pickup | Group discussions, meeting rooms |
| **Whisper mode** | Long-press hardware button | Single mic, boosted gain | Quick notes, solo voice memos |

### AI processing (zero-subscription)

1. **Transcription** — self-hosted Whisper / Qwen model on your LintLabs server.
2. **Summarization** — your DeepSeek/Grok agent produces key points, action items, decisions.
3. **Push** — sends structured notes to ERPNext, Slack, Notion, Obsidian, or your internal dashboard.

## Why badge form?

| | RP2040 "Second-Brain" (previous) | **Lint Badge (v1.0)** |
|---|---|---|
| Form | Small PCB, no enclosure | **Badge** — shirt-clip + lanyard + magnets |
| Audio | Single PDM mic, basic | **Dual studio I2S mics** + codec, noise reduction |
| Calls | ❌ USB-only, no BT | **HFP sink** — captures both call sides |
| Sync | USB cable only (Mass Storage) | **BLE (live status) + Wi-Fi (bulk sync)** |
| Storage | 128 MB NAND (~2.5 h) | **128 GB SD** (1000+ hours) |
| Battery | ~1.5–2.5 days (voice-activated) | **8–10 hours continuous** |
| Features | Basic recording | **3 modes, RGB LED, 3 buttons, e-ink option** |
| Cost | ~$5–8 | ~$48–62 (prototype ~$80) |

The badge adds meaningful complexity but delivers a **complete, employee-ready product**.

## ⚠️ Legal & privacy considerations (must read)

Recording conversations without consent is **illegal in many jurisdictions** (two-party / all-party
consent laws in the US, EU, etc.).

**Built-in safeguards:**

- **Visible recording indicator** — WS2812B RGB LED (red = recording, green = standby).
- **Hardware mute** — power button long-press kills audio path physically.
- **Local-first storage** — audio stays on the badge until explicitly synced.
- **End-to-end encryption** — encrypted uploads to your LintLabs server.
- **Consent mode** — app can prompt meeting participants before recording starts.

This repo is an engineering analysis; shipping such a device is the builder's legal responsibility.

## Repository layout

```
docs/
├── 01-mission.md          ← you are here: product mission & modes
├── 02-architecture.md     ← ESP32-C6 block diagram, firmware stack
├── 03-hardware-bom.md     ← full 14-component BOM with links
├── 04-power-battery.md    ← ESP32-C6 current draw + 1200 mAh analysis
├── 05-storage.md          ← SD card / eMMC (128 GB+), 1000+ hours
├── 06-connectivity.md     ← BLE 5.3 pairing + Wi-Fi 6 sync
└── 07-manufacturing.md    ← PCBWay assembly, 3D resin case, timeline
```

## Contributors

- [@vasanthsreeram](https://github.com/vasanthsreeram)
- [@felixisaac](https://github.com/felixisaac)
