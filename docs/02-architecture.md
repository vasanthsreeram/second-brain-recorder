# 02 · Architecture & Block Diagram (Lint Badge v1.0)

## High-level block diagram

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                        LINT BADGE v1.0                                │
  │                                                                     │
  │  ┌──────────┐    I2S     ┌──────────┐    I2S     ┌────────────────┐  │
  │  │ MIC 1    │ ─────────► │          │ ─────────► │                │  │
  │  │ICS-43434 │          │          │          │  SD CARD SLOT    │  │
  │  └──────────┘          │  ESP32   │          │  (128 GB U3)   │  │
  │  ┌──────────┐    I2S   │  C6-MINI │          │                │  │
  │  │ MIC 2    │ ─────────► │  -1      │          └────────────────┘  │
  │  │ICS-43434 │          │  (4MB)   │                                  │
  │  └──────────┘          │          │    ┌───────────────────────┐   │
  │                        │          │    │  BLUETOOTH STACK      │   │
  │  ┌──────────┐         │  ES8311  │    │  (HFP Sink + BLE 5.3)│   │
  │  │ ES8311   │ ◄────── │  Codec   │    │                       │   │
  │  │ (optional)│        │          │    │  ◄── Phone/Earbuds   │   │
  │  └──────────┘         └──────────┘    └───────────────────────┘   │
  │         ▲                          │                               │
  │         │ I2C                      │ USB-C                       │
  │  ┌──────┴──────┐                   │ (charge + DFU)              │
  │  │ MAX17048    │                   │                               │
  │  │ Fuel Gauge  │                   │                               │
  │  └──────┬──────┘                   │                               │
  │         │                          │                               │
  │  ┌──────┴──────┐    Buck-Boost    │                               │
  │  │ TPS63101    │ ◄────────────────┘                               │
  │  └──────┬──────┘                                                   │
  │         │ 3.3V Rail                                                │
  │  ┌──────┴──────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐  │
  │  │ 3 Buttons   │  │ WS2812B  │  │ Neodym.  │  │ 1200 mAh     │  │
  │  │ PWR/Mode/   │  │ RGB LED  │  │ Magnets  │  │ LiPo (JST)   │  │
  │  │ Whisper     │  │          │  │ (2×10×5) │  │                │  │
  │  └─────────────┘  └──────────┘  └──────────┘  └────────────────┘  │
  │                                                                     │
  │  Enclosure: 55 × 35 × 12 mm resin (retro badge), IP54               │
  └─────────────────────────────────────────────────────────────────────┘
```

## MCU: ESP32-C6-MINI-1 (why this chip)

The ESP32-C6 is the **only single-chip solution** that gives us everything needed:

| Feature | Why it matters for Lint Badge |
|---|---|
| **Wi-Fi 6 (802.11ax)** | Fast bulk sync of audio files to LintLabs server |
| **BLE 5.3** | Low-energy companion app pairing, live status, OTA updates |
| **802.15.4** | Future-proof for Thread/Zigbee sensor expansion |
| **HFP Sink** | Can pair with phone/earbuds and capture **both** call sides |
| **I2S interface** | Drive dual MEMS mics at 16 kHz / 16-bit |
| **4 MB Flash** | Firmware + small audio buffer (SD handles the rest) |
| **~$8.50** | Affordable for 100-unit pilot |

> **Not ESP32-S3** — the S3 lacks Wi-Fi 6 and its BLE is older. The C6 is the right step up for
> a product that needs both fast Wi-Fi sync and modern BLE.

## Audio architecture

### Path 1: Dual MEMS mics (Room mode — default)

```
  MEMS MIC 1 (ICS-43434) ──I2S──┐
                                ├─► ESP32-C6 ──► ADPCM/PCM ──► SD card
  MEMS MIC 2 (ICS-43434) ──I2S──┘
```

- **Stereo recording** — two mics spaced ~30 mm apart on the badge.
- **ESP-SR noise suppression** — on-device beamforming + environmental noise cancellation.
- **16 kHz, 16-bit mono** (summed stereo) for transcription-quality audio.

### Path 2: Bluetooth HFP (Call mode)

```
  Phone / Earbuds ──HFP──► Lint Badge (HFP Sink)
                              │
                              └─► Captures BOTH sides of the call
                              └─► Streams to SD / LintLabs server
