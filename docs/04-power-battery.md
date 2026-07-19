# 04 · Power & Battery-Life Analysis (Lint Badge v1.0)

## Usable capacity

A "1200 mAh" Li-Po does not deliver 1200 mAh of useful energy:

- Don't discharge below ~3.0 V (ES8311 + ESP32-C6 cutoff) → lose ~15–20%.
- Buck-boost converter (TPS63101) efficiency: **~92–95%** (vs LDO ~89%).

**Plan on ~950–1000 mAh usable** from a 1200 mAh cell.

## Current draw by mode (ESP32-C6)

| Mode | What it's doing | Avg current |
|---|---|---|
| **Deep sleep** | CPU off, BLE advertiser, mic off | ~0.05–0.3 mA |
| **Light sleep** | BLE connected, CPU idle, mic standby | ~5–10 mA |
| **Listening (Room mode)** | Dual I2S mics + ESP-SR VAD | ~35–50 mA |
| **Recording to SD** | Capture + encode + SD write (no BT) | ~55–75 mA |
| **Call mode (HFP)** | BT active + both sides captured | ~80–120 mA |
| **Wi-Fi sync** | Bulk upload to LintLabs server | ~150–250 mA (brief bursts) |
| **BLE pairing** | Companion app connection | ~10–20 mA |

> SD card write bursts can spike to **100–150 mA** — the TPS63101 handles this fine.

## Runtime estimates (1000 mAh usable)

| Usage pattern | Runtime |
|---|---|
| **Continuous Room mode recording** | **~13–18 hours** |
| **Continuous Call mode (HFP)** | **~8–12 hours** |
| **Mixed: 2 h call + 4 h room + 2 h standby** | **~8–10 hours** (full workday) |
| **Voice-activated, active ~15% of time** | **~2–3 days** |
| **Mostly deep sleep, button-to-record** | **~1–2 weeks** |

**The 1200 mAh battery targets 8+ hours** — a full workday of mixed use. This is the key
requirement for an employee badge.

## Charging

| Source | Current | Time to full (0–100%) |
|---|---|---|
| **Wall charger (5 V / 1 A)** | TP4056 set to 500 mA | ~2.5 hours |
| **Phone USB-C (USB PD)** | 300–500 mA (negotiated) | ~3–4 hours |
| **PC USB 2.0 (5 V / 0.5 A)** | 500 mA max | ~2.5 hours |
| **Wireless pad (optional)** | TBD — add Qi receiver | ~3 hours |

**TP4056 charge current** is set by a single resistor — calculate for 500 mA (fast but safe).

## Power management strategy

```
  ┌─────────────────────────────────────────────────────────┐
  │                    POWER MANAGEMENT                     │
  └─────────────────────────────────────────────────────────┘

  Battery (3.7V, 1200mAh)
       │
       ▼
  ┌──────────┐     5V from USB     ┌──────────────┐
  │ TP4056   │ ◄────────────────── │ USB-C Port   │
  │ Charger  │                     │              │
  └────┬─────┘                     └──────────────┘
       │ 3.7–4.2V (charging)
       ▼
  ┌──────────────┐
  │ MAX17048     │
  │ Fuel Gauge   │ ──I2C──► ESP32-C6 (battery % → BLE)
  └────┬───────┘
       │ 3.7–4.2V
       ▼
  ┌──────────────┐     3.3V rail
  │ TPS63101     │ ─────────────────────────────────┐
  │ Buck-Boost   │                                  │
  └──────────────┘                                  │
                                                    ▼
  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐
  │ ESP32-C6 │  │ ES8311   │  │ MEMS x2  │  │ SD Card Slot │
  └──────────┘  └──────────┘  └──────────┘  └──────────────┘

  Deep sleep modes:
  - Deep sleep: ~0.1 mA (RTC + BLE advertiser on)
  - Light sleep: ~5 mA (BLE connected, CPU clock-gated)
  - Active recording: ~55–120 mA (depends on mode)
```

## Levers to extend runtime

1. **Aggressive sleep between recordings** — the single biggest lever (10× impact).
   - VAD-based: only wake CPU when sound detected above threshold.
   - Timed wake: if no sound for 30 seconds, enter light sleep.
2. **Bigger battery** — 1500–2000 mAh barely changes the board (just connector/wire).
3. **Buck-boost (TPS63101)** — ~92–95% vs LDO ~89%. Already included in BOM.
4. **Disable Wi-Fi when not syncing** — Wi-Fi radio draws ~50–80 mA even idle.

## Battery life vs storage (they're independent)

| Storage | 16 kHz mono (record) | 16 kHz stereo (room mode) |
|---|---|---|
| 128 GB SD card | **~330 hours** (~14 days continuous) | **~165 hours** (~7 days continuous) |
| 64 GB SD card | **~165 hours** (~7 days) | **~83 hours** (~3.5 days) |

**Key insight:** The 1200 mAh battery (8–10 hours) will drain **long before** the 128 GB SD fills up.
The badge needs Wi-Fi sync to offload audio regularly — otherwise the SD fills and you lose recordings.

## Charging from a laptop / phone

- USB-C port handles **both charge and data** (DFU / firmware updates).
- iPhone 15+/Android USB-C can host-power the device.
- iPhone supplies limited power (~4.5 W); set TP4056 charge current to ~300–500 mA.
- **USB-C CC1/CC2:** 5.1 kΩ pull-down resistors for sink negotiation.

## Open decisions

- [ ] **Battery size** — 1200 mAh (default, 8–10 hrs) vs 1500–2000 mAh for margin
- [ ] **Wireless charging** — add Qi receiver module (~$1.50) for pad charging
- [ ] **Solar option** — tiny solar cell on badge face (negligible contribution, novelty)
