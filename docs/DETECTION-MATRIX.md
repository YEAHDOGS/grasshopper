# Grasshopper — Detection matrix

Honest accounting of what each mode can and cannot find. Rev A targets.

## Sweep mode finds

- Wi-Fi cameras and nanny cams that are powered and transmitting
- Bluetooth / BLE trackers (AirTag, Tile, knockoffs) via advertisements
- VHF/UHF analog listening bugs (SDR demod)
- Sub-GHz transmitters (433 / 915 MHz remotes, some telemetry bugs)
- Anything radiating in a covered band, by proximity — hotter/colder

## Map mode finds

- Every device on your Wi-Fi network (ARP/DHCP/mDNS inventory)
- 5 GHz Wi-Fi devices
- BLE advertisers in range, plotted by signal strength
- Rogue APs / evil twins (Wi-Fi frame analysis)
- New or unknown devices the moment they appear — the red pulse

## Conditional (later signature updates)

- Cellular GPS trackers — only while transmitting
- Zigbee / Z-Wave — already in-band; signatures ship free
- DECT / baby monitors — 1.9 GHz SDR demod
- Drones — 2.4/5.8 GHz video-link fingerprints

## Cannot find (no RF sweeper can)

- Wired devices (no radio, nothing to hear)
- Shielded / Faraday-bagged devices
- Powered-off devices
- Local-recording devices that never transmit (SD-card voice recorders)
- Laser microphones (optical — out of scope, documented in-app)
