# 03 · Hardware & Bill of Materials (Lint Badge v1.0)

## Full BOM — 14 components (SG-friendly, AliExpress / LCSC)

Prices approximate **July 2026**, bulk discounts applied for 100+ units. Includes 10% buffer.

| # | Component | Part / Spec | Qty | Unit (S$) | Total (S$) | Supplier | Notes |
|---|---|---|---|---|---|---|---|
| 1 | **MCU Module** | ESP32-C6-MINI-1-N4 (4MB Flash) | 1 | 8.50 | 8.50 | AliExpress / LCSC | Core + BT 5.3 for call routing |
| 2 | **Main PCB** | Custom 4-layer (or Waveshare ESP32-C6 Audio Base) | 1 | 12.00 | 12.00 | PCBWay (SG) | Custom badge shape + magnet pads |
| 3 | **Microphones** | Dual TDK ICS-43434 I2S MEMS (studio-grade) | 2 | 3.80 | 7.60 | AliExpress | Superior to S3 stock mics; noise reduction ready |
| 4 | **Audio Codec** | ES8311 or TLV320AIC3104 | 1 | 2.50 | 2.50 | LCSC | Clean I2S + echo cancellation |
| 5 | **Storage** | MicroSD slot + 128 GB U3 card (or 64 GB eMMC) | 1 | 4.20 | 4.20 | AliExpress | 1000+ hours recording |
| 6 | **Battery** | 3.7 V 1200 mAh LiPo (JST-PH, PCM protected) | 1 | 6.50 | 6.50 | AliExpress | 8–10 hrs continuous |
| 7 | **PMU / Charger** | TP4056 + MAX17048 fuel gauge + TPS63101 buck-boost | 1 | 3.80 | 3.80 | LCSC | Smart charging + battery % to app |
| 8 | **Magnetic Back** | Neodymium magnets (2× 10×5 mm) + adhesive metal plate | 1 set | 1.20 | 1.20 | AliExpress | HiDock-style mounting |
| 9 | **Buttons** | Tactile switch kit (PWR + Mode + Whisper) | 3 | 0.40 | 1.20 | LCSC | Physical controls |
| 10 | **LEDs** | WS2812B RGB (status) + power LED | 3 | 0.30 | 0.90 | AliExpress | Visual feedback |
| 11 | **Enclosure** | Custom 3D-printed resin case (retro badge) | 1 | 8.00 | 8.00 | Local SG print / PCBWay | Polished, clip + lanyard |
| 12 | **Antenna** | Onboard PCB antenna (or external if needed) | 1 | — | — | Included | Wi-Fi/BT |
| 13 | **Connectors** | USB-C, JST battery, I2C expansion | 1 set | 2.00 | 2.00 | LCSC | Programming + future sensors |
| 14 | **Misc** | Passives + solder mask + capacitors + resistors | 1 kit | 2.50 | 2.50 | LCSC | Standard kit |
| | | | | | | | |
| **TOTAL** | | | | | **~61.90** | | Prototype: S$80; 100 pcs: S$48/unit |

## Cost breakdown

| Scenario | Cost |
|---|---|
| **1 Prototype** | ~S$80 (parts + shipping + resin printing) |
| **10 Units (pilot)** | ~S$650 (S$65/unit incl. assembly) |
| **100 Units (production)** | ~S$5,800 (S$48–55/unit incl. PCBWay assembly) |
| **Tooling (case mold + PCB)** | ~S$450 one-time (amortised over production) |

## Board layout targets

- **Dimensions:** 55 × 35 mm (badge shape, rounded corners)
- **Layers:** **4-layer** FR-4 (RF traces for Wi-Fi/BT need controlled impedance)
- **Stack-up:** 1.6 mm, 2 oz copper for power traces
- **Shape:** Custom outline matching the resin enclosure (not a rectangle)
- **Magnet pads:** Copper pads on back for neodymium magnet adhesion
- **USB-C:** Bottom edge, centered
- **Mic array:** Top edge, 30 mm apart (stereo spacing)
- **SD card slot:** Side edge, accessible without disassembly

## Component placement notes

1. **ESP32-C6 module** — center of board, RF antenna area clear (no components within 5 mm).
2. **Dual I2S MEMS mics** — top edge, 30 mm apart, acoustic ports facing outward.
3. **ES8311 codec** — near mics, short I2S traces (<20 mm).
4. **SD card slot** — side edge, mechanical clearance for card insertion.
5. **1200 mAh LiPo** — mounted on back (or internal slot in enclosure).
6. **Neodymium magnets** — 2× 10×5 mm on back, epoxy-fixed.
7. **USB-C connector** — bottom edge, centered, CC resistors close.
8. **WS2812B RGB** — front face, visible through enclosure window.
9. **Buttons** — accessible through enclosure cutouts.

## Alternative components (if supply issues)

| If unavailable, substitute with: |
|---|
| ESP32-C6-MINI-1 → **ESP32-C6-WROOM-1-N4** (larger module, same pinout) |
| TDK ICS-43434 → **INMP441** I2S MEMS (cheaper, slightly lower SNR) |
| ES8311 → **TLV320AIC3104** (TI alternative, similar specs) |
| 128 GB SD card → **64 GB SD** (half capacity, same slot) |
| 1200 mAh LiPo → **1500 mAh LiPo** (slightly larger, more runtime) |

## Manufacturing notes

- **PCBWay** handles PCB fabrication + optional PCBA (pick-and-place + solder reflow).
- **LCSC** is the preferred component supplier (SG shipping, fast, stock items).
- **AliExpress** for specialty items (magnets, SD cards, LiPo cells).
- **Local SG resin printing** for enclosure (Formlabs or local 3D print service).
- Order **10% extra components** for assembly defects / rework.

## Open decisions

- [ ] **ES8311 vs TLV320AIC3104** — codec choice (both work, ES8311 is cheaper)
- [ ] **SD card vs eMMC** — SD is swappable (128 GB, $4.20); eMMC is soldered (64 GB, $3–8)
- [ ] **Battery size** — 1200 mAh (default) vs 1500 mAh for margin
- [ ] **E-ink display** — optional small e-ink for quick notes on the badge ($8–12)
