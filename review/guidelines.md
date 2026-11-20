# Reviewer Guidelines — OpenHVAC-ESP32

Welcome! This document outlines instructions, technical requirements, and reporting standards for any **reviewer** (AI agent or human engineer) auditing the **OpenHVAC-ESP32** project (custom Zigbee ESP32 PCB + enclosure replacing the Tuya Wi-Fi module on an Airton air conditioner).

The goal of a technical review is to provide rigorous critical scrutiny on the electrical schematic, PCB layout, routing, RF/Zigbee antenna integration, HVAC interface safety, bill of materials (BOM), or enclosure CAD, ensuring reliable, safe, and cost-effective hardware.

---

## 1. Format & Naming Conventions

### File Naming
Submit your review inside the `review/` directory using the following format:
`review/reviewXXX.md` *(e.g. `review001.md`, `review_schematic_01.md`, `review_pcb_02.md`)*.

> [!NOTE]
> **Lifecycle:** Once your review is placed in `review/`, the project team (user and AI coding assistant) will evaluate your observations, arbitrate each item, integrate accepted action points into `todo.md`, and then **delete** your `reviewXXX.md` file. Deletion signifies full resolution and keeps the intake clean for subsequent reviews.

### Mandatory Metadata Header
Every review **must begin** with this metadata block to guarantee full audit traceability:

```markdown
# Technical Review: [Short and descriptive title]

- **Reviewer / Agent Name:** [E.g. Claude, Gemini, GPT, Hardware Expert, etc.]
- **Date:** [YYYY-MM-DD]
- **Audited Scope:** [Schematic / PCB Layout / Routing / BOM / Enclosure / Architecture]
- **Examined Reference / Version:** [Git commit hash or document version]
- **Overall Verdict:** [🟢 Approved / 🟠 Approved with Reservations / 🔴 Rejected (Critical fixes required)]
```

---

## 2. Project Context, Audit Axes & Pitfalls to Avoid

To prevent false alarms and off-topic observations, you **must integrate** the following fundamental project constraints:

### 2.0 Universal Auditor Mindset: Tabula Rasa & Evidence-Based Scrutiny
- If you are an AI model with conversational memory on this repository or an engineer familiar with its history, you **must completely disregard past discussions**. Adopt the stance of an impartial third-party auditor discovering the board fresh.
- Every observation must be **strictly grounded in versioned repository files** (schematic PNG renders, bill of materials [`bom.json`](../documents/schema/bom.json), official datasheets archived under [`documents/datasheets/references/`](../documents/datasheets/references/), and the technical reference [`datasheets.md`](../documents/datasheets/datasheets.md)).
- **Mandatory Critical Thinking:** Never take text claims in documentation at face value. Systematically cross-check every guideline and statement against manufacturer datasheets.

### 2.1 Project Context & Known Constraints

1. **Target Application: Airton HVAC Tuya Wi-Fi Module Replacement**
   - The board replaces the stock Tuya Wi-Fi dongle/module inside or attached to an Airton split air conditioner indoor unit.
   - Supply voltage, connector pinout, and UART/logic levels must match the reverse-engineered Airton indoor unit interface documented in [`hardware.md`](../documents/hardware.md).

2. **Target MCU: Zigbee-Capable ESP32 (e.g. ESP32-C6 / ESP32-H2)**
   - Verify exact datasheet pinouts, strapping pins (`GPIO8`, `GPIO9`, `EN`), RF antenna keepout requirements, and 3.3V logic-level compatibility (ESP32 I/Os are **not 5V-tolerant**).

3. **Manufacturing Constraints & JLCPCB Cost Optimization**
   - **Absolute Priority for JLCPCB *Basic Parts*:** Any modification of passive or active components should favor Basic Parts from the LCSC/JLCPCB catalog to eliminate additional reel changeover fees (*Extended Part fee*).

4. **Consult Current Progress State**
   - Before raising an alert, quickly check [`todo.md`](../todo.md), [`hardware.md`](../documents/hardware.md), and [`bom.json`](../documents/schema/bom.json).

### 2.2 Hardware Audit Axes
- **Power Distribution & Electrical Survival:** Input voltage margins, reverse-polarity / ESD protection, regulator thermal dissipation (`PD = ΔV × I`), and current budget during RF TX peaks.
- **HVAC Interface & Logic Levels:** UART TX/RX level shifting or clamping if the Airton bus operates at 5V, pull-up/pull-down compatibility, and connector pinout accuracy.
- **MCU & Strapping Pins:** `EN` RC delay, boot strapping pull-ups/pull-downs, USB/UART programming header accessibility.
- **RF & Antenna Integration:** 2.4 GHz Zigbee antenna clearance, multi-layer copper keepout, ground plane return path, and enclosure material dielectric impact.
- **Thermals, Packages & BOM (DFM):** JLCPCB *Basic Parts* optimization, solder mask dams, via tenting, and mechanical fit inside the Airton unit / custom enclosure.

