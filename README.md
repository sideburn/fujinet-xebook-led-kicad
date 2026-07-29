# FN32ROV XEBook LED Board

A small daughter board carrying the three FujiNet status LEDs (WiFi, Bluetooth, SIO) for
the [FN32ROV-XEBook-KiCad](../FN32ROV-XEBook-KiCad) main board, which mounts the FujiNet
FN32ROV WiFi/SIO adapter permanently inside an XEBook laptop build.

On the main board, the LEDs and buttons were moved off-PCB onto JST pigtails so they can
be panel-mounted separately (see that project's `CLAUDE.md`, "External LED board
interface (J_LED1)"). This board is that panel-mounted LED daughter board.

## What's on it

- D1 (WiFi), D2 (Bluetooth), D3 (SIO) — 0402 SMD LEDs, common-anode to 3V3, each cathode
  wired back to its own signal pin. The ESP32 on the main board sinks current by driving
  the corresponding GPIO low.
  - D1 is **green** (Rohm SML-P12PTT86R) — the original FN32ROV-1.7.1 WiFi LED was white
    (SunLED XZBWR68F5MAV-3, now obsolete); this board uses green instead. The main board's
    R13 was resized from 1k to 2.7k to keep the current sane for the green LED's lower
    forward voltage (~2.2V vs ~2.9V for white).
  - D2 (blue, OSRAM LB QH9G-N1OO-35-1) and D3 (orange, Rohm SML-P12DTT86R) are unchanged
    from the original FN32ROV-1.7.1 BOM.
- J1 — 4-pin JST PH vertical header, **through-hole**
  (`JST_PH_B4B-PH-K_1x04_P2.00mm_Vertical`), mounted on the **back** of the board and
  hand-soldered. An earlier revision used an SMD connector specifically to avoid THT
  pads plating through to the light-pipe side, but assembling both sides of the board
  turned out to be too costly for JLCPCB assembly — hand-soldering a THT part is cheaper.
  Since a THT part's copper unavoidably lands on both faces, J1 is instead offset well
  clear of the LED row in Y (the board's height was grown from 10mm to 14mm specifically
  to make room for this), so its pads don't land under a light pipe. Mates via a JST PH
  pigtail to J_LED1 on the main board, pin-for-pin (1=3V3, 2=LED_WIFI, 3=LED_BT,
  4=LED_SIO).

Current-limiting resistors are **not** on this board — they live on the main board
(R13/R14/R15) since this board only ever sees the LED forward voltage drop.

## Mechanical

- Board outline: 40mm x 14mm.
- LEDs sit on 10mm pitch, centered on the board (x = 10/20/30mm from the left edge,
  y = 3mm), aligned to the XEBook case's light-pipe positions. LEDs use SMD packages
  with panel light pipes carrying light out to the case surface — no hole is drilled in
  the PCB itself under each LED.
- Two M2 clearance holes (`MountingHole_2.2mm_M2`), 4mm in from each end, at y = 7mm
  (the board's vertical center) — 4mm below the LED row.
- J1's housing sits with its near edge at that same y = 7mm row (pad row at y = 9.2mm,
  2.2mm further down — that offset is baked into the connector's footprint). This gives
  the light-pipe housing (~2.1mm radius around each LED) about 1.9mm of clearance to
  J1's physical housing edge — the dimension that actually matters here, not the copper
  pad edges, since it's the plastic housings that would physically collide if this were
  too tight.
- Board thickness: 1.6mm (JLCPCB/industry standard, matches the main board).

## Status

Work in progress — schematic captured and footprints/nets placed on the PCB (ERC clean,
DRC clean, schematic/PCB parity clean). **Not yet routed** — the user routes copper by
hand in the KiCad GUI. Not yet fabricated.

## Tooling

Built with KiCad 9. `kicad-cli sch erc` / `kicad-cli pcb drc --schematic-parity` are used
to verify the design as it's built.

## License

[CERN Open Hardware Licence Version 2 - Strongly Reciprocal](LICENSE)
(CERN-OHL-S-2.0), matching the main board.
