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

## Active path: jailbreaking the existing unit

Since the unit is out of warranty and not working well, the current focus is
getting low-level control of it (UART/FEL access, firmware dump) rather than
replacing it with new hardware. See
[`docs/jailbreak-guide.md`](docs/jailbreak-guide.md) for the step-by-step
process — it's blocked on physical board photos/UART probing, which only
you can do.

**Important:** this will *not* fix the Android Auto video lag. That's a
fixed hardware ceiling (no GPU, software video decode on a single-core
RISC-V chip) — see
[`docs/architecture-decision.md`](docs/architecture-decision.md). The
jailbreak's value is open access/control of the device itself.

## Longer-term option: a from-scratch receiver on better hardware

If fixing the lag itself ever becomes the priority, [`docs/project-plan.md`](docs/project-plan.md)
has a plan for building an open-source Android Auto **head-unit receiver**
on a Raspberry Pi 4/5 instead, with real hardware video decode and full UI
control. Not the active path right now.

## Quick wins on the existing unit (while the real build is in progress)

See [`docs/quick-fixes.md`](docs/quick-fixes.md) for things to try right now
that may reduce lag without any hardware changes.

## Repo layout

```
docs/           research notes, findings, architecture decisions
receiver/       the custom Android Auto head-unit receiver project (Pi target)
tools/          helper scripts (adb resolution tweaks, network recon notes)
```
