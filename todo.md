# Project Roadmap (TODO)

> **Active Constitution:** [`.specify/memory/constitution.md`](.specify/memory/constitution.md)  
> **Workflow:** `hw-specify` → `hw-plan` → `hw-implement` (each run lives in `specs/<id>-<slug>/` with its own `checkpoint.md` and `tasks.md`).

---

## Part 1 — Mise au point & Validation des Skills `hw-*` (KiCad Headless)

Objectif immédiat : valider la chaîne **`Prompt → spec.md → netlist.json → Gerber (+ 3D)`** avant de lancer la fabrication finale.

### 1.1 Prérequis Techniques Locaux
- [ ] **Environnement KiCad Headless :** Vérifier `kicad-cli` dans le `PATH` et la disponibilité de l'API Python `pcbnew` sous Windows.
- [ ] **Dépendances Python (`uv`) :** Vérifier/installer `kiutils`, `easyeda2kicad` (ou `JLC2KiCadLib`), `jsonschema` et FreeRouting CLI (+ JRE).
- [ ] **Rapatriement / Lien des skills existants :** Installer ou pointer les skills techniques (`ratio-solver`, `buck-compensation`, `stingy-schematics`, `pcb-placer`, `freerouting`, `pcb-synthesizer`, `pcb-stitcher`, `pcb-silkscreen`, `pcb-dfm`, `pcb-enclosure`, `review`) dans `.agents/skills/` (ou en global).

### 1.2 Validation de `hw-specify`
- [ ] **Contrôle de la Constitution :** Tester que `hw-specify` détecte d'éventuels placeholders dans `constitution.md` et valide `.specify/memory/constitution.md` sans erreur.
- [ ] **Génération `specs/001-openhvac-airton/` :** Lancer `hw-specify` sur le besoin Airton pour produire `spec.md` et `checkpoint.md` (avec questions de clarification pertinentes).

### 1.3 Validation de `hw-plan`
- [ ] **Calculs & BOM :** Vérifier le chaînage `ratio-solver` + `buck-compensation` + `stingy-schematics` → `specs.json`, `bom.json`, `semantics.json`.
- [ ] **Netlist & Contraintes :** Produire `netlist.json` (sans nets `$1N...`) et `constraints.json` (enveloppe Forhorse v2 `54.10 × 42.29 mm`).
- [ ] **ERC Programmatique :** Mettre en place et valider le contrôle `ERC = 0` (`reports/erc_report.json`) + diagramme lisible de revue + génération de `specs/001-openhvac-airton/tasks.md`.

### 1.4 Validation de `hw-implement`
- [ ] **Brique `lcsc-lib` :** Résoudre 100 % des empreintes `.kicad_mod` et modèles `.step` depuis `kicad-footprints` / LCSC + vérification orientation CPL 0° JLCPCB.
- [ ] **Brique `kicad-bridge` :** Valider `init_pcb`, `apply_placement`, `export_dsn`, `import_ses`, `fill_zones`, `run_drc` (`kicad-cli`) et `export_manufacturing`.
- [ ] **Test de bout en bout (`DRC = 0`) :** Sortir une première archive `production/` complète (Gerbers, Drill, CPL, BOM, `render_top.png`/`render_bot.png`) sur `specs/001-openhvac-airton/`.
- [ ] **Test de reprise multi-sessions :** Interrompre à mi-parcours, commiter, ouvrir une nouvelle session et vérifier la reprise propre depuis `checkpoint.md`.

### 1.5 Script d'Installation Multi-Environnements *(optionnel après validation)*
- [ ] **Installateur Python (`install_hw.py`) :** Permettre `uv run install_hw.py <dossier> --ai agy|copilot|...` déployant `.specify/memory/constitution-template.md`, les pointeurs et les skills adaptés.

---

## Part 2 — Projet OpenHVAC-ESP32 (Airton)

- [ ] **Relevé matériel Airton :** Couper le disjoncteur, inspecter le module Wi-Fi d'origine et confirmer le `12V`, l'UART `5V` et le brochage `JST XA 2.5` 4 broches (`devildant/acw02_esphome`).
- [ ] **Gel de la carte (`specs/001-openhvac-airton/`) :** Valider le diagramme électrique (`ERC = 0`), la BOM LCSC en stock et les rendus `render_*.png` (`DRC = 0`, DFM JLCPCB OK).
- [ ] **Contrôle 3D & Commande :** Vérifier l'absence de collision mécanique dans le boîtier Forhorse v2 et commander les PCBA chez JLCPCB.
- [ ] **Bring-up banc `12V` :** Vérifier le `3.3V` Buck, le `5V` LDO, le flash USB-C et les pics de courant RF TX avant raccordement à la clim.
- [ ] **Firmware & Intégration Airton :** Flasher ESPHome ou Zigbee 3.0, raccorder à l'unité intérieure Airton et valider les échanges UART, la stabilité thermique et l'absence de brownouts.
