---
name: hw-specify
description: Captures tool-agnostic hardware requirements from a prompt, validates the project constitution (no placeholders), clarifies ambiguities, and initializes specs/<id>-<slug>/spec.md and checkpoint.md.
---

# `hw-specify` — Hardware Specification & Clarification

## Purpose
Turn a user hardware prompt into an unambiguous, EDA-agnostic specification inside `specs/<id>-<slug>/` before any component selection, netlist generation, or PCB layout.

## Workflow
1. **Validate Constitution (Blocking Gate):**
   - Read `.specify/memory/constitution.md`.
   - Verify that it exists and contains **no remaining placeholders** (e.g. `[PROJECT_NAME]`, `[BACKEND]`, `[FAB_HOUSE]`, `[DOMAIN_CONSTRAINTS]`).
   - If any placeholder remains, ask the user to supply the missing values and update `.specify/memory/constitution.md` before proceeding.
2. **Initialize Spec Directory:**
   - Create `specs/<NNN>-<slug>/` (sequential numbering `001`, `002`, ...).
3. **Clarify Ambiguities (Gate):**
   - Identify missing or ambiguous electrical, mechanical, or interface requirements (voltages, current budgets, bus logic levels, dimensions, connectors, optional 3D enclosure).
   - Ask concise, blocking questions and wait for user confirmation.
4. **Write `spec.md` & `checkpoint.md`:**
   - Write `specs/<id>/spec.md` describing **what** the board must do and **why** (zero mention of KiCad, EasyEDA, or component designators).
   - Initialize `specs/<id>/checkpoint.md` with:
     - `stage: specify (done)`
     - `next: hw-plan`
     - `last_gate: constitution validated & spec approved`
     - `blockers: none` (or list pending physical measurements)
