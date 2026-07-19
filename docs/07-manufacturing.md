# 07 · Manufacturing & Production Plan (Lint Badge v1.0)

## Development Roadmap (8–12 Weeks)

| Phase | Weeks | Deliverables | Cost (SGD) |
|---|---|---|---|
| **1. Order BOM + prototype PCB** | 1–2 | PCBs from PCBWay, components from LCSC/AliExpress | ~S$80 |
| **2. Basic firmware: audio + SD** | 2–3 | Dual I2S mics → SD recording, VAD | ~S$80 (time) |
| **3. Bluetooth: HFP + BLE** | 3–4 | Call mode + BLE pairing + GATT | ~S$80 (time) |
| **4. Wi-Fi sync** | 4–5 | Encrypted upload to LintLabs | ~S$80 (time) |
| **5. 3D case + magnet integration** | 5–6 | Resin-printed badge, clip, lanyard, magnets | ~S$50 (printing) |
| **6. App MVP** | 6–7 | Flutter companion app (BLE pairing, sync, notes) | ~S$200 (time) |
| **7. 10-unit pilot** | 7–8 | Assemble 10 badges, employee testing | ~S$650 |
| **8. Full production** | 8–12 | 100 units, PCBWay assembly, quality testing | ~S$5,800 |

### Total first run: **~S$6,600** → ROI via employee productivity (meeting notes automated).

## Manufacturing partners

| Step | Partner | What they do |
|---|---|---|
| **PCB fabrication** | PCBWay (Singapore) | 4-layer PCB, custom shape, silkscreen |
| **PCBA (assembly)** | PCBWay (optional) | Pick-and-place + reflow soldering |
| **Components** | LCSC (SG) | Most components, fast SG shipping |
| **Specialty parts** | AliExpress | Magnets, SD cards, LiPo cells |
| **Enclosure** | Local SG 3D print / PCBWay | Resin-printed retro badge, matte black + metallic accents |

## What PCBWay needs (for PCBA)

| File | Source | Notes |
|---|---|---|
| **Gerbers** | KiCad export | Copper, soldermask, silkscreen (one ZIP) |
| **Drill file** | KiCad export | NC drill file (usually included in Gerber ZIP) |
| **BOM** | KiCad export (CSV) | LCSC part numbers for each component |
| **CPL / Pick-and-place** | KiCad export (CSV) | Component positions (X, Y, rotation, ref) |
| **Solder paste stencil** | Optional | For reflow soldering (extra ~$30) |

## Enclosure plan

```
  ┌─────────────────────────────────────────────┐
  │           LINT BADGE — FRONT                │
  │                                             │
  │        ┌─────────┐                            │
  │        │  LINT   │  ← Laser-engraved logo     │
  │        │  BADGE  │                            │
  │        └─────────┘                            │
  │                                             │
  │  [PWR]  [MODE]  [WHISPER]   ← Buttons       │
  │                                             │
  │  ●○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○  ← RGB LED  │
  │                                             │
  │  ──────────────────────────────────────     │
  │  |  MIC 1  |  MIC 2  |  (dual array)       │
  │  ──────────────────────────────────────     │
  └─────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────┐
  │         LINT BADGE — BACK                   │
  │                                             │
  │  ┌─────────────────────────────────────┐    │
  │  │  [Metal plate for magnetic mount]   │    │
  │  │                                     │    │
  │  │    [Neodymium magnet 1]             │    │
  │  │                                     │    │
  │  │    [Neodymium magnet 2]             │    │
  │  └─────────────────────────────────────┘    │
  │                                             │
  │  [USB-C port]  ← Bottom edge               │
  │  [SD card slot] ← Side edge                │
  │  [Lanyard hole]  ← Top edge                │
  └─────────────────────────────────────────────┘
```

### Enclosure specs

- **Material:** Resin (3D printed) — matte black finish, metallic accents
- **Dimensions:** 55 × 35 × 12 mm
- **Weight:** <60 g (with battery)
- **IP rating:** IP54 splash-proof (conformal coating on PCB)
- **Branding:** Laser-engraved "Lint" logo on front
- **Features:**
  - Shirt clip (integrated into enclosure)
  - Lanyard loop (top edge)
  - Strong magnets (2× 10×5 mm neodymium)
  - Metal plate included for non-magnetic surfaces
  - Nameplate slot (optional, for employee ID)

## Pilot plan (10 units)

| Step | Action |
|---|---|
| 1 | Assemble 10 badges (PCBWay PCBA + manual enclosure assembly) |
| 2 | Flash firmware (v0.1: audio + SD + basic BLE) |
| 3 | Distribute to 10 employees (volunteers) |
| 4 | 2-week test: wear in meetings, record calls, sync to app |
| 5 | Collect feedback: audio quality, battery life, ergonomics |
| 6 | Iterate firmware + enclosure based on feedback |

## Full production (100 units)

| Step | Action |
|---|---|
| 1 | Finalize schematic + PCB layout (KiCad) |
| 2 | Order PCBs + components (PCBWay + LCSC) |
| 3 | PCBA (pick-and-place + reflow) |
| 4 | Enclosure (resin printing, 100 units) |
| 5 | Manual assembly: insert PCB into enclosure, attach magnets, clip |
| 6 | Flash firmware (production build) |
| 7 | Quality testing: audio, BT, Wi-Fi, battery, buttons |
| 8 | Ship to employees |

## Timeline summary

```
  Week 1-2:  Order BOM, receive components, prototype PCB
  Week 3-4:  Firmware: audio + SD + VAD
  Week 5-6:  Firmware: HFP + BLE + Wi-Fi sync
  Week 7-8:  Enclosure: 3D print, test fit, iterate
  Week 9-10: App MVP: Flutter companion app
  Week 11-12: 10-unit pilot → feedback → iterate
  Week 13+:  Full production (100 units)
```

## Risks & mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| **BT call quality varies by phone** | High | Test across 5+ devices; fallback to room mode |
| **Battery life <8 hours** | High | 1500 mAh option; aggressive sleep tuning |
| **SD card corruption** | Medium | Wear leveling (internal) + checksum on files |
| **Enclosure cracking** | Medium | Resin quality varies; order from reputable SG printer |
| **HFP not supported on iOS** | Medium | iOS 17+ LE Audio; fallback: room mode recording |
| **Supply chain delays** | Low | LCSC has most items in stock; AliExpress 2–4 weeks |

## Open decisions

- [ ] **PCBWay vs JLCPCB** — both handle 4-layer; PCBWay has better SG support
- [ ] **Resin vs CNC aluminium** — resin (cheaper, faster) vs aluminium (premium feel, $15+/unit)
- [ ] **Conformal coating** — IP54 splash-proof (yes, ~$0.50/unit)
- [ ] **Quality testing fixture** — custom jig for 100-unit testing (~$200 one-time)

## Next steps (immediate)

1. **Order prototype BOM** — LCSC (most parts) + AliExpress (magnets, SD cards, LiPo).
2. **Design schematic in KiCad** — ESP32-C6 + dual I2S mics + ES8311 + SD + PMU.
3. **Route PCB** — 4-layer, 55×35 mm custom outline.
4. **Export Gerbers** — to PCBWay for fabrication.
5. **Start firmware** — ESP-IDF project (skeleton code in `firmware/`).
6. **Start app** — Flutter project (skeleton code in `app/`).
