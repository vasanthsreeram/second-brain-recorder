# 06 · Connectivity: BLE + Wi-Fi (Lint Badge v1.0)

## Dual-mode connectivity

The Lint Badge uses **both BLE and Wi-Fi** — each for a different purpose:

| Protocol | Role | Speed | Range | When active |
|---|---|---|---|---|
| **BLE 5.3** | Companion app pairing, live status, quick file transfer | 125 kbps – 2 Mbps | ~30 m | Always (advertising) + when paired |
| **Wi-Fi 6** | Bulk sync of audio files to LintLabs server | 50–500 Mbps | ~30 m | On-demand (when Wi-Fi available) |

## BLE 5.3 — Companion app pairing

```
  Lint Badge (BLE Peripheral)  ◄──►  Lint Voice App (BLE Central)
  ─────────────────────────────        ──────────────────────────
  GATT Services:                  GATT Services:
  ┌──────────────────────┐        ┌──────────────────────┐
  │ Service: Badge Status │        │ Service: Badge Status │
  │ ─────────────────────│        │ ─────────────────────│
  │  char: battery_level  │ ◄──── │  Subscribe to notify  │
  │  char: recording_mode │ ◄──── │  Subscribe to notify  │
  │  char: battery_pct    │ ◄──── │  Subscribe to notify  │
  │  char: sd_capacity    │ ◄──── │  Subscribe to notify  │
  └──────────────────────┘        └──────────────────────┘
  ┌──────────────────────┐        ┌──────────────────────┐
  │ Service: File Transfer│        │ Service: File Transfer│
  │ ─────────────────────│        │ ─────────────────────│
  │  char: file_list      │ ◄──── │  Request file listing │
  │  char: file_chunk     │ ◄──── │  Request chunk (≤200B)│
  └──────────────────────┘        └──────────────────────┘
  ┌──────────────────────┐        ┌──────────────────────┐
  │ Service: Control      │        │ Service: Control      │
  │ ─────────────────────│        │ ─────────────────────│
  │  char: set_mode       │ ────► │  Write: room/call/    │
  │  char: delete_file    │ ────► │  Write: filename      │
  │  char: factory_reset  │ ────► │  Write: confirm       │
  └──────────────────────┘        └──────────────────────┘
```

### BLE characteristics

| Characteristic | Direction | Size | Update rate |
|---|---|---|---|
| `battery_level` | Badge → App | 1 byte | Every 5 s |
| `recording_mode` | Badge → App | 1 byte | On mode change |
| `battery_pct` | Badge → App | 1 byte (0–100) | Every 10 s |
| `sd_capacity` | Badge → App | 2 bytes (used/total MB) | Every 30 s |
| `file_list` | Badge → App | Variable | On request |
| `file_chunk` | Badge → App | 200 bytes max | On request |
| `set_mode` | App → Badge | 1 byte | On user action |
| `delete_file` | App → Badge | 32 bytes (filename) | On user action |

### Pairing

- **BLE pairing** — simple numeric comparison (like AirDrop) or just connect with a shared
  passphrase (stored in badge NVS + app).
- **No MFi needed** — BLE is open.
- **Pairing persists** — badge remembers last paired device (stored in NVS flash).

## Wi-Fi 6 — Bulk sync to LintLabs

```
  Lint Badge (Wi-Fi Station)  ──TLS──►  LintLabs Server (self-hosted)
  ──────────────────────────           ──────────────────────────
  Steps:
  1. Scan for known Wi-Fi networks (saved in NVS)
  2. Connect to first available (WPA2/WPA3)
  3. Authenticate to LintLabs (API key / token)
  4. Upload completed recordings (chunked, TLS 1.3)
  5. Delete locally after successful upload
  6. Disconnect, return to BLE advertising
```

### Wi-Fi sync details

| Parameter | Value |
|---|---|
| **Protocol** | HTTPS (TLS 1.3) |
| **Auth** | API key in header or JWT token |
| **Chunk size** | 1 MB per POST (to avoid timeouts) |
| **Speed** | ~1–5 MB/s on 2.4 GHz (typical office) |
| **Full 128 GB sync** | ~7–35 hours (realistic: do partial syncs) |
| **Power** | ~150–250 mA during sync (battery drains in ~5–7 h) |

### Sync strategy

```
  On power-on / wake:
  1. Check if known Wi-Fi is available
  2. If yes → connect, sync completed recordings, disconnect
  3. If no → stay on BLE only (no sync)

  On app connection (BLE):
  1. App shows "Upload to Wi-Fi?" button
  2. User taps → badge connects to Wi-Fi, uploads, returns to BLE

  Manual sync:
  1. Long-press Mode button → badge auto-connects to Wi-Fi
  2. Uploads all unmarked recordings
  3. Marks them as "synced" (or deletes if configured)
```

## Bluetooth HFP — Call recording (both sides)

This is the **killer feature** of the Lint Badge: capturing **both sides** of a phone call.

```
  Phone / Earbuds  ──HFP──►  Lint Badge (HFP Sink)
  ─────────────────        ───────────────────────
  What's captured:
  - Speaker audio (what the other person says)
  - Microphone audio (what you say)
  - Both mixed or separate channels

  How it works:
  1. Phone pairs to Lint Badge via BLE (HFP profile)
  2. Badge takes over audio routing (phone audio → badge)
  3. Badge records both channels to SD card
  4. Badge streams to LintLabs server (via Wi-Fi)
```

### HFP implementation

- **ESP-IDF BT stack** — `esp_bt_hfp_client` or custom HFP sink implementation.
- **Tested on:** Android (works well), iOS (limited, LE Audio required).
- **Quality:** Depends on phone's BT stack — test across devices.
- **Fallback:** If HFP not supported, use room mode (dual mics) as backup.

## Connectivity summary

| Feature | BLE 5.3 | Wi-Fi 6 | HFP (BT) |
|---|---|---|---|
| Pairing | ✅ Simple numeric | ❌ N/A (station mode) | ✅ Phone pairs to badge |
| Live status | ✅ Battery, mode, SD | ❌ N/A | ❌ N/A |
| File transfer | ✅ Small files (<10 MB) | ✅ Bulk (128 GB) | ❌ N/A |
| Call recording | ❌ N/A | ❌ N/A | ✅ Both sides captured |
| Sync to LintLabs | ❌ (BLE too slow) | ✅ TLS, chunked | ✅ (via Wi-Fi) |
| Power draw | ~5–20 mA | ~150–250 mA | ~80–120 mA |

## Open decisions

- [ ] **HFP stack** — ESP-IDF built-in vs custom (Android works, iOS needs LE Audio)
- [ ] **Wi-Fi security** — WPA2 vs WPA3 (office networks vary)
- [ ] **Sync frequency** — automatic (every charge cycle) vs manual (app-triggered)
- [ ] **Offline mode** — badge works fully without network (files on SD, sync later)
