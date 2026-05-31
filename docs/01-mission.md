# 01 · Mission & End-to-End Flow

## Vision

A wearable/pocketable **second brain**: a small device that passively captures the audio of your
day, and — through a partner app — turns it into a searchable, structured knowledge base in
**Obsidian**. You don't take notes; the device and app do it for you.

## End-to-end flow

```
  ┌──────────────┐   sound    ┌───────────────┐   USB-C drive   ┌──────────────┐   cloud    ┌────────────┐
  │  Environment │ ─────────► │  The board    │ ──────────────► │  Partner app │ ─────────► │ Transcribe │
  │  (your day)  │  detected  │  records +    │  (Mass Storage) │ (iOS/Android)│   upload   │  + LLM     │
  └──────────────┘            │  compresses + │                 └──────┬───────┘            └─────┬──────┘
                              │  stores audio │                        │                          │
                              └───────────────┘                        ▼                          ▼
                                                              ┌──────────────────┐      ┌────────────────────┐
                                                              │ Generate Obsidian│ ◄────│ transcript + summary│
                                                              │ markdown notes   │      └────────────────────┘
                                                              └──────────────────┘
```

1. **Capture** — voice-activity detection (VAD) triggers recording only when there's sound, to save power and storage.
2. **Compress** — ADPCM (~4:1) keeps files small; speech only needs 8–16 kHz mono.
3. **Store** — written to onboard NAND flash with a wear-leveling filesystem (LittleFS).
4. **Offload** — plug into phone/computer; the device enumerates as a **USB Mass Storage** drive.
5. **Process** — the partner app uploads audio to a cloud transcription service (e.g. Whisper-class),
   then an LLM summarizes/structures it.
6. **Knowledge** — the app writes **Obsidian-compatible markdown** (with frontmatter, links, tags)
   into the user's vault.

## Why "appears as a drive" is the key design choice

Treating the device as a **USB drive** instead of a custom-protocol gadget means:

- **No drivers, no MFi** — works with the iPhone Files app (iPhone 15+ USB-C) and any Android phone.
- **Cross-platform for free** — Mass Storage is universal.
- **Cheaper silicon** — no Bluetooth radio required (see [`02-architecture.md`](02-architecture.md)).

## ⚠️ Legal & privacy considerations (must read)

"Record all the sounds" is technically simple but **legally loaded**:

- Many jurisdictions are **two-party (all-party) consent** for audio recording (e.g. parts of the US,
  much of the EU). Recording conversations without consent can be a **criminal offense**.
- Continuous ambient recording of bystanders raises **GDPR / wiretapping** issues.
- The partner app sends audio to the **cloud** — data-handling, retention, and consent must be explicit.

**Recommendations for a responsible product:**

- Clear, visible **recording indicator** (the status LED) and an easy hardware mute/off.
- **On-device or consented-only** capture modes; default to push-to-record where required.
- Encrypt audio at rest on the flash and in transit.
- A privacy policy and per-jurisdiction guidance shipped with the app.

This repo is an engineering analysis; shipping such a device is the builder's legal responsibility.
