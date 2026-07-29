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
- J1 — 4-pin JST PH vertical header, **surface mount**
  (`JST_PH_B4B-PH-SM4-TB_1x04-1MP_P2.00mm_Vertical`), mounted on the **back** of the
  board. SMD rather than through-hole on purpose: a THT part's pads plate through the
  whole board, landing copper on the front (light-pipe) side even when the body sits on
  the back — SMD pads only exist on whichever face the footprint is flipped to. Mates via
  a JST PH pigtail to J_LED1 on the main board, pin-for-pin (1=3V3, 2=LED_WIFI, 3=LED_BT,
  4=LED_SIO).

Current-limiting resistors are **not** on this board — they live on the main board
(R13/R14/R15) since this board only ever sees the LED forward voltage drop.

## Mechanical

- Board outline: 40mm x 10mm.
- LEDs sit on 10mm pitch, centered on the board (x = 10/20/30mm from the left edge),
  aligned to the XEBook case's light-pipe positions. LEDs use SMD packages with panel
  light pipes carrying light out to the case surface — no hole is drilled in the PCB
  itself under each LED.
- Two M2 clearance holes (`MountingHole_2.2mm_M2`), 4mm in from each end, for mounting
  into the case.
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
