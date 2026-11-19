# Jailbreak Guide: F133-based Plimpton Display Unit

**Goal:** gain control of the unit's software (dump firmware, get a shell,
potentially run our own minimal OS on it) via its hardware debug interfaces,
since the network side is fully locked down (see `hardware-findings.md`).

**Scope note:** this is a hardware recovery/jailbreak project on your own,
out-of-warranty device. It will not fix the Android Auto video lag (that's
a fixed decode-hardware ceiling on the F133 — see
`architecture-decision.md`) — the value here is open access to run our own
software on it, for whatever we then decide to do with that (custom boot
screen, a lightweight native UI for non-AA functions, learning/tinkering).

Status: **blocked on physical board access** — everything below needs you
to open the case and report back. I have no way to interact with the
hardware from this session.

---

## Tools you'll need

| Tool | Purpose | Notes |
|---|---|---|
| Small Phillips/Torx driver set + plastic spudger | Open the case | Standard cheap electronics kit |
| Multimeter | Identify GND, VCC, and continuity-check pads | Any basic one works |
| USB-to-TTL serial adapter (CP2102 or CH340 based, 3.3V!) | UART console | **Must be 3.3V logic**, not 5V — 5V can kill the SoC's UART pins |
| A PC (Linux/Mac/WSL preferred) | Run `sunxi-fel`, serial terminal, flashing tools | This cloud session can relay commands/interpret output, but can't drive your USB hardware directly |
| Jumper wires / fine-tip soldering iron (optional) | Connect to unpopulated pads if needed | Many of these boards have pads but no header soldered on |
| microSD card (8GB+) + reader | Test SD-boot path | |

---

## Step 1 — Open the case, photograph both sides of the main board

Send me clear photos (good lighting, in focus) of:
- The full board, both sides
- Close-up of the main SoC's markings
- Close-up of any row of 3-6 unpopulated pads/holes (headers) anywhere on
  the board — especially near the SoC or near an edge
- The `JY_WIFI` module/chip if it's a visually separate daughterboard or
  chip

I'll use these to identify the exact chip revision, locate candidate UART
pads, and check for an FEL/USB-OTG test point.

## Step 2 — Find UART (if present)

Allwinner dev-style boards almost always expose UART0 for boot logging, even
on consumer products, because it's cheap (just 2-4 pads) and used in
factory test. Look for a row of pads labeled (or silkscreened, or just
bare) near the SoC, often 4 in a line: **GND, TX, RX, VCC** (order varies —
never assume, always verify with the multimeter before connecting).

**Identifying pads with a multimeter (power OFF):**
- **GND**: continuity-beep to the board's shield/USB connector shell or a
  large copper pour.
- **VCC**: skip connecting this one regardless — power the board from its
  own supply, and only tap TX/RX/GND from the adapter. Connecting your
  adapter's 3.3V to the board's VCC pad risks back-powering/conflicting
  supplies.
- **TX (from the board's perspective)**: with the board powered on, this
  pin will show toggling voltage (looks "noisy"/non-zero) right at boot —
  that's the SoC transmitting boot logs.
- **RX**: typically idles high and is quiet.

**Wiring**: adapter **RX → board TX**, adapter **TX → board RX**, adapter
**GND → board GND**. Cross TX/RX, don't run them straight.

**On the PC**, open a serial terminal at common Allwinner baud rates (try in
this order): `115200 8N1`, then `921600`, then `1500000`. Power-cycle the
board while the terminal is open and watch for boot text.

```
# Linux example
screen /dev/ttyUSB0 115200
# or
minicom -D /dev/ttyUSB0 -b 115200
```

If you see boot log text, paste it to me — it'll tell us the bootloader
(likely U-Boot), whether it drops to an interactive shell (often by
pressing a key within a short window at boot), and the exact partition
layout.

## Step 3 — Check for Allwinner FEL mode

FEL is Allwinner's mask-ROM USB recovery mode, present on essentially every
Allwinner chip including F133, and cannot be disabled by the vendor's
firmware — it's burned into the SoC's boot ROM. This is the most reliable
jailbreak/recovery primitive for this class of chip.

Entering FEL usually requires one of:
- A dedicated FEL/boot-select button on the board (uncommon on consumer
  units, but check for any unlabeled tactile button)
- Shorting a specific "boot select" pad to GND or VCC at power-on (chip- and
  board-specific — board photos needed to find candidates)
- On some designs, holding a particular key combo on an attached
  button/touch area during power-on

There's likely no populated USB port for FEL to enumerate over, per earlier
recon. If UART (Step 2) reveals a U-Boot prompt, that's actually easier to
work with than FEL, so try Step 2 first.

**If we do get FEL access**, from the PC:
```
# Install the sunxi tools (Debian/Ubuntu/WSL)
sudo apt install sunxi-tools

# Confirm FEL mode is detected
sudo sunxi-fel ver

# Full firmware backup BEFORE anything else — this is our undo button
sudo sunxi-fel -p spiflash-read 0 0x1000000 backup_full_flash.bin
# (size depends on the actual flash chip — sunxi-fel ver / board research
# will tell us the real size once we're at this step)
```

## Step 4 — SD card boot test (no soldering needed, try this anytime)

Allwinner's boot ROM checks SD card before internal flash by default on
most designs. Worth testing in parallel with Steps 2-3:

1. Format a spare microSD card FAT32.
2. Power off the unit, insert the blank SD card, power on.
3. Note any behavior change (different boot time, different screen, boot
   loop, error message) vs. booting with no SD card. Even "nothing changes"
   is useful information — it may mean SD-boot is disabled in this
   design's boot order.

Don't put anything executable on the SD card yet — this first test is just
to observe default behavior safely.

## What to send me after each step

- Step 1: the board photos
- Step 2: UART boot log text (if any), or "no response on pads X/Y/Z at
  baud rates tried" if nothing shows
- Step 3: whether a boot-select button/pad was found, and `sunxi-fel ver`
  output if you get that far
- Step 4: observed behavior with blank SD inserted vs. not

I'll interpret each result and give you the next concrete action — this is
a "go slow, verify each step" process since the wrong pad/voltage can
damage the board, and we have no second unit to fall back on.
