# Custom Android Auto Head Unit

This project exists because a cheap third-party "CarPlay/Android Auto" wireless
display unit (SSID `carplay_wifi_E674`) is slow and closed-source, and we want
a custom UI experience instead.

## Why this isn't a "debloat the existing unit" project

Android Auto's UI is rendered entirely on the **phone** by Google's closed,
signed Android Auto app. It's streamed to the head unit as H.264 video over
Wi-Fi/Bluetooth. The head unit's only job is decode + display + send touches
back. There's no bloat on the unit to strip for Android Auto itself — the
unit's firmware is a thin, closed receiver.

See [`docs/hardware-findings.md`](docs/hardware-findings.md) for everything
we've confirmed about the existing unit, and
[`docs/architecture-decision.md`](docs/architecture-decision.md) for why we're
not trying to flash it.

## What we're building instead

An open-source Android Auto **head-unit receiver** (the car side of the
protocol), based on the same approach as `aasdk`/OpenAuto/Crankshaft-style
projects, running on hardware that can actually decode video — a Raspberry Pi
4/5, not the existing F133-based unit. This gives us:

- Full control over the on-screen UI (custom theme, layout, shortcuts)
- Proper hardware video decode, fixing the lag
- A platform we can actually extend (our own launcher, widgets, etc.)

See [`docs/project-plan.md`](docs/project-plan.md) for the build plan and
current status.

## Quick wins on the existing unit (while the real build is in progress)

See [`docs/quick-fixes.md`](docs/quick-fixes.md) for things to try right now
that may reduce lag without any hardware changes.

## Repo layout

```
docs/           research notes, findings, architecture decisions
receiver/       the custom Android Auto head-unit receiver project (Pi target)
tools/          helper scripts (adb resolution tweaks, network recon notes)
```
