# Grasshopper — Hardware (Rev A proposal)

> Concept documentation. Nothing here is final — this is the engineering
> target the industrial design and DFM pass will firm up. All parts below are
> real, in-production components chosen for cost and availability.

## Concept

Grasshopper is a **standalone handheld sweeper**: its own screen, its own
battery, one button. Take it anywhere in the world, push the button, scan
for spyware. There is no app and no phone pairing — the phone is only ever
a charger. One model, everything in the box.

## Two modes

### Mode 1 — Sweep (metal-detector style)

Point-and-sweep RF proximity. The front-end reports per-band received power;
the ESP32-S3 drives a **piezo beeper + ERM vibration motor** with
Geiger-counter feedback — faster beeps and a stronger buzz as you close in.
The directional whip gives bearing. No screen, no graphs, no waterfall.

### Mode 2 — Map (your house as a diagram)

Grasshopper joins the home Wi-Fi as a client and inventories the network:
ARP/DHCP sweep, mDNS/Bonjour + UPnP names (so "Living Room TV" labels
itself), BLE advertisements, and per-device RSSI. Everything renders on the
on-device screen: an editable floor plan — swap floors, drag rooms, rename
them — with each device plotted as a dot positioned by RSSI multilateration
(room-level accuracy), pulsing red when a device is unknown or new.
Walk-the-house calibration: carry Grasshopper room to room once and the
dots sharpen. No app, no phone — the map lives on the device.

**Honest limits:** Wi-Fi alone cannot auto-draw an architect-accurate floor
plan — the plan starts as your quick sketch (or a room list) and Grasshopper
fills it with dots. Person-tracking through walls via Wi-Fi channel-state
(CSI) sensing is real published research and sits on the roadmap, not in
Rev A. Wired, shielded, powered-off, or purely local-recording devices emit
nothing detectable — no RF sweeper can find those. See
docs/DETECTION-MATRIX.md.

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
                         │ beeper + haptic driver   │
                         └────────────┬─────────────┘
                                      │
                    ┌─────────────────┴──────────────────┐
                    │                                    │
         ┌──────────▼──────────┐              ┌──────────▼──────────┐
         │  1,500 mAh Li-Po    │              │  2.0" TFT DISPLAY   │
         │  USB-C charging     │              │  240×320 · on-device│
         │  ~5 hr sweep        │              │  UI · no app needed │
         └─────────────────────┘              └─────────────────────┘
```

## Power

- **Primary:** internal 1,500 mAh Li-Po — ~5 hours of continuous sweeping
  with the screen on.
- **Charging:** USB-C, 5 V in, from any source — wall, laptop, or your
  phone's reverse-charge. The phone is only ever a charger: no app, no
  pairing, no data leaves the device. Full charge in ~90 minutes.
- **Power budget (typical):**

  | Block            | Typical | Peak   |
  |------------------|---------|--------|
  | ESP32-S3 + DSP   | 160 mA  | 240 mA |
  | SDR front-end    | 280 mA  | 350 mA |
  | Sub-GHz (RX)     | 20 mA   | 120 mA |
  | 2.0" TFT display | 90 mA   | 150 mA |
  | Regulators/misc  | 40 mA   | 60 mA  |
  | **Total @ 5 V**  | **~1.3 W** | ~3 W |

  A 10-minute full sweep ≈ **under 4% of the internal cell.**

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
| 12 | PC/ABS clamshell | Enclosure | Injection mold (TBD) |
| 13 | 15 cm USB-C extension cable | Cased-phone accessory | Off-the-shelf |
| 14 | Piezo buzzer, 3 V | Sweep-mode audio feedback | LCSC / CUI via Digi-Key |
| 15 | ERM coin vibration motor, 3 V | Sweep-mode haptic feedback | LCSC / Jinlong |
| 16 | 2.0" TFT LCD, 240×320 SPI (ILI9341-class) | On-device display — no app | LCSC / BuyDisplay |
| 17 | 5-way nav switch + 2 tactile buttons | UI input: sweep, map, select | LCSC / CUI via Digi-Key |
| 18 | 5 GHz Wi-Fi scan radio module | Dedicated 5 GHz sweep | Espressif / LCSC |
| 19 | Extended-band SDR module | 1.8–2.7 GHz cellular coverage | Module vendor (TBD) |

**Cost envelope:** RF + silicon ≈ $55–80 at 1k units; battery + power +
haptics ≈ $7–10; display + buttons ≈ $7–10; enclosure + cable ≈ $9–14;
assembly/test ≈ $12–18. Total COGS target **$100–130**, leaving healthy
margin at the single $299 retail price. No tiers, no options — one model,
everything in the box.

## Band coverage (single model)

| Band | Covered | Notes |
|------|---------|-------|
| 500 kHz – 1.75 GHz (SDR) | ✓ | VHF/UHF bugs, DECT, 433/868/915 ISM |
| 2.4 GHz Wi-Fi / BLE | ✓ | ESP32-S3 monitor mode |
| 5 GHz Wi-Fi | ✓ | Dedicated scan radio |
| 1.8 – 2.7 GHz cellular | ✓ | Extended SDR module |
| 3.5 GHz 5G | roadmap | Antenna + tuner rev |

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