---

## 3. Mandatory Criticality Classification

Observations must be strictly categorized into 3 severity levels:

### 🔴 1. Blocking / Critical (*High / Critical*)
*Issues that prevent circuit operation or present component destruction risks.*
* Component destruction risk, overvoltage (e.g. 5V UART injected into 3.3V ESP32 GPIO), overcurrent, unprotected reverse polarity.
* Absolute Maximum Ratings exceeded (`VIN`, `VGPIO`, junction temperature `TJ`).
* Hard short circuits, blocking ERC/DRC violations, floating critical pins (Reset, Boot, Power rails).
* Swapped TX/RX or mismatched Airton connector pinout.

### 🟠 2. Important (*Medium*)
*Issues that compromise robustness, RF performance, EMC compliance, or long-term reliability.*
* Insufficient decoupling or capacitors placed too far (`> 2 mm` from power pins).
* Compromised 2.4 GHz Zigbee antenna keepout or ground plane discontinuity.
* Missing ESD protection on external HVAC / USB connectors.
* Insufficient noise immunity (`EN` pin lacking RC filter).

### 🟢 3. Minor / Suggestion (*Low / Info*)
*Opportunities to optimize cost, manufacturability, or ergonomics.*
* Bill of Materials (BOM) optimization: substituting an *Extended Part* with an equivalent *Basic Part*.
* Silkscreen clarity (connector pinout labels, polarity markers, boot/reset indicators).
* Indicator visibility or test point accessibility.
* Schematic or documentation formatting and layout polish.

---

## 4. Expectations for High-Quality Feedback

For observations to be directly actionable, each remark must provide:

1. **Pinpoint Precision:**
   - Cite **exact designators** (e.g. `U1`, `R2`, `C3`, `J1`).
   - Name **exact nets** (e.g. `+5V_HVAC`, `3.3V`, `UART_TX`, `ESP_EN`).
   - Specify **pin number and pin name**.

2. **Technical Demonstration (Why?):**
   - Support analysis with **calculations**, **formulas**, or **datasheet excerpts**.

3. **Turnkey Actionable Solution (How to fix?):**
   - Provide concrete parameters: value, tolerance, package/footprint (`0603`, `0805`, `SOT-23`...), exact manufacturer part number, and **LCSC code**.
   - Note whether the proposal qualifies as a JLCPCB *Basic Part*.

4. **New Components to Add (No advance designators):**
   - When suggesting new components, **do not assign speculative designators**.
   - Describe components strictly by function, value, footprint, and constraints (e.g. *« Add a 10 kΩ 0603 pull-up on Boot »*).
   - The final sequential designator will be assigned at schematic integration time according to the next available number in the BOM.

---

## 5. Review Template (`reviewXXX.md`)

Copy and complete the template below in your `review/reviewXXX.md` file:

```markdown
# Technical Review: [Short title of your audit]

- **Reviewer / Agent Name:** [Your name / AI model]
- **Date:** [YYYY-MM-DD]
- **Audited Scope:** [Schematic / PCB Layout / BOM / Enclosure]
- **Examined Reference / Version:** [Git commit hash or document version]
- **Overall Verdict:** [🟢 Approved / 🟠 Approved with Reservations / 🔴 Rejected]

---

## Executive Summary
[3 to 5 sentence summary on overall design health, compliance, and key priorities.]

---

## Observations Summary

| Ref | Severity | Target / Scope | Summary Description |
| :--- | :--- | :--- | :--- |
| **B1** | 🔴 Blocking | `...` | ... |
| **I1** | 🟠 Important | `...` | ... |
| **M1** | 🟢 Minor | `...` | ... |

---

## Detailed Observations & Recommendations

### 🔴 Blocking Observations (Critical)

#### B1. [Descriptive Title]
- **Affected Targets / Scope:** `...`
- **Technical Observation:** [Detailed problem description]
- **Justification & Risk:** [Datasheet citation, calculation, or standard demonstrating hazard]
- **Recommended Solution:** [Concrete action: part number, value, package, LCSC code]

---

### 🟠 Important Observations (Robustness & EMC)

#### I1. [Recommendation Title]
- **Affected Targets / Scope:** `...`
- **Technical Observation:** [Description]
- **Justification:** [Impact on EMC immunity, RF performance, or signal integrity]
- **Recommended Solution:** [Recommended component, layout change, or filter]

---

### 🟢 Minor Observations (Optimization & DFM)

#### M1. [Optimization Title]
- **Affected Targets / Scope:** `...`
- **Observation & Proposal:** [E.g. Basic Part migration, silkscreen labeling]
- **Expected Benefit:** [Cost reduction, visual clarity, test accessibility]
```
