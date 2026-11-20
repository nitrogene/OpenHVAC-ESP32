# Project Roadmap & Active Checklist (TODO)

This document constitutes the operational action plan for **OpenHVAC-ESP32** (custom ESP32 Zigbee PCB + enclosure replacing the Tuya Wi-Fi module on an Airton air conditioner).

---

## Phase 1: Reverse Engineering & Electrical Specifications

- [ ] **Stock Tuya Module Inspection:** Document the original Airton Tuya Wi-Fi module (module model, MCU/chipset, physical dimensions, connector type/pitch, mounting constraints).
- [ ] **Pinout & Electrical Characterization:** Measure and verify connector pinout (`VCC` voltage & current capability, `GND`, `UART_TX`, `UART_RX`, optional `EN`/`RST` or status pins) and logic voltage levels (`3.3V` vs `5V`).
- [ ] **Enclosure & Mechanical Envelope:** Measure maximum PCB dimensions, connector orientation, cable exit, and available clearance inside/around the Airton indoor unit.
- [ ] **Architecture Selection (`documents/hardware.md`):**
  - Select target Zigbee ESP32 module/SoC (e.g., `ESP32-C6-WROOM-1` / `ESP32-C6-MINI-1` / `ESP32-H2`).
  - Define power supply stage (LDO or Buck converter from Airton rail to `3.3V`, input protection, decoupling).
  - Define UART level shifting / protection (if Airton bus is `5V` or requires series/ESD protection).
  - Define programming & debug interface (USB-C or UART header / test pads + Boot/Reset buttons).

---

## Phase 2: Schematic Design & BOM (`documents/schema/`)

- [ ] **Datasheets Archive:** Collect and archive all component datasheets under `documents/datasheets/references/` and index them in `documents/datasheets/datasheets.md`.
- [ ] **Schematic Capture (EasyEDA Pro):** Build schematic with explicit net names, physical wires on all NetPorts, and `ERC = 0`.
- [ ] **BOM & JLCPCB Basic Parts Optimization:** Generate `documents/schema/bom.json` and `documents/schema/semantics.json`, run `stingy-schematics` / `sync_semantics.py check`, and maximize JLCPCB Basic Parts usage.
- [ ] **Schematic Technical Review:** Export schematic PNGs (`documents/schema/*.png`) and run a full review per `review/guidelines.md`.

---

## Phase 3: PCB Layout, Routing & DFM (`documents/pcb/`)

- [ ] **Constraints & Floorplan:** Define `documents/pcb/constraints.json` (board outline, connector anchors, mounting holes, RF antenna keepout).
- [ ] **Placement (`pcb-placer`):** Run automated placement, ensure `< 2 mm` local decoupling and unobstructed 2.4 GHz antenna clearance, and generate `documents/pcb/placement.svg`.
- [ ] **Routing & Ground Stitching (`pcb-synthesizer` / `freerouting` / `pcb-stitcher`):** Complete signal/power routing, continuous GND planes, and perimeter stitching with `DRC = 0`.
- [ ] **Silkscreen (`pcb-silkscreen`):** Add functional connector pinout labels, polarity markers, and title block (`documents/pcb/silkscreen.json`).
- [ ] **DFM Audit (`pcb-dfm`):** Run JLCPCB DFM checks (solder mask bridges, via tenting, clearances) and resolve all alerts.

---

## Phase 4: Parametric Enclosure & Production Packaging (`documents/enclosure/`, `documents/production/`)

- [ ] **Parametric Enclosure (`pcb-enclosure`):** Configure `documents/pcb/enclosure.json` and generate `STEP` / `STL` enclosure shells with connector/antenna/ventilation clearances.
- [ ] **3D Interference Check:** Export PCBA `STEP`, convert to `pcba.stl` (`scripts/convert_step_to_stl.py`), and verify mechanical fit with zero collisions.
- [ ] **Production Deliverables:** Export Gerber, Drill, CPL, SMT BOM (`documents/production/`), and `documents/easyeda/OpenHVAC-ESP32.epro2`.

---

## Phase 5: Hardware Bring-up & Airton Integration

- [ ] **Benchtop Bring-up:** Verify `3.3V` rail, current draw, USB/UART flashing, and Zigbee 3.0 RF connectivity before connecting to the HVAC unit.
- [ ] **Airton HVAC Integration Test:** Connect to the Airton indoor unit, verify UART frame exchange (Tuya MCU / Airton protocol), and validate thermal/RF stability inside the housing.
