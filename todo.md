# Project Roadmap & Active Checklist (TODO)

This document constitutes the operational action plan for **OpenHVAC-ESP32** (custom ESP32 Zigbee PCB + enclosure replacing the Tuya Wi-Fi module on an Airton air conditioner).

---

## Phase 1: Mechanical Baseline (`devildant/acw02_esphome`) & Electrical Specifications

- [ ] **Mechanical Envelope & Connector Baseline:** Extract PCB dimensions, mounting/case clearances, and connector footprint/orientation (`JST XA 2.5` 4-pin or variant) from `devildant/acw02_esphome`, and verify physical fit against the target Airton indoor unit.
- [ ] **Airton Bus & Electrical Verification:** Confirm the target unit variant (`12V` ACW02 header vs `5V` USB variant), pinout (`VCC`, `GND`, `UART_TX`, `UART_RX`), UART logic levels (`5V` vs `3.3V`), and `12V` rail current/brownout behavior.
- [ ] **Integrated SMD Architecture Selection (`documents/hardware.md`):**
  - Select integrated Zigbee ESP32 SMD module (e.g., `ESP32-C6-MINI-1` or `ESP32-C6-WROOM-1`), checking JLCPCB stock and Extended Part cost.
  - Define a direct `12V` to `3.3V` Buck converter stage (replacing `12V -> 5V -> LDO`), including input bulk capacitance for brownout mitigation, reverse-polarity protection, and TVS/ESD protection.
  - Define an integrated SMD UART level-shifting stage (`5V` <-> `3.3V`) with ESD protection on `TX`/`RX`.
  - Define programming & debug interface (USB-C using native ESP32-C6 USB-Serial/JTAG, or compact UART test pads + `BOOT`/`RESET` buttons).

---

## Phase 2: Schematic Design, Sizing & BOM (`documents/schema/`)

- [ ] **Datasheets Archive:** Collect and archive all component datasheets under `documents/datasheets/references/` and index them in `documents/datasheets/datasheets.md`.
- [ ] **Power & Feedback Sizing:** Validate Buck converter loop compensation (`buck-compensation` if TI `TPS543xx` family is selected) and resistor divider ratios (`ratio-solver`).
- [ ] **Schematic Capture (EasyEDA Pro):** Build schematic with explicit net names, physical wires on all NetPorts, and `ERC = 0`.
- [ ] **BOM & JLCPCB Cost Optimization:** Generate `documents/schema/bom.json` and `documents/schema/semantics.json`, run `stingy-schematics` / `sync_semantics.py check`, and minimize Extended Parts setup fees (`$3`/ref) in favor of JLCPCB Basic Parts.
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

- [ ] **Parametric Enclosure (`pcb-enclosure`):** Configure `documents/pcb/enclosure.json` aligned with the `devildant/acw02_esphome` mechanical envelope, adding cutouts/clearances for the integrated SMD layout (USB-C/buttons if present, JST exit, 2.4 GHz antenna clearance), and generate `STEP` / `STL` shells.
- [ ] **3D Interference Check:** Export PCBA `STEP`, convert to `pcba.stl` (`scripts/convert_step_to_stl.py`), and verify mechanical fit with zero collisions.
- [ ] **Production Deliverables:** Export Gerber, Drill, CPL, SMT BOM (`documents/production/`), and `documents/easyeda/OpenHVAC-ESP32.epro2` for JLCPCB PCBA order.

---

## Phase 5: Hardware Bring-up & Airton Integration

- [ ] **Benchtop Bring-up (`12V` Supply):** Verify `3.3V` Buck rail stability/ripple, current draw under Zigbee TX peaks, USB/UART flashing, and Zigbee 3.0 RF connectivity before connecting to the HVAC unit.
- [ ] **Airton HVAC Integration Test:** Connect to the Airton indoor unit, verify UART frame exchange (ACW02 / Airton protocol), confirm absence of brownouts, and validate thermal/RF stability inside the housing.
