# Grasshopper — Hardware (Rev A proposal)

> Concept documentation. Nothing here is final — this is the engineering
> target the industrial design and DFM pass will firm up. All parts below are
> real, in-production components chosen for cost and availability.

## Concept

Grasshopper is a **battery-powered handheld sweeper**. It runs solo for
hours on its own cell — the phone is an optional display, not a requirement.
Snap it onto your phone over USB-C and it can sip phone power instead of its
own battery, running indefinitely.

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
                                      │
                    ┌─────────────────┴──────────────────┐
                    │                                    │
         ┌──────────▼──────────┐              ┌──────────▼──────────┐
         │  1,500 mAh Li-Po    │              │    YOUR PHONE       │
         │  USB-C charging     │              │  app · display      │
         │  ~6 hr sweep        │              │  optional: BLE or   │
         └─────────────────────┘              │  USB-C (power+data) │
                                              └─────────────────────┘
```

## Power

- **Primary:** internal 1,500 mAh Li-Po — ~6 hours of continuous sweeping.
- **Charging:** USB-C, 5 V in. Full charge in ~90 minutes.
- **Phone power (optional):** snap it onto your phone over USB-C and it runs
  off phone power instead of its own cell — indefinitely. The phone can also
  top up the internal cell while attached.
- **Phone link (optional):** BLE for the app display when wireless, or USB-C
  for data + power when docked. The dongle classifies on-device; the phone is
  just a screen.
- **Power budget (typical):**

  | Block            | Typical | Peak   |
  |------------------|---------|--------|
  | ESP32-S3 + DSP   | 160 mA  | 240 mA |
  | SDR front-end    | 280 mA  | 350 mA |
  | Sub-GHz (RX)     | 20 mA   | 120 mA |
  | Regulators/misc  | 40 mA   | 60 mA  |
  | **Total @ 5 V**  | **~1 W**| ~2.5 W |

  A 10-minute full sweep ≈ **under 3% of the internal cell.**
- **OS support:** Android first (BLE, no drivers). iPhone via BLE; USB-C
  (15 and later) for docked power + data. The dongle streams classified
  detections, not raw I/Q, so even mid-range phones keep up.
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
| 7 | 1,500 mAh Li-Po pouch cell | Internal battery, ~6 hr sweep | PKCell / LCSC |
| 8 | TI BQ24075 | USB-C charge controller | Digi-Key / Mouser |
| 9 | Maxim MAX17048 | Fuel gauge | Digi-Key / Mouser |
| 10 | USB-C receptacle, mid-mount | Charging + optional phone dock | LCSC / GCT via Mouser |
| 11 | 4-layer PCB, ENIG | Main board | JLCPCB / PCBWay |
| 12 | PC/ABS clamshell + phone clip | Enclosure | Injection mold (TBD) |
| 13 | 15 cm USB-C extension cable | Cased-phone accessory | Off-the-shelf |

**Cost envelope:** RF + silicon ≈ $38–55 at 1k units; battery + power ≈
$6–9; enclosure + cable ≈ $9–14; assembly/test ≈ $12–18. Total COGS target
**$75–95**, leaving healthy margin at the $299 / $399 retail targets.

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
