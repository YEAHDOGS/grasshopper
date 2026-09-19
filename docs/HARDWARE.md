# Grasshopper — Hardware (Rev A proposal)

> Concept documentation. Nothing here is final — this is the engineering
> target the industrial design and DFM pass will firm up. All parts below are
> real, in-production components chosen for cost and availability.

## Concept

Grasshopper is a **phone-powered USB-C dongle**, not a standalone gadget.
The phone provides the screen, the compute assist, the battery, and the
network (for signature updates). The dongle is pure RF front-end + DSP.

```
                         ┌──────────────────────────┐
                         │   MULTIBAND ANTENNAS     │
                         │ whip 700MHz–6GHz + PCB   │
                         └────────────┬─────────────┘
                                      │ RF
                         ┌────────────▼─────────────┐
                         │      RF FRONT-END        │
                         │ R820T2 tuner + RTL2832U │  500 kHz – 1.75 GHz
                         │ SX1276 sub-GHz/LoRa     │  137 – 1020 MHz
                         │ ESP32-S3 radio          │  2.4 GHz Wi-Fi/BLE
                         └────────────┬─────────────┘
                                      │ I/Q + IQ-lite
                         ┌────────────▼─────────────┐
                         │   ESP32-S3 (DSP/host)    │
                         │ signal detect + classify │
                         └────────────┬─────────────┘
                                      │ USB 2.0
                         ┌────────────▼─────────────┐
                         │  USB-C (male plug)       │
                         │  OTG: 5 V in, data out   │
                         └────────────┬─────────────┘
                                      │
                         ┌────────────▼─────────────┐
                         │       YOUR PHONE         │
                         │ app · display · battery  │
                         └──────────────────────────┘
```

## Phone attach & power

- **Connector:** USB-C male plug on a short rigid neck; folding flat against
  the dongle for pocket carry. A 15 cm USB-C extension cable ships in the box
  for phones in bulky cases.
- **Power:** phone acts as USB OTG host, supplying 5 V. Onboard buck
  (3.3 V rail) + supercapacitor to ride through brownouts when the RF
  front-end peaks.
- **Power budget (typical):**

  | Block            | Typical | Peak   |
  |------------------|---------|--------|
  | ESP32-S3 + DSP   | 160 mA  | 240 mA |
  | SDR front-end    | 280 mA  | 350 mA |
  | Sub-GHz (RX)     | 20 mA   | 120 mA |
  | Regulators/misc  | 40 mA   | 60 mA  |
  | **Total @ 5 V**  | **~1 W**| ~2.5 W |

  A 10-minute full sweep ≈ **under 5% of a 4,000 mAh phone battery.**
- **OS support:** Android first (USB host API, no drivers — standard USB
  CDC). iPhone via USB-C (15 and later). The app owns all DSP visualization;
  the dongle streams classified detections, not raw I/Q, so even mid-range
  phones keep up.
- **Mount:** spring phone-clip molded into the enclosure; doubles as a
  kickstand for tabletop sweeps.

## Proposed BOM (Rev A)

| # | Part | Function | Supplier |
|---|------|----------|----------|
| 1 | Espressif ESP32-S3-WROOM-1 | Host MCU, 2.4 GHz Wi-Fi/BLE scan, USB OTG, DSP | Digi-Key / Mouser / LCSC |
| 2 | Rafael R820T2 + Realtek RTL2832U | Wideband SDR tuner, 500 kHz–1.75 GHz RX | LCSC / RTL-SDR module vendors |
| 3 | Semtech SX1276 | Sub-GHz / LoRa RX, 137–1020 MHz | Digi-Key / Mouser |
| 4 | Multiband whip antenna, 700 MHz–6 GHz | Wideband RX | Taoglas / Molex via Digi-Key |
| 5 | 2× PCB trace antennas (BLE, sub-GHz) | Dedicated-band RX | PCB fab (JLCPCB) |
| 6 | TI TPS62162 (or equiv.) buck | 5 V → 3.3 V rail | Digi-Key / Mouser |
| 7 | 5.5 V supercapacitor, ~1 F | Brownout ride-through | Digi-Key |
| 8 | USB-C male plug, mid-mount | Phone attach | LCSC / GCT via Mouser |
| 9 | 4-layer PCB, ENIG | Main board | JLCPCB / PCBWay |
| 10 | PC/ABS clamshell + phone clip | Enclosure | Injection mold (TBD) |
| 11 | 15 cm USB-C extension cable | Cased-phone accessory | Off-the-shelf |

**Cost envelope:** RF + silicon ≈ $38–55 at 1k units; enclosure + cable ≈
$9–14; assembly/test ≈ $12–18. Total COGS target **$70–90**, leaving healthy
margin at the $299 / $399 retail targets.

**Pro tier delta:** adds a 5 GHz Wi-Fi radio module (dedicated scan radio)
and extended-band SDR coverage for 2.1 / 2.6 / 3.5 GHz cellular — the two
blocks that push the base BOM past the $299 envelope.

## Band coverage

| Band | Base | Pro | Notes |
|------|------|-----|-------|
| 500 kHz – 1.75 GHz (SDR) | ✓ | ✓ | VHF/UHF bugs, DECT, 433/868/915 ISM |
| 2.4 GHz Wi-Fi / BLE | ✓ | ✓ | ESP32-S3 monitor mode |
| 5 GHz Wi-Fi | – | ✓ | Dedicated radio module |
| 1.8 – 2.7 GHz cellular | – | ✓ | Extended SDR module |
| 3.5 GHz 5G | – | roadmap | Antenna + tuner rev |

## Signature roadmap (devices you haven't thought of yet)

The hardware is a **general-purpose RF ear** — new device classes ship as
free over-the-air signature updates, no new hardware:

- **Zigbee / Z-Wave** — already in-band (2.4 GHz / sub-GHz); signatures only
- **DECT cordless & baby monitors** — 1.9 GHz SDR demod
- **Drones** — 2.4/5.8 GHz video-link fingerprints (DJI OcuSync et al.)
- **RFID/NFC skimmers** — 13.56 MHz coil is a $0.40 BOM add (planned rev B)
- **IR night-vision** — IR photodiode catches camera IR LEDs in dark rooms ($0.20, planned rev B)
- **Smart meters / home chatter** — sub-GHz protocol fingerprints
- **Rogue APs / evil twins** — Wi-Fi frame analysis on-device
- **Laser mics** — optical, out of scope; documented honestly in-app

## Regulatory (planned)

- FCC Part 15 ( unintentional radiator ) for US; CE-RED for EU.
- Receive-only by design — Grasshopper never transmits except BLE
  advertisements during tracker-tag ranging (Pro), keeping certification
  to the simplest class.
