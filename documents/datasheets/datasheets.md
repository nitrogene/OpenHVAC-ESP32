# Datasheets & Technical References

This index catalogs all component datasheets archived under [`references/`](references/) and key electrical specifications for **OpenHVAC-ESP32**.

| Component / Function | Part Number | Package | LCSC | JLCPCB Tier | Datasheet | Key Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **MCU (Wi-Fi 6 + Zigbee 3.0)** | `ESP32-C6-WROOM-1-N4` | `18 × 25.5 mm` SMD | `C6034714` | Extended | [`esp32-c6-wroom-1_datasheet_en.pdf`](references/esp32-c6-wroom-1_datasheet_en.pdf) | `3.3V` (`3.0–3.6V`), `4 MB` Flash, native USB `GPIO12/13`, UART0 `GPIO16/17` |
| **Buck `12V -> 3.3V` (Synchronous)** | `AP63203WU-7` | `TSOT-23-6` | `C780769` | Extended | [`AP63203_AP63200_datasheet.pdf`](references/AP63203_AP63200_datasheet.pdf) | `3.8V–32V` in, fixed `3.3V` `2A` out, `1.1 MHz`, internal compensation |
| **LDO `5V` (UART Level Shifter)** | `AMS1117-5.0` | `SOT-223` | `C6187` | **Basic** | [`AMS1117_datasheet.pdf`](references/AMS1117_datasheet.pdf) | `12V -> 5V` (`max 15V` in), `< 5 mA` load for HVAC-side `74LVC1T45` |
| **UART Level Shifter (`2×`)** | `74LVC1T45W6-7` | `SOT-23-6` | `C168856` | Extended | [`74LVC1T45_datasheet.pdf`](references/74LVC1T45_datasheet.pdf) | Unidirectional 1-bit (`TX` `A->B`, `RX` `B->A`), `Ioff` isolation |
| **Reverse Polarity P-MOSFET** | `AO3401A` | `SOT-23` | `C15127` | **Basic** | [`AO3401A_datasheet.pdf`](references/AO3401A_datasheet.pdf) | `-30V`, `-4A` P-MOSFET for `12V` input reverse-polarity protection |
| **USB-C ESD Protection** | `USBLC6-2SC6` | `SOT-23-6` | `C7519` | Extended | ST `usblc6-2` (pending PDF) | `5V` rail + `D+`/`D-` ESD array (`17 pF`) |
| **Input TVS (`15V` / `18V`)** | `SMAJ15A` / `SMF15A` | `SMA` / `SOD-123FL` | `C19077535` / `C19077509` | Preferred | — | Clamps `12V` HVAC rail transients |
| **Inductor (`3.3V` Buck)** | `SWPA4030S4R7MT` (eq.) | `4.0 × 4.0 mm` | `C48601350` / `C38117` | Extended | — | `4.7 µH` shielded, `>= 2.3A` |
| **OR-ing Schottky (`VBUS` 5V)** | `1N5819WS` | `SOD-323` | `C191023` | **Basic** | — | Allows USB-C `5V` to power Buck `3.3V` during flashing |
| **HVAC Connector** | `S04B-XASK-1(LF)(SN)` | `JST XA 2.5mm` 4P | `C265097` | Through-Hole | — | Pin 1 `GND`, Pin 2 `PRX`, Pin 3 `PTX`, Pin 4 `+12V` |
| **USB-C Receptacle** | `TYPE-C 16PIN 2MD(073)` | `USB-C SMD` | `C2765186` | Extended | — | Native USB-Serial/JTAG (`5.1 kΩ` CC1/CC2, `22 Ω` D+/D-) |
| **BOOT Switch** | `TS-1088-AR02016` | `SW-SMD 3.9×3.0` | `C720477` | Extended | — | Pulls `GPIO9` to `GND` for bootloader mode |
| **RST Switch** | `TS24CA` | `SW-SMD 2P` | `C393942` | Extended | — | Pulls `EN` (`CHIP_PU`) to `GND` |
| **Power LED (`PWR`)** | `KT-0805G` (Green) | `LED 0805` | `C2297` | **Basic** | — | `+5V` power indicator (`560 Ω` `C23204` → `~5.3 mA`, satisfies `AMS1117` min load) |
| **Status LED (`STA`)** | `KT-0603R` (Red) | `LED 0603` | `C2286` | **Basic** | — | Status LED on `GPIO3` (`2.2 kΩ` `C4190`) |
| **Bulk / Output Caps (`22 µF`)** | `CL31A226KAHNNNE` / `CL21A226MAQNNNE` | `1206` / `0805` | `C12891` / `C45783` | **Basic** | — | `25V X5R` (`2× 1206` on `12V`, `0805` on `3.3V`/`5V`/MCU) |
| **Input / LDO Caps (`10 µF`)** | `CL31A106KBHNNNE` | `1206` | `C13585` | **Basic** | — | `50V X5R` (`VIN_BUCK` & `AMS1117` input) |
| **EN RC & HF Decoupling** | `CL10A105KB8NNNC` / `CL05B104KO5NNNC` / `CL05B104KB54PNC` | `0603` / `0402` | `C15849` / `C1525` / `C307331` | **Basic** | — | `1 µF` (`50V`, `EN` RC), `100 nF` (`16V` decoupl., `50V` `BST-SW`) |
| **Passives (Resistors)** | `0603WAF...` | `0603` | `C25803`, `C25804`, `C23186`, `C23345`, `C23204`, `C4190` | **Basic** | — | `100 kΩ` (`Q1` `Vgs=-6V`), `10 kΩ`, `5.1 kΩ`, `22 Ω`, `560 Ω`, `2.2 kΩ` |
