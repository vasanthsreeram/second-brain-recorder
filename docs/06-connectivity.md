# 06 · Connectivity: The "Drive" on iPhone & Android

The device offloads audio by **enumerating as a USB Mass Storage device** — i.e. it shows up as a
**drive**. This is the cross-platform, driver-free, license-free path.

## What works on each platform

| Method | iPhone (15+, USB-C) | Android | Notes |
|---|---|---|---|
| **USB Mass Storage** (shows as a drive) | ✅ **Files app**, natively | ✅ natively | No app/driver/MFi needed. The chosen path. |
| USB Audio Class (live mic) | ✅ | ✅ | live audio, not stored files — not our use case |
| **Custom USB protocol → your own app** | ❌ needs Apple **MFi** (expensive, HW-locked) | ⚠️ possible | avoid on iOS |
| Bluetooth LE → your app | ✅ (ESP32 design only) | ✅ | slower; wireless convenience |

**Takeaway:** present the recordings as a **drive** (USB MSC). On iPhone they appear in the **Files
app**; on Android they mount like any USB stick. The partner app reads files from there (share sheet /
file picker) and uploads for transcription.

## Power + data at the same time

When the phone is USB host, it puts 5 V on the cable and the board is the USB peripheral. The board
can **charge its battery and act as a USB drive simultaneously** — standard USB behavior.

- iPhone supplies limited power (~4.5 W); keep TP4056 charge current modest (~300–500 mA).
- USB-C **CC1/CC2 each need a 5.1 kΩ pull-down** so the device negotiates as a powered sink.

## Transfer speed

- **USB** (RP2040/ESP32 native USB, Full-Speed 12 Mbps): ~1 MB/s effective → a full 128 MB dump in ~2 min.
- **BLE** (ESP32 only): ~50–700 kbps effective → much slower; fine for trickle sync, not bulk.

USB is the fast path; BLE (if present) is the wireless-convenience path.

## Why this removes the need for Bluetooth

Because both iPhone and Android read a USB drive natively, the **radio is optional**. That is exactly
why the recommended design drops the ESP32-S3 in favor of the cheaper, radio-less **RP2040**
(see [`02-architecture.md`](02-architecture.md)).

## The MFi caveat (important)

If you ever want **your own iOS app to talk to the device directly over USB** (not via the Files app),
that requires Apple's **MFi** program — costly and hardware-locked, not realistic for a cheap DIY
board. Stick to: **USB = drive (Files app)**, **BLE = your app** (ESP32 design only).
