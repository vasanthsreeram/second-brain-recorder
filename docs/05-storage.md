# 05 · Storage: SD Card / eMMC (Lint Badge v1.0)

## Decision: MicroSD card (swappable) vs soldered eMMC

The Lint Badge needs **massive storage** — 1000+ hours of audio. Soldered flash chips
(max ~256 MB) are inadequate. Two options:

| | **MicroSD card (chosen)** | **Soldered eMMC** |
|---|---|---|
| Matches "badge" form | ✅ swappable card, user-replaceable | ✅ no moving parts |
| Size / cost | ✅ SD slot ~15×14 mm, 128 GB ~$4.20 | BGA-153, 64 GB ~$3–8 |
| Reliability | ⚠️ card can be lost/swapped | ✅ no contacts to wear |
| Capacity | ✅ **128 GB–1 TB** (up to 330 h audio) | 4–64 GB practical |
| Get data out | ✅ pull card, read on PC | only via USB/BLE/Wi-Fi |
| Firmware | FATFS on SD card | FATFS on eMMC |
| Badge aesthetics | ⚠️ card slot visible on side | ✅ clean, no slots |

**Recommendation: MicroSD card.** Swappability is a feature for employees who want to:
- Swap cards between badges (hand-off).
- Read recordings on a PC without the badge.
- Replace cards independently of the badge hardware.

## Options & cost

| Card | Capacity | Speed Class | ~Price | Audio recording |
|---|---|---|---|---|
| SanDisk Ultra | 128 GB | U3, A1 | ~$4.20 | **~330 hours** (mono 16 kHz) |
| SanDisk Extreme | 256 GB | U3, A2 | ~$8.00 | **~660 hours** (mono 16 kHz) |
| SanDisk Ultra | 64 GB | U1 | ~$2.50 | **~165 hours** (mono 16 kHz) |
| eMMC 5.1 | 64 GB | — | ~$5.00 | **~165 hours** (mono 16 kHz) |

## File system & firmware

### File system: FAT32 (via LwIP + FATFS)

- **FAT32** — universally readable on any OS (Windows, macOS, Linux, Android).
- **FATFS** — lightweight FAT implementation for ESP-IDF (built into ESP-IDF).
- **Max file size:** 4 GB (FAT32 limit) → ~8 hours of stereo audio per file.
- **Filename scheme:** `REC_YYYYMMDD_HHMMSS_MODE.wav` (mode = room/call/whisper).

### Recording lifecycle

```
  Button press / VAD trigger
       │
       ▼
  1. Create new file on SD card
       │
       ├── Filename: REC_20260719_143022_room.wav
       ├── Header: WAV (RIFF) + PCM/ADPCM format
       └── Start writing I2S samples
       │
       ▼
  2. Write audio chunks (every 100 ms = 1.6 KB @ 16 kHz / 16-bit mono)
       │
       ├── Check SD write status (block vs page writes)
       ├── Update file length in header (post-write)
       └── Rotate to new file at 4 GB boundary
       │
       ▼
  3. Close file (flush buffers, update FAT)
       │
       └── Mark as "synced" when uploaded to LintLabs
```

### Wear considerations

SD cards have limited write cycles (~10k–100k per block). Mitigations:

1. **Wear leveling** — handled by SD card controller (internal).
2. **File rotation** — new file every 4 GB → spreads writes across card.
3. **Sync-and-delete** — after Wi-Fi upload, delete local file → frees space.
4. **Over-provisioning** — use 128 GB card for ~64 GB of actual audio → 50% spare.

**Realistic lifespan:** At 2 hours of recording per day, a 128 GB card lasts **years** before
wear becomes a concern.

## SD card slot pinout (ESP32-C6)

```
  SD Card Pin   ──►  ESP32-C6 GPIO
  ─────────────        ─────────────────
  CMD            ──►  GPIO 47  (SPI command)
  CLK            ──►  GPIO 14  (SPI clock)
  DAT0           ──►  GPIO 48  (SPI data 0)
  DAT1           ──►  (not used, SD 1-line mode)
  DAT2           ──►  (not used)
  DAT3           ──►  (not used)
  VDD            ──►  3.3V rail
  VSS            ──►  GND
  CD/DAT3        ──►  GPIO 46  (card detect, optional)
  WP             ──►  (write protect switch, optional)
```

**1-line SPI mode** (CMD + DAT0) reduces GPIO usage. Full 4-line mode is faster but uses more pins.

## Storage vs battery — the key relationship

| Battery | 1200 mAh (8–10 hours recording) |
|---|---|
| 128 GB SD card | **~330 hours** (mono 16 kHz continuous) |

The battery drains **~33× faster** than the SD fills. This means:

- **Wi-Fi sync is mandatory** — you'll drain the battery long before the SD fills.
- **Sync frequency:** Every 8–10 hours (each charge cycle), upload recorded files.
- **Auto-sync:** Badge connects to known Wi-Fi, uploads completed recordings, deletes locally.

## Open decisions

- [ ] **SD card capacity** — 128 GB (default) vs 256 GB for margin
- [ ] **SD card vs eMMC** — SD is swappable; eMMC is cleaner but less capacity
- [ ] **File rotation size** — 4 GB (FAT32 limit) vs smaller (more files, more overhead)
- [ ] **Encryption at rest** — encrypt audio files on SD (AES-128 via ESP32 Crypto)
