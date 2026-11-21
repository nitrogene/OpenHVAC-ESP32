# Project Roadmap & Active Checklist (TODO)

This document constitutes the operational action plan for **OpenHVAC-ESP32** (custom `ESP32-C6-WROOM-1-N4` PCB supporting Wi-Fi/ESPHome and Zigbee 3.0, built on the `devildant/acw02_esphome` Forhorse v2 mechanical envelope for Airton air conditioners).

**Locked Architecture Baseline:**
- **Mechanical Envelope:** Forhorse v2 outline (`54.10 × 42.29 mm`), `M3`/`M2` mounting holes, and anchored positions for `JST XA 2.5` 4-pin HVAC connector, `USB-C`, `BOOT`, and `RST` (reference files archived in `documents/forhorse/`). Extra headers (`1×5P`, `1×6P`, `JST XH 2P`) and `12V` jumper are omitted.
- **MCU:** `ESP32-C6-WROOM-1-N4` (`4 MB` Flash, Wi-Fi 6 + IEEE 802.15.4 Zigbee/Thread + native USB-Serial/JTAG).
- **Power & Protection:** Direct synchronous `12V -> 3.3V` Buck (`AP63203WU-7`) with bulk capacitance for Airton brownout mitigation, `AMS1117-5.0` Basic Part LDO (`5V` rail for HVAC-side UART level shifters only), reverse-polarity P-MOSFET (`AO3401A`), input TVS, and USB-C ESD protection (`USBLC6-2SC6`).
- **UART Interface:** `2×` unidirectional level shifters (`74LVC1T45W6-7`) for `TX` and `RX`.

---

## Phase 1: Component List, Datasheets & Sizing

- [ ] **Stock Airton Hardware Verification:** Power off the AC unit at the breaker, inspect the stock Wi-Fi module, and confirm `12V` rail, `5V` UART logic levels, and `JST XA 2.5` 4-pin wiring against `devildant/acw02_esphome`.
- [ ] **LCSC Component List & Stock Verification:** Finalize exact LCSC part numbers and live JLCPCB stock (maximizing Basic Parts) for all functions: `ESP32-C6-WROOM-1-N4`, `AP63203WU-7` synchronous Buck, `4.7 µH` shielded inductor, `AMS1117-5.0` LDO, `2× 74LVC1T45W6-7`, `AO3401A`, input TVS, `USBLC6-2SC6`, `USB-C` receptacle, `JST XA 2.5` 4-pin header, `BOOT`/`RST` tactile switches, status LEDs, and passives.
- [x] **Datasheets Archive:** Collect and archive all component datasheets under `documents/datasheets/references/` and index them in `documents/datasheets/datasheets.md`.
- [x] **Power & Passive Sizing:** Validate `AP63203WU-7` external passives (`4.7 µH`, `2× 22 µF`, `100 nF` BST, `44 µF` `25V` bulk against Airton `12V` brownouts), `AO3401A` `Vgs` divider (`100 kΩ / 100 kΩ`), `AMS1117-5.0` `5.3 mA` min load (`PWR` LED), `ESP32-C6` `EN` RC (`10 kΩ + 1 µF`) & `GPIO8` pull-up, and `USB-C` `5.1 kΩ` CC + `22 Ω` series resistors.

## Phase 2: `Prompt -> Schematic` Pipeline Development (`.agents/skills/sch-synthesizer/`)

- [x] **Declarative Netlist Schema (`documents/schema/netlist.schema.json`):** Define an agnostic intermediate JSON schema (`documents/schema/netlist.json` created and validated) describing functional blocks, LCSC part numbers, and explicit net-to-pin connections (no `$1N...` auto-nets).
- [ ] **LCSC-to-EasyEDA Device Resolver:** Implement `eda.lib_Device.search()` resolution mapping LCSC parts to `(deviceUuid, libraryUuid)` with a user validation gate before schematic instantiation.
- [ ] **Block Placer & Pin Coordinate Reader:** Implement functional-block schematic placement and query exact pin coordinates via `getAllPinsByPrimitiveId()`.
- [ ] **Orthogonal Wire & NetPort Router:** Implement orthogonal `SCH_PrimitiveWire` routing and `NetFlag` / `NetPort` placement (always connected via physical wires, never directly on pins).
- [ ] **Closed-Loop ERC Certification:** Integrate `eda.sch_Drc.check()` verification loop (max 3 consecutive retry attempts) targeting `ERC = 0`.

