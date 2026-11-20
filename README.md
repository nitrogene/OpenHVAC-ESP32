# OpenHVAC-ESP32 — Zigbee HVAC Controller Hardware

[![Fabrication: JLCPCB](https://img.shields.io/badge/Fabricated%20%26%20Assembled%20by-JLCPCB-0072C6?style=for-the-badge)](https://jlcpcb.com)
[![EDA: EasyEDA Pro](https://img.shields.io/badge/Designed%20with-EasyEDA%20Pro-1890FF?style=for-the-badge)](https://pro.easyeda.com)
[![Protocol: Zigbee 3.0](https://img.shields.io/badge/Protocol-Zigbee%203.0-EB0443?style=for-the-badge)](https://csa-iot.org)
[![OSHW](https://img.shields.io/badge/Hardware-Open%20Source-brightgreen?style=for-the-badge)](https://oshwlab.com)

Open-source hardware engineering repository (schematic design, component dimensioning, PCB layout, 3D-printable enclosure, and JLCPCB manufacturing deliverables) for **OpenHVAC-ESP32**: a custom **ESP32 Zigbee** module designed to replace the stock **Tuya Wi-Fi module** in **Airton** split air conditioners.

This project is inspired by [**devildant/acw02_esphome**](https://github.com/devildant/acw02_esphome), which serves as a primary reference source for Airton / ACW02 hardware and protocol details.

> [!NOTE]
> **Project Ecosystem & Repository Scope:**
> - **This repository:** Dedicated exclusively to **hardware & mechanical engineering** (EasyEDA Pro CAD, schematics, PCB automation, DFM checks, 3D enclosure generation, and production files).
> - **Firmware:** Tracked in a **separate repository** (ESP-IDF / Zigbee stack).
> - **Home Assistant Integration:** The ultimate goal is to control the Airton HVAC unit via a **Home Assistant plugin/integration** (either leveraging an existing integration or developing a dedicated one in a **separate repository** if needed).

---

## 1. Project Objectives

- **Drop-in Tuya Wi-Fi Replacement:** Replace the proprietary Tuya Wi-Fi dongle/PCB on Airton HVAC indoor units with a local, cloud-free **Zigbee 3.0** controller.
- **ESP32 Zigbee Architecture:** Built around a native IEEE 802.15.4 / Zigbee ESP32 SoC (e.g., ESP32-C6 or ESP32-H2) with dedicated RF antenna keepout and clean 3.3V power regulation.
- **Safe Electrical Interface:** Match the Airton indoor unit connector pinout, supply rail, and UART logic levels (with level shifting / ESD protection if required).
- **Custom 3D Enclosure:** Parametric 3D-printable enclosure (`STEP` / `STL`) tailored to fit the Airton unit's internal mounting bay or external dongle housing.
- **DFM & Cost Optimization:** Designed for **JLCPCB SMT Assembly** prioritizing **JLCPCB Basic Parts** to minimize setup fees.

---

## 2. Repository Structure

```text
OpenHVAC-ESP32/
├── .agents/                    # Agnostic EDA/PCB/CAD automation skills & EasyEDA bridge hooks
├── .github/                    # Issue & PR templates
├── documents/
│   ├── datasheets/             # Reference datasheets & electrical notes
│   ├── easyeda/                # EasyEDA Pro project archive (OpenHVAC-ESP32.epro2)
│   ├── enclosure/              # Parametric 3D enclosure CAD (STEP / STL)
│   ├── pcb/                    # PCB constraints, manifest, silkscreen, DFM reports & schemas
│   ├── production/             # Gerber, Drill, BOM, Pick & Place deliverables
│   └── schema/                 # BOM, circuit semantics, schemas & schematic PNG renders
├── review/                     # Hardware technical review intake & guidelines
├── scripts/                    # Utility scripts (e.g., STEP -> STL converter)
├── AGENTS.md                   # AI & engineering workflow rules
├── learnings.md                # EasyEDA Pro API pitfalls & hardware design decisions
├── LICENSE                     # Dual license: CERN-OHL-P v2 (Hardware) & Apache 2.0 (Software)
├── pyproject.toml              # Python environment & dependencies (managed via uv)
└── todo.md                     # Hardware design & bring-up roadmap
```

---

## 3. Tooling & Automation Setup

This project uses `uv` for Python dependency management and a suite of modular hardware automation skills under `.agents/skills/`:

- `uv sync` — Install Python dependencies (`build123d`, `jsonschema`, `requests`, `pytest`).
- `pcb-placer` — Constraint-driven IPC-7351 & EMC component placement.
- `freerouting` & `pcb-synthesizer` — Headless Specctra auto-routing and closed-loop synthesis.
- `pcb-stitcher` — Ground plane continuity and stitching vias.
- `pcb-dfm` — Automated JLCPCB DFM & SMT assembly checks.
- `pcb-silkscreen` — Declarative silkscreen labeling.
- `pcb-enclosure` — Parametric 3D enclosure generation (`STEP` & `STL`).
- `stingy-schematics` — BOM & JLCPCB Basic Parts optimizer + semantic synchronization.

---

## 4. License

- **Hardware (Schematics, PCB, Production Files, 3D Enclosure):** [CERN-OHL-P v2](LICENSE)
- **Software & Automation Tooling:** [Apache License 2.0](LICENSE)
