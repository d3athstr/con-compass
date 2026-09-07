# Con Compass

A lanyard badge that finds your friends in a crowd of 85,000 people.

Built for DragonCon, where the three things you would normally reach for all
fail at once: cell data is saturated to uselessness inside the host hotels, GPS
does not work inside the Marriott Marquis or the Peachtree Center tunnels, and
2.4GHz is drowned by tens of thousands of phones. So the badge carries its own
radio.

## How it works

Three radio layers, each doing the job it is actually good at.

| Layer | Radio | Job |
|-------|-------|-----|
| Tether | BLE | The phone feeds the badge its fix; the badge feeds the phone the peer list |
| Mesh | LoRa 915MHz | Position + status between badges, through concrete |
| Homing | LoRa RSSI, then BLE RSSI | The last 50 metres, where no fix exists |

**The phone supplies position, not the badge.** A GNSS module would cost ~$20
per unit, eat the power budget, and still not work indoors — which is exactly
where it is needed. The phone already has a far better fix, because it fuses
GPS, WiFi positioning, cell and its own IMU. The badge takes that over BLE for
free and degrades gracefully: no phone fix, fall back to RSSI homing.

**LoRa is the trunk** because at 915MHz packets get through hotel concrete and
elevator lobbies where 2.4GHz and cellular do not, and a closed group of ten
generates almost no traffic.

**RSSI homing is the point.** A map pin stops being useful once you are within a
room's distance in a packed ballroom, because the position error exceeds the
distance to the target. Homing mode drops the map and shows a hot/cold bar
driven by smoothed RSSI — LoRa first, handing off to BLE inside ~30m where the
update rate is higher and the gradient much sharper.

## Firmware

Custom firmware speaking a **Meshtastic-compatible channel**, so the badges also
appear to the existing LoRa fleet and to anyone in the group running only the
stock Meshtastic app.

Custom rather than stock because stock Meshtastic does not do the part that
matters: position broadcasts are throttled by airtime rules to intervals far too
slow for "where did they go", and there is no RSSI homing UI at all. Stock
already covers tether + phone GPS + peer map, so that half is kept
interoperable rather than reinvented.

The XIAO ESP32-S3 + Wio-SX1262 pairing is an official Meshtastic target
(`variants/esp32s3/seeed_xiao_s3`), so a badge can be flashed with stock
firmware for debugging without rewiring. The pin assignment in the schematic
deliberately matches that variant's I2C pins to keep that true.

## Hardware

`hardware/badge/schematic.txt` is the authoritative wiring document — net list,
power path, polarity, and the PartsBin component behind every refdes. Read it
before touching anything.

Two things in there that are easy to get wrong and expensive to get wrong:

- **The XIAO land is 15.24mm, not 17.0mm.** 17.0mm is the SMD castellation
  span. This mistake scrapped 25 fabbed boards across six carriers in August.
- **The Wio-SX1262 is not on the carrier.** It mates to the XIAO's underside
  B2B connector and hangs beneath it, so the board needs a keepout and
  clearance cutout under the XIAO land, and there is no SPI on the carrier at
  all.

## Status

Planning. Firmware unstarted. The board is specified but **not drafted** — no
KiCad tree, no gerbers. One item still blocks layout: antenna placement on a
body-worn badge needs measuring on a prototype, because a wearer absorbs a great
deal of 915MHz and that decides the outline.

Parts are tracked in PartsBin project 22, which is the source of truth for the
BOM and what is short.

## Regulatory

915MHz ISM under FCC Part 15.247 with a pre-certified module — unlicensed, fine
for personal use, no amateur licence required. Nothing here transmits outside
ISM.

## Privacy

Everyone carrying a badge is broadcasting their position to everyone else
carrying one. That is the entire point, and it is consensual by construction —
but it means the group PSK is the only thing standing between the group and a
stranger with an SDR. Rotate it between events, and do not put a badge in
someone's bag as a surprise.
