# Technical Knowledge Base & EasyEDA Pro Pitfalls Guide

This document serves as the **technical knowledge base** for the **OpenHVAC-ESP32** project. It catalogs non-obvious EasyEDA Pro API pitfalls and critical hardware design decisions that must not be regressed.

> [!NOTE]
> Mandatory workflow rules live in [`AGENTS.md`](AGENTS.md). Reusable algorithms and calculations live in `.agents/skills/<name>/`.

---

## 1. Knowledge Management Protocol

Any newly discovered anomaly or technical insight follows this lifecycle:
1. **Record** symptom, root cause, and verified workaround in this file.
2. **Route** to the right home:
   - Mandatory process/safety directive $\rightarrow$ [`AGENTS.md`](AGENTS.md)
   - Reusable algorithm/calculation $\rightarrow$ `.agents/skills/<name>/`
   - EasyEDA Pro API quirk $\rightarrow$ Section 2 below
   - Non-obvious hardware design decision / trap $\rightarrow$ Section 3 below
3. **Prune** periodically once knowledge is codified into a skill or rule.

---

## 2. Quick-Reference Cheat Sheets: EasyEDA Pro API Pitfalls & Golden Rules

### A. Schematic (`eda.sch_*`)

| Pitfall / Behavior | Cause & Risk | Proven Rule & Solution |
| :--- | :--- | :--- |
| **Network flag (`NetFlag`) without physical wire** | Placing a NetFlag directly onto a pin pad without a physical wire segment causes EasyEDA to treat the pin as **floating**, generating an orphan net in the PCB netlist (`$1N...`). | **Always insert a physical wire (`sch_PrimitiveWire.create([x1, y1, x2, y2])`)** between the pin pad and the NetFlag. |
| **"Multiple net names" warning on wire** | Assigning an explicit net name to a wire segment already connected to a NetFlag or NetPort triggers `[Warn] : Wire has multiple net names`. | **Omit the net name parameter** during `sch_PrimitiveWire.create(line)` if the wire connects to a NetPort/NetFlag: net propagation is automatic. |
| **Empty stubs in `createNetLabel`** | `eda.sch_PrimitiveAttribute.createNetLabel()` is an unimplemented empty stub in the current runtime. | **Use NetPorts instead:** `eda.sch_PrimitiveComponent.createNetPort(...)`, which immediately binds any overlapping wire. |
| **Component instantiation in scripts** | Passing a lightweight object `{ libraryUuid, uuid }` stalls execution with a 30-second timeout. | **Resolve the full Device object first** via `eda.lib_Device.search(...)` before calling `sch_PrimitiveComponent.create(devs[0], ...)`. |
| **Schematic Y-axis orientation** | On a schematic sheet (`documentType: 1`), **the Y-axis points UPWARD** (inverted compared to the PCB canvas coordinate system). | Always verify coordinates: `y = 0` is at the bottom of the A4 page, `y = 825` is at the top. |
| **Multi-sheet enumeration (`sch_PrimitiveComponent.getAll`)** | `eda.sch_PrimitiveComponent.getAll(undefined, true)` may omit components if their corresponding sheets have not been opened in the editor (not loaded in memory). | **Iterate through sheets via `eda.dmt_Schematic.getAllSchematicPagesInfo()`**, activate each tab with `eda.dmt_EditorControl.openDocument(page.uuid)`, then call `getAll()` sheet by sheet. |
| **Graphic `LINE` crossing a passive body** | A schematic `LINE` primitive accidentally crossing the body of a passive component shorts its two pads in the netlist without visual warnings. | Audit primitive continuity and never draw graphic lines across component pins/bodies. |
| **Multiline text annotations (`sch_PrimitiveText.create`)** | Multi-line text strings can require separate calls in some EDA tools. | `eda.sch_PrimitiveText.create` natively supports `\n` newlines. |
| **Automated project export (`sys_FileManager.getProjectFile`)** | `eda.sch_ManufactureData.getPngFile` is unavailable in some EasyEDA Pro runtimes. | Use `eda.sys_FileManager.getProjectFile('OpenHVAC-ESP32', undefined, 'epro2')` to extract the binary archive. |

