# Project Plan: Custom Android Auto Receiver

> **Status update:** active path is now the jailbreak of the existing F133
> unit — see `jailbreak-guide.md`. This document's Raspberry Pi plan is kept
> as the longer-term option if/when fixing the video lag itself becomes the
> priority, since the jailbreak alone won't resolve that (fixed decode
> hardware ceiling on the F133 — see `architecture-decision.md`).

## Goal

A custom-UI Android Auto head unit, built on hardware capable of smooth
video decode, replacing the laggy F133-based unit as the thing mounted in
the car.

## Target hardware

**Raspberry Pi 4 (2GB+) or Pi 5**, plus:
- An official 7"+ touchscreen (DSI) or any HDMI touchscreen matching the
  dash cutout size of the existing unit, if we want a drop-in physical swap
- A USB-C/USB-A cable to the phone (wired AA — far more reliable than
  wireless, and avoids re-implementing the wireless AOAP handshake)
- Car power (5V/3A+ supply from the vehicle's 12V, e.g. a buck converter)

Wired-first is the right call for v1: it sidesteps Wi-Fi link-quality
problems entirely (the #1 suspected cause of lag) and the AA wired protocol
(AOAP) is the most mature path in `aasdk`-based projects.

## Software stack

- **`aasdk`** (open-source C++ library implementing Google's Android Auto
  protocol as a receiver) as the protocol layer — this is the same
  foundation used by OpenAuto/Crankshaft-ng projects referenced in
  `architecture-decision.md`.
- **Qt (5 or 6)** for the custom UI chrome/launcher around the AA video
  surface — theming, a custom home screen, shortcuts, vehicle info panel.
- **Raspberry Pi OS Lite (64-bit)** as the base OS — minimal, no desktop
  bloat, boots fast.

## Phases

### Phase 0 — Validate the hardware swap is viable (you)
- [ ] Confirm the Pi + touchscreen can physically fit the existing dash
      opening, or decide to mount separately
- [ ] Source a Pi 4/5, touchscreen, and 12V→5V power supply

### Phase 1 — Bring up `aasdk` + stock reference UI
- [ ] Build `aasdk` and its reference `h-rae`/autoapp-style demo on the Pi
- [ ] Confirm wired Android Auto connects from the Pixel and video renders
- [ ] Measure latency/framerate as a baseline vs. the F133 unit

### Phase 2 — Custom UI shell
- [ ] Qt app wrapping the AA video surface: custom home screen shown when AA
      isn't connected, our own status bar/theme around the AA surface,
      quick-launch tiles for non-AA functions (e.g. OBD data, camera feed)

### Phase 3 — Polish
- [ ] Boot-to-UI time optimization (splash screen, fast boot config)
- [ ] Optional: reverse camera input via a cheap capture device
- [ ] Optional: wireless AA support once wired is solid (adds back the
      Wi-Fi-link complexity we're avoiding in v1)

## Current status

**Phase 0, not started.** This repo currently contains research/findings
only (`docs/hardware-findings.md`), the architecture decision to build fresh
rather than flash the F133 unit, and this plan. `receiver/` has a scaffold
for Phase 1 (empty until hardware is in hand to build/test against).

## What's needed from you next

1. Confirm the Pi 4/5 + wired-first approach is the direction you want
   (vs. continuing to chase a wireless/F133-based fix).
2. Source the hardware listed above.
3. Once you have a machine with the Pi attached (or the Pi itself reachable
   over SSH), we can move from scaffolding to actually building/testing —
   this cloud session can't compile/flash against real hardware itself.