```

- The badge acts as a **Hands-Free Profile (HFP) sink**.
- Phone pairs to the badge instead of (or in addition to) earbuds.
- **Both speaker and microphone audio** captured.
- Tested with Android + iOS (LE Audio on newer devices).

### Path 3: Whisper mode (quick note)

- Long-press the **Whisper button** → single mic, boosted gain.
- 3-second beep confirms activation.
- File tagged `whisper_YYYYMMDD_HHMMSS.wav`.

## Firmware stack (ESP-IDF)

```
firmware/
└── main/
    ├── audio/
    │   ├── mic_i2s.c          ← Dual I2S mic capture (16 kHz, 16-bit)
    │   ├── codec_es8311.c     ← ES8311 codec init (optional, line-in)
    │   ├── vad.c              ← Voice activity detection (on-device)
    │   ├── encoder.c          ← ADPCM / PCM encoder (to SD)
    │   └── noise_suppress.c   ← ESP-SR environmental noise cancellation
    ├── bluetooth/
    │   ├── hfp_sink.c         ← HFP sink for call capture (both sides)
    │   ├── ble_gap.c          ← BLE advertising + GATT server
    │   └── ble_gatt.c         ← GATT services: battery, status, file transfer
    ├── storage/
    │   ├── sd_card.c          ← SD card init, FATFS, file creation
    │   └── file_manager.c     ← Recording lifecycle (create, append, close)
    ├── power/
    │   ├── pmu.c              ← TP4056 charge + TPS63101 buck-boost
    │   ├── fuel_gauge.c       ← MAX17048 battery % via I2C
    │   └── sleep.c            ← Deep sleep / light sleep strategies
    ├── connectivity/
    │   ├── wifi.c             ← Wi-Fi station mode (sync to LintLabs)
    │   └── sync.c             ← Encrypted upload (TLS) + chunked transfer
    ├── ui/
    │   ├── buttons.c          ← PWR / Mode / Whisper (debounce, long-press)
    │   └── led.c              ← WS2812B RGB (status colors)
    ├── modes/
    │   ├── room_mode.c        ← Dual mic, noise suppression, VAD
    │   ├── call_mode.c        ← HFP sink, call detection, recording
    │   └── whisper_mode.c     ← Single mic, boosted gain, quick note
    ├── app/
    │   └── app_main.c         ← Entry point, mode scheduler, event loop
    └── CMakeLists.txt
```

## Sync architecture

```
  Lint Badge ──BLE 5.3──► Lint Voice App (iOS/Android/Mac)
                          │
                          ├─ Live: battery %, recording status, mode
                          └─ Quick: single-file transfer (BLE, <10 MB)

  Lint Badge ──Wi-Fi 6──► LintLabs Server (self-hosted)
                          │
                          ├─ Bulk: 1000+ hours of audio (chunked, TLS)
                          ├─ Transcription: Whisper / Qwen ASR
                          ├─ Summarization: DeepSeek/Grok agent
                          └─ Push: ERPNext / Slack / Notion / Obsidian
```

**Key design: BLE for status + small files, Wi-Fi for bulk.** BLE alone is too slow for 128 GB.

## Decision: why ESP32-C6 over RP2040

| | RP2040 (previous) | **ESP32-C6 (Lint Badge)** |
|---|---|---|
| Radio | ❌ none | ✅ Wi-Fi 6 + BLE 5.3 + 802.15.4 |
| Call recording | ❌ impossible | ✅ HFP sink (both sides) |
| Sync | USB cable only | BLE (live) + Wi-Fi (bulk) |
| Noise suppression | Basic PIO | ✅ ESP-SR (on-device) |
| Storage | 128 MB NAND | ✅ 128 GB SD card |
| Battery life | ~1.5–2.5 days (VAD) | 8–10 hours (continuous) |
| Form factor | Bare PCB | ✅ Badge (55×35×12 mm) |
| Cost | ~$5–8 | ~$48–62 |
| Complexity | Low | Moderate (radio + audio + power) |

The C6 adds meaningful complexity but delivers a **complete, polished product** that replaces
the need for a dedicated phone app for call recording.

## Open decisions

- [ ] **ES8311 vs TLV320AIC3104** — codec choice (both work, ES8311 is cheaper)
- [ ] **SD card vs eMMC** — SD is swappable (128 GB, $4.20); eMMC is soldered (64 GB, $3–8)
- [ ] **E-ink display** — optional small e-ink for quick notes on the badge ($8–12)
- [ ] **Battery size** — 1200 mAh (default) vs 1500 mAh for margin
- [ ] **Firmware: ESP-IDF vs Arduino** — ESP-IDF for full control; Arduino for rapid prototyping
