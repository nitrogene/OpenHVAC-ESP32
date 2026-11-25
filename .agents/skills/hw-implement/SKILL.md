---
name: hw-implement
description: Executes specs/<id>/tasks.md on the active backend defined in constitution.md (e.g. kicad-headless) to produce DRC-clean PCB layout, Gerbers, CPL, BOM, and optional 3D STEP/STL deliverables.
---

# `hw-implement` — Headless PCB Synthesis & Manufacturing Export

## Purpose
Execute `specs/<id>/tasks.md` step-by-step on the active backend declared in `.specify/memory/constitution.md`, updating `tasks.md` and `checkpoint.md` after every completed gate so work can be paused and resumed across sessions.

## Inputs
- `.specify/memory/constitution.md` (determines active backend, e.g. `kicad-headless`)
- `specs/<id>/checkpoint.md` & `specs/<id>/tasks.md`
- `specs/<id>/netlist.json`, `bom.json`, `constraints.json`

## Workflow
1. **Resume Protocol:**
   - Read `specs/<id>/checkpoint.md` and `specs/<id>/tasks.md` to identify the exact next unchecked step.
2. **Libraries & Board Init (`pcb/`):**
   - Resolve 100% of footprints and 3D models (`lcsc-lib`) with JLCPCB 0° CPL orientation checks.
   - Initialize `board_init.kicad_pcb` with `Edge.Cuts`, mounting holes, and footprints.
3. **Closed-Loop Placement, Routing & Metrology:**
   - Run `pcb-placer` -> `freerouting` (DSN/SES) -> `pcb-synthesizer` metrology.
   - Loop back to placement on EMC threshold failure (max 3 consecutive attempts per constitution).
4. **Copper, Silkscreen & Certification:**
   - Fill GND planes and stitching vias (`pcb-stitcher`), add legible labels (`pcb-silkscreen`).
   - Run `kicad-cli pcb drc --exit-code-violations` (`DRC = 0`) and `pcb-dfm` -> `reports/drc_report.json`, `reports/dfm_report.md`.
5. **Production & Optional 3D Export (`production/`):**
   - Export Gerbers, Drill, CPL, SMT BOM, `render_top.png` / `render_bot.png`, and optional `STEP` / `STL` enclosure fit check.
6. **Checkpoint Sync:**
   - Mark completed items `[x]` in `specs/<id>/tasks.md` and update `specs/<id>/checkpoint.md` at every milestone.
