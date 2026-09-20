# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## What this repository is

A small KiCad 9 daughter board carrying the three FujiNet status LEDs (WiFi, Bluetooth,
SIO) for the [FN32ROV-XEBook-KiCad](../FN32ROV-XEBook-KiCad) main board. See `README.md`
for the full description.

Status: **shipped** — rev 1.0 fabricated at **JLCPCB** (not PCBWay; the main board went to
PCBWay under order `T-1D22W845207A`, this board and the SD daughter board were ordered
separately at JLCPCB) and already shipped as of 2026-08-01, from the `Fab/` package.

This board needed **no revision at all** — see "The floating copper pour here is
intentional" below — so its `Fab/` still matches source exactly and was left flat, with no
`as-built-1.0-*/` archive and no 1.1 package. Nothing to rework on arrival either.

Schematic captured, PCB routed (19 track segments,
2 vias), ERC clean, DRC clean, schematic/PCB parity clean, 0 unrouted items. The 40mm x
14mm board carries D1/D2/D3 on F.Cu with J1 (through-hole, `B4B-PH-K`) mounted on **B.Cu**
so the connector body sits on the opposite face from the LEDs.

### The floating copper pour here is intentional — leave it

The 2026-07-31 audit found all three boards in this family shared a single zone on
`(net 0)` — a floating pour rather than a ground plane. That was fixed in source on the
main and SD boards. **It cannot be fixed here and does not need to be: this board has no
GND net at all.** The circuit is common-anode 3V3 plus three cathode signals; there is
nothing to tie the pour to. With three DC-driven LEDs it makes no practical difference.
Don't "fix" it by tying the pour to 3V3. See the main board's `CLAUDE.md`, "Design
review", for the full context.

## Interface contract with the main board

This board's J1 mates via a 4-pin JST PH pigtail to J_LED1 on FN32ROV-XEBook-KiCad,
pin-for-pin (straight-through cable, not crossed):

| Pin | Net | Role on this board |
|---|---|---|
| 1 | 3V3 | Common anode supply for D1/D2/D3 |
| 2 | LED_WIFI | D1 cathode |
| 3 | LED_BT | D2 cathode |
| 4 | LED_SIO | D3 cathode |

Current-limiting resistors (R13/R14/R15) live on the **main** board, not here. If the LED
color/part choices on this board change, the main board's resistor values may need to
change too — see its `CLAUDE.md`, "External LED board interface (J_LED1)".

D1 (WiFi) is green (Rohm SML-P12PTT86R), not the original white — see README for why.

**Brightness matched (2026-09-18).** On the first build D2 (blue) and D3 (orange) were about
twice as bright as D1 (green). D1 is the reference; the fix is on the **main** board —
R14 1k → 30k and R15 1.2k → 10K (R13 stays 2.7k), found by a bench match with this board
powered at 3.3V. Nothing changes on this board. Full detail in the main board's
`CLAUDE.md`, "External LED board interface (J_LED1)".

## Mechanical constraints (hard requirements, don't change without confirming)

- Board outline: 40mm x 10mm.
- LEDs on 10mm pitch, centered on the board (x = 10, 20, 30mm), y = 5mm (mid-height).
  These positions must stay aligned to the XEBook case's light-pipe hole positions.
- M2 mounting holes 4mm in from each end (x = 4mm and x = 36mm), y = 5mm.
- J1 is **surface mount** (`JST_PH_B4B-PH-SM4-TB_1x04-1MP_P2.00mm_Vertical`), not
  through-hole — a THT part's pads plate through the entire board, so copper would land
  on the front (light-pipe) side no matter which face the body sits on. SMD pads exist
  only on the face the footprint is flipped to. J1 is mounted on the **back** (B.Cu),
  centered at x = 20mm, y = 5mm (board center) — same row as the LEDs is fine, since
  there's no shared-layer copper between front-side LED pads and back-side J1 pads to
  short against.
- Board thickness: 1.6mm.

## Working conventions

Same as the main board:

- The user does manual placement/routing/symbol cleanup themselves in the KiCad GUI;
  prioritize correctness over polish when editing files directly.
- KiCad ref/value labels for R/C (and other 2-pin parts) go beside the body, not
  above/below. Net-label stub length should scale with the label's name length.
- Verification is two-tier and both are required before calling the board correct:
  1. `kicad-cli sch erc` on the schematic.
  2. `kicad-cli pcb drc --schematic-parity` — PCB vs. schematic (references, footprints,
     net connectivity must match).
- This board has no "original design" netlist to diff against (unlike the main board,
  which is a rebuild of an existing FujiNet design) — its ground truth is the interface
  contract table above, which comes from the main board's J_LED1 documentation.
- The schematic (`.kicad_sch`) was hand-authored as an s-expression file. The PCB
  (`.kicad_pcb`) was generated via KiCad's bundled `pcbnew` Python scripting
  (`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/3.9/bin/python3.9`,
  which has a working `pcbnew` module, unlike the system `python3`) rather than
  hand-written, since it reliably handles the front/back mirroring math for J1. Prefer
  the same approach — scripted via `pcbnew`, or the KiCad GUI — for further PCB edits
  rather than hand-editing the `.kicad_pcb` s-expressions directly, since footprint
  mirroring and pad/net wiring are easy to get subtly wrong by hand.
- Gotcha found while building this: a `.kicad_sch`'s cached `lib_symbols` entries must use
  the fully-qualified name (e.g. `"Device:LED"`, `"Connector_Generic:Conn_01x04"`), not
  the bare name from the raw global library file (`"LED"`, `"Conn_01x04"`). A mismatch
  here doesn't error — it segfaults `kicad-cli sch erc`. Hit this bug twice: once
  originally, and once again after regenerating the schematic from a `/tmp` scratch
  script that still had the bare names baked into its cached symbol text — if ERC
  segfaults again, check this first before suspecting the new content.
- Symbol `Reference`/`Value` text `(at X Y)` positions in a `.kicad_sch` are absolute
  sheet coordinates, not relative to the symbol. If 2-pin parts are placed close together
  (e.g. LEDs on a tight row pitch) and you use the standard ~2.54mm above/below offset for
  a horizontal part, check the offset doesn't reach exactly onto a neighboring part's
  position — ERC won't catch this, only a visual check (or DRC's silkscreen-overlap
  checks) will.
