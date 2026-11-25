# Hardware Engineering Constitution — [PROJECT_NAME]

> **Version:** [VERSION] | **Last Updated:** [DATE]
> **Active Backend:** `[BACKEND]` (e.g., `kicad-headless` | `easyeda`)

---

## 1. Non-Negotiable Engineering Gates
- **ERC = 0 before layout:** No floating inputs, no output-to-output conflicts, all IC power rails connected, logic levels verified.
- **DRC = 0 before production:** Zero design rule violations reported by the active backend (`kicad-cli pcb drc --exit-code-violations`).
- **DFM & BOM compliance:** Prioritize JLCPCB **Basic Parts** to minimize SMT setup fees; verify live stock and CPL 0° orientation before finalizing production files.
- **Explicit net naming:** Auto-generated net names (`$1N...`) are strictly forbidden in `netlist.json` and PCB files.

## 2. Physical, EMC & Layout Invariants
- **Decoupling proximity:** Local decoupling capacitors must sit `< 2 mm` from IC power pins.
- **RF keepout:** Keep copper, ground planes, and traces clear of antenna keepout zones when wireless modules are present.
- **Autonomy limit:** Maximum **3 consecutive automated retry attempts** for any recurring ERC, EMC metrology, or DRC failure. On a 3rd failure, stop cleanly, save state in `checkpoint.md`, and escalate to the user.

## 3. Tooling & Execution Rules
- **Python via `uv`:** All Python scripts and tools must be executed via `uv run` (never bare `python`).
- **No LaTeX for simple values:** Use plain Markdown code for designators and values (e.g., `U1`, `R1`, `10 kΩ`, `3.3V`). Reserve `$ ... $` for complex mathematical equations only.

## 4. Workflow & Multi-Session Resumption (`specs/<id>/`)
- Every hardware design or revision lives in its own isolated directory: `specs/<id>-<slug>/`.
- **`hw-specify`** validates this constitution (no remaining placeholders), clarifies ambiguities, and produces `spec.md` + `checkpoint.md`.
- **`hw-plan`** produces executable JSON contracts (`specs.json`, `bom.json`, `semantics.json`, `netlist.json`, `constraints.json`), runs ERC + consistency checks, and generates `tasks.md`.
- **`hw-implement`** executes `tasks.md` step-by-step on `[BACKEND]`, updating `tasks.md` and `checkpoint.md` at every completed gate.
- **Session resumption:** At the start of any session, read `todo.md` and the active `specs/<id>/checkpoint.md` (+ `git log -n 5 --oneline`) before proposing next actions.

## 5. Project-Specific Constraints
- **Target Manufacturer & Stackup:** [FAB_HOUSE] — [PCB_STACKUP]
- **Domain & Mechanical Constraints:** [DOMAIN_CONSTRAINTS]