---

## Phase 3: Schematic Generation, Verification & Documentation (`documents/schema/`)

- [ ] **Schematic Synthesis (EasyEDA Pro):** Run the `Prompt -> Schematic` pipeline for OpenHVAC-ESP32, verify all connections and symbols in EasyEDA Pro, and certify `ERC = 0`.
- [ ] **Documentation & BOM Sync:** Write `documents/hardware.md`, generate `documents/schema/bom.json` and `documents/schema/semantics.json` using the verified schematic designators, and run `uv run python .agents/skills/stingy-schematics/scripts/sync_semantics.py check`.
- [ ] **Schematic Technical Review:** Export schematic PNGs (`documents/schema/*.png`) and run a full audit per `review/guidelines.md`.

---

## Phase 4: PCB Layout, Routing & DFM (`documents/pcb/`)

- [ ] **Constraints & Floorplan:** Generate `documents/pcb/constraints.json` from `documents/forhorse/pcb/` (`54.10 × 42.29 mm` outline polygon, `M3`/`M2` NPTH mounting holes, anchored `USB-C`, `BOOT`, `RST`, and `JST XA 2.5` coordinates, plus `ESP32-C6-WROOM-1` `2.4 GHz` antenna keepout).
- [ ] **Placement (`pcb-placer`):** Run automated placement, enforce `< 2 mm` local decoupling and unobstructed `2.4 GHz` antenna clearance, and generate `documents/pcb/placement.svg`.
- [ ] **Routing & Ground Stitching (`pcb-synthesizer` / `freerouting` / `pcb-stitcher`):** Complete signal/power routing, continuous GND planes, and perimeter stitching with `DRC = 0`.
- [ ] **Silkscreen (`pcb-silkscreen`):** Add functional connector pinout labels (`12V`, `RX`, `TX`, `GND`), polarity markers, and title block (`documents/pcb/silkscreen.json`).
- [ ] **DFM Audit (`pcb-dfm`):** Run JLCPCB DFM checks (solder mask bridges, via tenting, clearances, BOM/CPL match) and resolve all alerts.

---

## Phase 5: 3D Fit Check & Production Deliverables (`documents/enclosure/`, `documents/production/`)

- [ ] **3D Interference Check:** Export PCBA `STEP`, convert to `pcba.stl` (`scripts/convert_step_to_stl.py`), and verify mechanical fit with zero collisions against the Forhorse 3D enclosure (`documents/forhorse/3d/full pcb acw02 case bot with air flow.stl` and v2 top covers).
- [ ] **Production Deliverables:** Export Gerber, Drill, CPL, SMT BOM (`documents/production/`), and `documents/easyeda/OpenHVAC-ESP32.epro2` for JLCPCB PCBA order.

---

## Phase 6: Hardware Bring-up, Firmware & Airton Integration

- [ ] **Benchtop Bring-up (`12V` Supply):** Verify `3.3V` Buck rail stability/ripple, `5V` LDO rail, current draw under RF TX peaks, and native USB-C flashing before connecting to the HVAC unit.
- [ ] **Firmware & Home Assistant Integration:** Flash either ESPHome (Wi-Fi, `devildant/acw02_esphome` UART protocol) or custom Zigbee 3.0 firmware (`esp-zigbee-sdk` + Zigbee OTA + Z2M External Converter / ZHA quirk).
- [ ] **Airton HVAC Integration Test:** Connect to the Airton indoor unit, verify UART frame exchange, confirm absence of brownouts, and validate thermal/RF stability inside the PETG enclosure.
