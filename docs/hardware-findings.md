# Hardware & Network Findings

Everything confirmed so far about the existing third-party display unit,
from recon done against the live device.

## Device identification strings (from the unit's own UI)

```
OS:  F133-20220107
APP: F133-SSV-RS070WS098_A0-250604-1030
MCU: JY_WIFI_20241123-1651
BT MAC: 41:42:00:00:04:22
```

- **F133** — Allwinner F133 SoC family: single-core RISC-V (XuanTie C906),
  no dedicated GPU, used in cheap display/IoT boards. (Inference from the
  model string, not confirmed via datasheet lookup against the physical
  chip markings — board photos would confirm.)
- **RS070WS098** — likely a 7" display panel part number.
- **JY_WIFI** — a *separate* MCU handling Wi-Fi/Bluetooth, independent of the
  main F133 SoC. This is the chip that creates the `carplay_wifi_*` AP and
  speaks Bluetooth (`CarBT_52DD` observed as the BT pairing name).
- Manufacturer: **Plimpton** (per user). No public firmware/update portal
  found yet.

## On-unit connectivity UI

Observed modes, each independently connectable:

| Mode         | Status (observed)      |
|--------------|-------------------------|
| CarPlay      | Not connected (needs iPhone/MFi auth — can't be replicated from Android) |
| AndroidAuto  | **Connected** to Pixel 9 Pro |
| AirPlay      | Not connected |
| AndroidCast  | Not connected — showed `INVALID` after attempting to connect while Android Auto was already active |
| Call (BT)    | Connected |
| BT Music     | Connected |
| SD           | Not inserted |
| Rear Camera  | Off |

Bluetooth pairing name: `CarBT_52DD`.

## Network recon

The unit hosts its own Wi-Fi AP:

- **SSID:** `carplay_wifi_E674`
- **BSSID:** `88:49:2D:61:A4:43`
- **Subnet:** `192.168.43.0/24`, gateway/unit at `192.168.43.1`
- Phone joins as a client (e.g. `192.168.43.4`), link type Wi-Fi 4
  (802.11n), ~26 Mbps TX / ~65 Mbps RX observed — a likely contributor to
  video lag, since this is well below what 720p+ H.264 streaming wants.

### Port scan results (from phone, via Termux nmap, host on the unit's own AP)

```
nmap -Pn -sT -p 21,22,23,53,80,443,554,5000,5555,7000,7100,8000,8080,8554,8888 192.168.43.1
-> all 15 ports: filtered (no response at all)

nmap -Pn -sT -p 5288 192.168.43.1   (AOAP/Android Auto wireless control port)
-> filtered (tried twice, including with AA actively connected)

nmap -sn 192.168.43.0/24  (real host discovery, no -Pn)
-> only 192.168.43.4 (the phone itself) responds
```

Fing (proper ARP-based discovery on the phone) also found no devices beyond
the phone itself on `carplay_wifi_E674`.

**Conclusion:** the unit does not expose any conventional service (web admin,
SSH, telnet, open AA port) to normal TCP probing or ARP discovery. It likely
only responds within its own proprietary session-negotiation protocol, which
doesn't look like a plain TCP accept(). This is consistent with a
closed, locked-down embedded receiver — there is no evidence of a
network-based customization path.

## Open questions (need physical access to the board)

These need to be filled in from board photos/UART before any firmware-level
work is attempted:

- [ ] Confirm chip markings match Allwinner F133 (vs. a similar/rebadged part)
- [ ] Locate UART header (TX/RX/GND/VCC pads) near the main SoC
- [ ] Locate USB test points for Allwinner FEL recovery mode
- [ ] Identify the `JY_WIFI` MCU part number
- [ ] Check SD card boot priority behavior
