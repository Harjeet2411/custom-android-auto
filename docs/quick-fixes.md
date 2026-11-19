# Quick Fixes To Try On The Existing Unit

None of these require opening the device or flashing anything. Try them in
order — cheapest/least-effort first.

## 1. Switch Android Auto to wired (USB) if at all possible

Wireless AA is almost always laggier than wired, because it depends on
Wi-Fi link quality instead of a stable USB connection. The unit has no
exposed USB port per earlier findings, but double check — some of these
"CarPlay wifi" dongles are themselves a USB dongle that plugs into a
*different*, USB-equipped head unit. If this unit truly has zero USB in or
out, wired isn't an option and skip to #2.

## 2. Force the phone's hotspot/AA link to 5GHz if the unit supports it

Settings → Network & internet → Hotspot & tethering → Wi-Fi hotspot →
Advanced → AP Band. Try 5 GHz if the unit's AP (`carplay_wifi_E674`) also
offers it — check in the unit's own Wi-Fi settings/menu. 5 GHz has far less
interference and higher throughput than 2.4 GHz, though range is shorter
(not an issue in a car).

If the unit only does 2.4 GHz (likely, given the JY_WIFI MCU is a cheap
part), this won't help — but it's a 30-second test.

## 3. Lower Android Auto's video resolution/bitrate

On the phone, with Android Auto's developer settings unlocked (tap the AA
version number ~10 times in its settings to reveal Developer settings):

- **Video resolution**: try forcing 480p instead of 720p/1080p if offered.
  Lower resolution = less data to decode on a weak chip = less lag.
- **Video Framerate**: lower to 20fps/16fps if offered.
- Some builds also expose a bitrate/DPI override — lower DPI slightly
  reduces encode/decode cost too.

This is the single highest-leverage fix given the F133 has no real GPU —
less video data directly reduces decode time.

## 4. Reduce Bluetooth/Wi-Fi contention

- Turn off Bluetooth features you're not using (e.g. if you're not using BT
  Music/Call while AA is active over Wi-Fi, disabling them frees radio time).
- Close other apps using the phone's Wi-Fi/mobile data heavily in the
  background while driving (uploads, syncs) — they compete for the same
  radio and CPU.

## 5. Keep the phone plugged in / cool

Thermal throttling on the phone during sustained H.264 encode + hotspot
operation can itself introduce stutter, independent of the unit. If the
phone gets warm during use, that's a contributing factor — vent mount, not
direct sun, and keep it charging.

## 6. Try an alternative mirroring app's bitrate settings

If using AAAD/Fermata Auto for mirroring (not plain Android Auto), check
their settings for resolution/bitrate caps — same logic as #3 applies, the
F133 side has no horsepower to spare.

## Expected outcome

These won't make the unit fast — its hardware ceiling is real — but
lowering resolution/bitrate (step 3) is usually the single biggest lever on
a weak-decode device like this, and costs nothing to try.
