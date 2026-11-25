# OpenHVAC-ESP32 — Zigbee / Wi-Fi HVAC Controller Hardware

[![Fabrication: JLCPCB](https://img.shields.io/badge/Fabricated%20%26%20Assembled%20by-JLCPCB-0072C6?style=for-the-badge)](https://jlcpcb.com)
[![EDA: KiCad Headless](https://img.shields.io/badge/EDA-KiCad%20Headless-314CB0?style=for-the-badge)](https://www.kicad.org)
[![Protocol: Zigbee 3.0](https://img.shields.io/badge/Protocol-Zigbee%203.0-EB0443?style=for-the-badge)](https://csa-iot.org)
[![OSHW](https://img.shields.io/badge/Hardware-Open%20Source-brightgreen?style=for-the-badge)](LICENSE)

Open-source hardware repository for **OpenHVAC-ESP32**: a custom **`ESP32-C6-WROOM-1-N4`** PCB (Wi-Fi 6 / ESPHome & Zigbee 3.0) designed to replace the stock Tuya Wi-Fi module in **Airton** split air conditioners.

Inspired by [**devildant/acw02_esphome**](https://github.com/devildant/acw02_esphome) and the [**Forhorse v2**](https://github.com/devildant/acw02_esphome/blob/main/docs/version_Forhorse.md) mechanical envelope (`54.10 × 42.29 mm`).

> [!NOTE]
> **Spec-Driven Hardware Workflow:**
> Engineering rules live in [`.specify/memory/constitution.md`](.specify/memory/constitution.md). Each board design/revision is executed via `hw-specify` → `hw-plan` → `hw-implement` inside its own `specs/<id>-<slug>/` workspace.

---

## Repository Structure

```text
OpenHVAC-ESP32/
├── .specify/
│   └── memory/
│       ├── constitution-template.md   # Generic hardware constitution template
│       └── constitution.md            # Active project constitution (KiCad headless, JLCPCB)
├── .agents/
│   └── skills/                        # Spec-driven hardware workflow skills (hw-specify, hw-plan, hw-implement)
├── AGENTS.md                          # Agent entry point -> .specify/memory/constitution.md
├── LICENSE                            # CERN-OHL-P v2 (Hardware) & Apache 2.0 (Software)
├── README.md                          # Project overview
└── todo.md                            # Active roadmap (Part 1: hw-* skills validation, Part 2: Airton board)
```

*(Note: Previous EasyEDA-oriented files are preserved in Git history under tag `archive-easyeda`.)*
