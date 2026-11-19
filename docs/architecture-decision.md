# Architecture Decision: Why Not Flash the Existing Unit

## The ask

"Debloat this device and build a custom UI that works as Android Auto,
because the current Android Auto experience is slow."

## Why the existing F133 unit is not a viable target

1. **Android Auto's UI isn't on the unit at all.** It's rendered by Google's
   closed, cryptographically-signed Android Auto app on the *phone*, then
   streamed as H.264 video. There is no "Android Auto bloat" on the unit to
   strip — the unit only decodes and displays video and reports touch input.
   A custom UI *for Android Auto's own screens* isn't something we can build
   by modifying the unit; it would require modifying Google's app, which is
   closed-source, signed, and against its terms of service.

2. **The slowness is a hardware/link ceiling, not a software one.**
   - The unit's own Wi-Fi link tops out around 26/65 Mbps (802.11n/Wi-Fi 4),
     thin for sustained H.264 video.
   - The F133 is a single-core RISC-V SoC with no dedicated GPU. Likely
     software video decode. This is a hard ceiling — no firmware change
     fixes a lack of hardware decode.
   - Reflashing the *same chip* with different software cannot make its
     video decode faster.

3. **The unit is locked down at the network level** — see
   `hardware-findings.md`. No open admin interface, no open relevant ports.
   Any firmware-level jailbreak would require physical UART/FEL access, with
   no guarantee of a usable outcome even if we get in (the chip's
   constraints don't change).

## What "a custom UI that works as Android Auto" actually means, and how to get it

What you want is functionally: **be the car side of the Android Auto
protocol ourselves**, so we control what's drawn on screen instead of
Google's app. This is a known, open-source-trodden path — it's how projects
like OpenAuto, Crankshaft-ng, and commercial "OpenAuto Pro" units work. They
implement the Android Auto *protocol* (via libraries like `aasdk`) as a
receiver/host, and render their own UI (commonly via Qt) instead of just
proxying Google's video stream passively... actually in standard Android
Auto, the receiver still gets a rendered video stream from the phone for the
*built-in* Android Auto apps (nav, media, etc.) — full "replace Google's UI
pixel-for-pixel" is not something any of these projects do either, since
Android Auto fundamentally works by remote-rendering on the phone.

What IS achievable and is the actual value proposition of a custom receiver:
- A **custom home/launcher screen** that is not the Android Auto home —
  shown when AA isn't active, or as a wrapper around it.
- **Lower latency decode** than the F133 by using real hardware (Pi 4/5 has
  a proper H.264/H.265 decode path and a GPU).
- **Our own additional features** layered around the AA video surface —
  shortcuts, media controls, vehicle data, theming of chrome/borders.
- Still subject to the same protocol: Google's app still renders Nav/Media
  templates from the phone side. We can't re-skin those screens; we *can*
  control everything around them and how smoothly they're displayed.

## Decision

- **Do not attempt to flash/replace firmware on the existing F133 unit.**
  Hardware ceiling makes it pointless even if a jailbreak succeeds.
- **Build a custom Android Auto receiver on a Raspberry Pi 4/5**, based on
  the `aasdk` protocol library, with our own UI chrome around it. This
  becomes the new "head unit" — the existing screen can stay as a
  cheap spare/backup, or be repurposed as a secondary display.
- **In parallel**, apply the tuning in `quick-fixes.md` to reduce lag on the
  current unit short-term while the Pi build is in progress.

This is a multi-week hardware + software project, not a firmware patch. See
`project-plan.md` for phased scope.
