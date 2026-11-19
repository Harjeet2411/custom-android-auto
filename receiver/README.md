# Receiver (Raspberry Pi target)

Scaffold for Phase 1 of `../docs/project-plan.md`. Empty until we have Pi
hardware to build and test against from a session attached to it.

Planned layout once build-out starts:

```
receiver/
  aasdk/          vendored or submoduled aasdk library
  ui/             Qt-based custom shell (home screen, theming, status bar)
  CMakeLists.txt
  scripts/
    setup-pi.sh    provisions Raspberry Pi OS Lite with build deps
```

## Dependencies (Raspberry Pi OS Lite, 64-bit)

To be installed via `scripts/setup-pi.sh` once written:

- `build-essential`, `cmake`
- `libusb-1.0-0-dev` (aasdk talks to the phone over USB/AOAP)
- `libssl-dev` (AA protocol uses TLS for the control channel)
- `libprotobuf-dev`, `protobuf-compiler` (AA messages are protobuf)
- Qt6 dev packages (`qt6-base-dev`, `qt6-multimedia-dev` or Qt5 equivalents)
- `libboost-dev` (aasdk dependency)

## Next step

Needs a Raspberry Pi reachable from a session (local machine attach, or SSH)
to actually build and test. See `docs/project-plan.md` → "What's needed from
you next".