### B. PCB Layout & Primitives (`eda.pcb_*`)

| Pitfall / Behavior | Cause & Risk | Proven Rule & Solution |
| :--- | :--- | :--- |
| **Signature of `pcb_PrimitiveVia.create` (CRITICAL)** | The API signature is `(net, x, y, holeDiameter, diameter)`. Inverting drill diameter and outer diameter creates an invalid via and blocks DRC. | **The drill diameter always precedes the outer diameter.** JLCPCB standard: `hole = 12 mil` (~0.3 mm), `diameter = 24 mil` (~0.6 mm). |
| **Signature of `pcb_PrimitiveLine.create` (CRITICAL)** | The API signature is `(net, layer, startX, startY, endX, endY, width)`. Placing width before coordinates offsets all arguments. | **Coordinates always precede track width.** |
| **Runtime units** | PCB API coordinates are expressed in **mils** (1 mil = 0.0254 mm, 1 mm = 39.3701 mil). | Always explicitly convert using `mm_to_mil()` and `mil_to_mm()`. |
| **Layer identifiers (Layers)** | Layers are referenced by integers: `1` = Top Layer, `2` = Bottom Layer, `11` = Board Outline, `12` = Multi-layer. | Use integer constants or the `EPCB_LayerId` enumeration. |
| **Moving / relocating a via** | Modifying existing via coordinates in place fails to trigger recalculation of copper clearance regions during polygon pouring. | **Delete the old via** (`delete([oldId])`), **create the new via**, then call `rebuildCopperRegion()`. |
| **Differential pair chamfering** | 90° bends cause impedance discontinuities and EMI radiation (critical for USB). | **Always use 45° mitered corners** (Δx = Δy = 25 mil for a 10 mil track). |
| **Pad geometry extraction (`Pad`)** | Methods `getState_Width()` / `getState_Height()` do not exist on `pcb_PrimitivePad`. | **Use `p.getState_Pad()`** (`['RECT', w, h, r]`) and `p.getState_Hole()` (`['ROUND', drill]`). |

### C. Copper Pours, DRC & WebGL Cache

| Pitfall / Behavior | Cause & Risk | Proven Rule & Solution |
| :--- | :--- | :--- |
| **Persistence of old track labels (WebGL Cache)** | After updating a net via the API, the text displayed on the WebGL canvas retains the old name as long as the tab remains open. | **Close and reopen the PCB document**: `save()`, `closeDocument(tabId)`, then `openDocument(pcbUuid)`. |
| **Copper micro-slivers between SMD pads** | Ground plane pouring can introduce parasitic copper slivers between tightly spaced pads (0603/0805). | Insert a local `NO_POURS` keepout region (`ruleType: [7]`) between pads via `pcb_PrimitiveRegion.create()`. |
| **Multi-layer RF antenna keepout** | An ESP32 PCB/chip antenna requires a total copper void under and around the 2.4 GHz radiator across all layers. | Create a region on layer `12` (`MULTI`) with `ruleType: [5, 6, 7]` (`NO_WIRES`, `NO_FILLS`, `NO_POURS`). |
| **Ground plane continuity verification** | After calling `rebuildCopperRegion()`, verify that the plane has not fragmented into isolated islands. | Check via `pcb_PrimitivePoured.getAll()` that `pouredCount == 2` (1 Top region, 1 Bottom region). |

---

## 3. Critical Hardware Design Decisions & Reference Sources (Do Not Regress)

- **Primary Reference Source:** [devildant/acw02_esphome](https://github.com/devildant/acw02_esphome) — Reference project for Airton / ACW02 hardware reverse engineering, pinout/electrical interface, and protocol details.

*(To be populated further as Airton HVAC interface, power supply, and ESP32 Zigbee hardware decisions are validated.)*
