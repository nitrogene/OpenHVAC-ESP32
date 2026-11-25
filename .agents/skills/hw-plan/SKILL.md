---
name: hw-plan
description: Translates specs/<id>/spec.md into executable hardware JSON contracts (specs.json, bom.json, semantics.json, netlist.json, constraints.json), runs programmatic ERC and consistency checks, and generates tasks.md.
---

# `hw-plan` — Electrical Architecture, Netlist & ERC

## Purpose
Convert `specs/<id>/spec.md` into verified, EDA-agnostic JSON contracts and an executable `tasks.md` checklist, certifying `ERC = 0` before any PCB layout begins.

## Inputs
- `.specify/memory/constitution.md`
- `specs/<id>/spec.md`
- `specs/<id>/checkpoint.md`

## Workflow
1. **Analog & Power Sizing (`specs.json`):**
   - Run sizing calculations (e.g. `ratio-solver`, `buck-compensation`) and record margins and tolerances.
2. **BOM & Semantics (`bom.json`, `semantics.json`):**
   - Select JLCPCB parts (prioritizing Basic Parts, verifying stock) via `stingy-schematics`.
3. **Declarative Netlist & Constraints (`netlist.json`, `constraints.json`):**
   - Generate explicit pin-to-pin connectivity (no `$1N...` auto-nets) and mechanical/EMC constraints (board outline, mounting holes, anchored connectors, RF keepouts, `< 2 mm` decoupling rules).
4. **Analyze & Programmatic ERC (Blocking Gate):**
   - Validate `netlist.json` against schema and check `bom.json` ↔ `semantics.json` consistency.
   - Run programmatic ERC (`ERC = 0`: no floating inputs, no output-to-output shorts, all IC power rails connected, logic levels compatible) and output `reports/erc_report.json`.
   - Generate a concise functional block diagram (Mermaid/SVG) for quick human sign-off.
5. **Generate `tasks.md` & Update `checkpoint.md`:**
   - Write `specs/<id>/tasks.md` with ordered implementation steps and measurable pass gates.
   - Update `specs/<id>/checkpoint.md` (`stage: plan (done)`, `next: hw-implement`).
