# What If : Un pipeline « Prompt → Gerber » 100 % Headless (sans EasyEDA MCP)

> [!NOTE]
> Document exploratoire. Ce scénario alternatif n'est pas actif dans le projet actuel et reste volontairement isolé ici pour étudier une architecture de conception électronique entièrement automatisée basée sur des outils CLI externes (KiCad CLI, `pcbnew` headless, FreeRouting, bibliothèques Python) et les skills déjà présents dans `.agents/skills/`.

---

## 1. Constat et changement de paradigme

Dans le workflow actuel d'OpenHVAC-ESP32, la chaîne de conception repose sur **EasyEDA Pro** piloté via un bridge WebSocket local (`http://localhost:49620` dans `easyeda-api`). Ce modèle fonctionne bien en co-pilote assisté par un humain, mais il impose trois freins majeurs à une automatisation complète « du prompt au Gerber » :

1. **Dépendance à une GUI ouverte** : impossible d'exécuter le pipeline en CI/CD, dans un conteneur ou en mode purement headless.
2. **Saisie schématique manuelle** : l'agent ne peut pas instancier de manière fiable un schéma complet depuis zéro sans intervention humaine.
3. **Format propriétaire** : les fichiers `.epro2` ne se prêtent pas au diff Git granulaire ni à la manipulation directe hors de l'éditeur.

### L'idée cible : approche « Netlist-First » & KiCad comme noyau headless

Techniquement, **un schéma graphique n'est pas nécessaire pour produire des Gerbers**. Un PCB n'est qu'une **liste d'empreintes (footprints) + une netlist (connectivité) + des contraintes géométriques et électriques**.

En adoptant une approche **Netlist-First** :
- La source de vérité devient un ensemble de fichiers ouverts générés par code/LLM : `semantics.json` + `bom.json` + `netlist.json` + `constraints.json`.
- La représentation graphique (`.kicad_sch` ou diagramme SVG/Mermaid) devient un **artefact optionnel de revue humaine**, et non un prérequis bloquant.
- **KiCad** est utilisé comme un **noyau de calcul, de vérification et d'export headless** (`kicad-cli` + API Python `pcbnew`), sans fork de son code C++.
- **La majorité de l'intelligence algorithmique** déjà codée dans `.agents/skills/` (placement EMC par recuit simulé, métrologie de boucle, stitching, optimisation BOM JLCPCB, DFM, boîtier 3D) est conservée et réutilisée.

---

## 2. Architecture cible : du Prompt au Gerber (Netlist-First)

Légende visuelle des briques :
- **Vert** : Skill déjà existant dans le dépôt (réutilisable tel quel)
- **Bleu** : Outil externe standard (`kicad-cli`, `pcbnew` headless, `FreeRouting CLI`, `easyeda2kicad`)
- **Orange** : Brique custom à développer (`netlist-gen`, `lcsc-lib`, `kicad-bridge`)
- **Violet** : Hybride (cœur algorithmique existant + adaptateur `kicad-bridge`)

```mermaid
flowchart TD
    classDef existant fill:#d4edda,stroke:#28a745,color:#155724
    classDef externe fill:#cce5ff,stroke:#004085,color:#004085
    classDef custom fill:#ffe5cc,stroke:#d9534f,color:#7a2e00
    classDef hybride fill:#e2d9f3,stroke:#6f42c1,color:#3b1e70

    IN_P["Entree : Prompt utilisateur\n(Cahier des charges, dimensions, tensions, interfaces)"]

    subgraph P1["Phase 1 : Logique, Composants et Validation Electrique"]
        E1["1. Specification et Calculs analogiques\n[Existant] ratio-solver + buck-compensation\nSorties : specs.json"]:::existant
        E2["2. Selection composants et BOM JLCPCB\n[Existant] stingy-schematics (API JLCPCB)\nSorties : bom.json, semantics.json"]:::existant
        E3["3. Netlist et ERC programmatique\n[Custom + Existant] netlist-gen (SKiDL) + review\nSorties : netlist.json, erc_report.json"]:::custom
        E4["4. Empreintes et modeles 3D\n[Custom + Externe] lcsc-lib (kicad-footprints + easyeda2kicad)\nSorties : *.kicad_mod, *.step"]:::custom
        E1 --> E2 --> E3 --> E4
    end

    subgraph P2["Phase 2 : Placement, Routage et Boucle EMC"]
        E5["5. Initialisation PCB vierge\n[Hybride] size_estimator + kicad-bridge (kiutils)\nSorties : board_init.kicad_pcb"]:::hybride
        E6["6. Placement stochastique EMC\n[Hybride] pcb-placer + kicad-bridge (kiutils)\nSorties : board_placed.kicad_pcb, placement.svg"]:::hybride
        E7["7. Export Specctra et Injection contraintes\n[Hybride] kicad-bridge (pcbnew) + dsn_patcher\nSorties : board_raw.dsn -> board_patched.dsn"]:::hybride
        E8["8. Autoroutage headless\n[Existant + Externe] freerouting (FreeRouting CLI)\nSorties : board_routed.ses"]:::externe
        E9["9. Import SES et Metrologie EMC\n[Hybride] kicad-bridge (pcbnew) + dsn_metrics\nSorties : board_routed.kicad_pcb, emc_metrics.json"]:::hybride
        E5 --> E6 --> E7 --> E8 --> E9
        E9 -->|"Non conforme (cycle < 3)"| E6
    end

    subgraph P3["Phase 3 : Cuivre, Certification et Fabrication"]
        E10["10. Plans de masse et Stitching vias\n[Hybride] pcb-stitcher + kicad-bridge (pcbnew ZONE_FILLER)\nSorties : board_stitched.kicad_pcb"]:::hybride
        E11["11. Serigraphie sans collision\n[Hybride] pcb-silkscreen + kicad-bridge (kiutils)\nSorties : board_final.kicad_pcb"]:::hybride
        E12["12. Certification DRC et Audit DFM\n[Externe + Hybride] kicad-cli pcb drc + pcb-dfm\nSorties : drc_report.json, dfm_report.md"]:::externe
        E13["13. Export Fabrication\n[Externe] kicad-cli pcb export\nSorties : gerbers/*.gbr, drill/*.drl, cpl.csv, render.png"]:::externe
        E10 --> E11 --> E12
        E12 -->|"DRC = 0"| E13
    end

    IN_P --> E1
    E4 --> E5
    E9 -->|"Conforme"| E10
```

### Matrice des étapes : Implémentation, Livrables et Critères de validation

| # | Étape | En face : KiCad CLI vs Existant vs Custom | Entrées | Livrables (Sorties) | Critère de passage (Gate) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Spécification & calculs** | **Existant** (`ratio-solver`, `buck-compensation`) + LLM | Prompt utilisateur | `specs.json` (tensions, courants, valeurs R/C/L, marges) | Marge de phase > 45°, tolérances E24/E96 validées |
| **2** | **Choix composants JLCPCB** | **Existant** (`stingy-schematics` + API JLCPCB) | `specs.json` | `bom.json` (LCSC `Cxxxxx`), `semantics.json` | 100 % des pièces en stock, priorité *Basic Parts* |
| **3** | **Netlist & ERC** | **Custom** (`netlist-gen` via SKiDL/DSL) + **Existant** (`review`) | `semantics.json`, `bom.json` | `netlist.json`, `erc_report.json`, résumé lisible | **ERC = 0** (aucune broche flottante/conflit), validation humaine optionnelle |
| **4** | **Empreintes & 3D** | **Custom** (`lcsc-lib`) wrappant **Externe** (`kicad-footprints`, `easyeda2kicad`) | `bom.json` | `libraries/*.kicad_mod`, `*.step` | 100 % des composants ont un `.kicad_mod` et pin-mapping vérifié |
| **5** | **Init PCB vierge** | **Existant** (`size_estimator.py`) + **Custom** (`kicad-bridge` via `kiutils`) | `netlist.json`, `constraints.json`, `*.kicad_mod` | `board_init.kicad_pcb` (`Edge.Cuts`, trous M2/M3, empreintes) | Surface estimée valide, toutes empreintes instanciées |
| **6** | **Placement EMC** | **Existant** (`pcb-placer`) + **Custom** (`kicad-bridge` via `kiutils`) | `board_init.kicad_pcb`, `constraints.json`, `manifest.json` | `board_placed.kicad_pcb`, `placement.svg` | 0 collision courtyard, keepouts RF respectés, découplage < 2 mm |
| **7** | **Export & Patch DSN** | **Externe** (`pcbnew.ExportSpecctraDSN`) + **Existant** (`dsn_patcher.py`) | `board_placed.kicad_pcb`, `constraints.json` | `board_raw.dsn`, `board_patched.dsn` | Toutes les netclasses, largeurs et clearances injectées |
| **8** | **Autoroutage** | **Existant** (`router.py`) + **Externe** (`FreeRouting CLI` + JRE 25) | `board_patched.dsn` | `board_routed.ses` | 100 % des nets routés (hors `plane_nets` comme `GND`) |
| **9** | **Import & Métrologie** | **Externe** (`pcbnew.ImportSpecctraSES`) + **Existant** (`dsn_metrics.py`) | `board_placed.kicad_pcb`, `board_routed.ses`, `board_patched.dsn` | `board_routed.kicad_pcb`, `emc_metrics.json` | Aire de boucle / inductance / skew dans les seuils (sinon feedback → étape 6) |
| **10** | **Stitching & Plans GND** | **Existant** (`pcb-stitcher`) + **Externe** (`pcbnew.ZONE_FILLER`) | `board_routed.kicad_pcb`, `constraints.json` | `board_stitched.kicad_pcb` | 0 îlot GND isolé, zones remplies sans erreur |
| **11** | **Sérigraphie** | **Existant** (`pcb-silkscreen`) + **Custom** (`kicad-bridge` via `kiutils`) | `board_stitched.kicad_pcb`, `silkscreen.json` | `board_final.kicad_pcb` | 0 chevauchement texte / pastille / via |
| **12** | **DRC & Audit DFM** | **Externe** (`kicad-cli pcb drc`) + **Existant** (`pcb-dfm`) | `board_final.kicad_pcb`, `bom.json` | `drc_report.json`, `dfm_report.md` | **DRC = 0** (code retour 0), règles fabricant JLCPCB respectées |
| **13** | **Export Gerbers** | **Externe** (`kicad-cli pcb export` : `gerbers`, `drill`, `pos`, `render`) | `board_final.kicad_pcb` | `gerbers/*.gbr`, `drill/*.drl`, `cpl.csv`, `render.png` | Archive Gerber complète + inspection visuelle `render.png` OK |

---

## 3. Remplacer l'ERC schématique en mode Netlist-First

Si l'on supprime l'étape `.kicad_sch` obligatoire, on n'exécute plus `kicad-cli sch erc`. Il faut donc garantir la sûreté électrique **avant** le placement par deux garde-fous :

1. **ERC programmatique sur la netlist** :
   - En utilisant **SKiDL** (ou un vérificateur dédié dans `review`), chaque broche porte son type électrique (`POWER_OUT`, `POWER_IN`, `INPUT`, `OUTPUT`, `BIDIR`, `NC`).
   - Le moteur vérifie automatiquement l'absence de court-circuit entre sorties, l'absence d'entrées flottantes, la présence d'alimentation sur chaque circuit intégré et la cohérence des niveaux logiques.
2. **Revue humaine synthétique avant routage** :
   - Plutôt qu'une netlist brute illisible, l'outil génère un résumé par bloc fonctionnel (ex. *Alimentation Buck*, *MCU*, *Bus*) ou un diagramme SVG/Mermaid auto-généré pour valider l'intention électrique en quelques secondes.

---

## 4. Réutilisation des skills actuels du dépôt

Un audit des scripts sous `.agents/skills/` montre que la séparation entre **moteur algorithmique** et **connecteur EasyEDA** est déjà très nette :

| Skill actuel | Couplage EasyEDA actuel | Effort d'adaptation vers KiCad Headless | Rôle dans le nouveau pipeline |
| :--- | :--- | :--- | :--- |
| `ratio-solver` | Aucun (100 % pur Python) | **0 %** — Réutilisable tel quel | Calcul des ponts diviseurs, feedback LDO/Buck, atténuateurs ADC. |
| `buck-compensation` | Aucun (100 % pur Python) | **0 %** — Réutilisable tel quel | Calcul de boucle Type II et stabilité (marge de phase/gain). |
| `stingy-schematics` | Aucun (API JLCPCB + JSON) | **0 %** — Réutilisable tel quel | Sélection de composants JLCPCB Basic Parts, économie de frais de setup. |
| `review` | Aucun (JSON + Markdown) | **Faible** — Étendre `schematic_auditor.py` à l'ERC de `netlist.json` | Audit électrique/BOM et triage des non-conformités. |
| `pcb-enclosure` | Aucun (`constraints.json`) | **0 %** — Réutilisable tel quel | Génération du boîtier 3D STEP/STL aligné sur les perçages et connecteurs. |
| `pcb-placer` | Moyen (`easyeda_client.py`, `export_geometry.py`, `apply_placement.py`) | **Faible** — Remplacer le client HTTP par un adaptateur `kiutils` / `pcbnew` | Placement stochastique, physique EMC ($1/r^3$, $1/r^2$), keepouts RF, ancrages. |
| `freerouting` | Faible (`dsn_exporter.py`, `ses_importer.py`, `drc_checker.py`) | **Faible** — `jre_manager.py`, `dsn_patcher.py`, `router.py` sont déjà agnostiques | Routage automatique headless via FreeRouting v2.4.1. |
| `pcb-synthesizer` | Faible (`synthesizer_pipeline.py` ; `dsn_metrics.py` est pur Python) | **Faible** — Brancher l'orchestrateur sur l'adaptateur KiCad | Boucle fermée Placement → Routage → Métrologie → Raffinement. |
| `pcb-stitcher` | Moyen (`geometry.py` pur Python ; injection via bridge) | **Moyen** — Écrire les vias/pistes directement dans `.kicad_pcb` | Fanout GND et pontage d'îlots de cuivre par A* 3D. |
| `pcb-silkscreen` | Moyen (`silkscreen_resolver.py` pur Python ; injection via bridge) | **Moyen** — Écrire les textes `F.SilkS`/`B.SilkS` dans `.kicad_pcb` | Placement lisible des marquages sans chevauchement de pastilles. |
| `pcb-dfm` | Moyen (`drc_bridge.py`, `export_fabrication.py`, `export_pcb_images.py`) | **Faible** — Remplacer par `kicad-cli` (`drc`, `gerbers`, `drill`, `pos`, `svg`) | Audit DFM/SMT et génération du dossier de fabrication JLCPCB. |
| `easyeda-api` | 100 % EasyEDA | **À supprimer / archiver** | Remplacé par `kicad-bridge`. |

---

## 5. KiCad comme noyau open source : CLI, `pcbnew` et limites du fork C++

KiCad est 100 % open source (code C++ sous licence **GPL v3+**, bibliothèques `kicad-symbols` et `kicad-footprints` sous **CC-BY-SA 4.0**).

### Pourquoi NE PAS forker le code C++ de KiCad
- **Maintenance lourde** : base C++ massive (wxWidgets, OpenCascade, Boost, ngspice) coûteuse à compiler et à maintenir à jour.
- **Contagion GPL v3** : lier directement le code C++ de KiCad dans un outil tiers soumet cet outil à la GPL v3, alors qu'appeler `kicad-cli` ou l'IPC API en sous-processus garde l'orchestrateur indépendant.
- **Inutile dans 99 % des cas** : tout ce dont on a besoin est déjà accessible sans toucher au C++.

### Ce que `kicad-cli` fait nativement vs ce que `pcbnew` / `kiutils` complètent

| Capacité | Outil utilisé | Détail |
| :--- | :--- | :--- |
| **DRC final (`DRC = 0`)** | `kicad-cli pcb drc --exit-code-violations --format json` | Audit complet avec rapport JSON parsable par l'agent. |
| **Gerbers, Drill, CPL** | `kicad-cli pcb export gerbers` / `drill` / `pos` | Génération directe des fichiers de fabrication JLCPCB. |
| **Rendu visuel & 3D** | `kicad-cli pcb render` / `export svg` / `export step` | Inspection visuelle par l'agent (PNG top/bottom) et export mécanique 3D. |
| **Manipulation `.kicad_pcb`** | **`kiutils`** (pur Python sur S-expressions) | Lecture des courtyards, écriture des positions/rotations, vias, sérigraphie, `Edge.Cuts`. |
| **DSN/SES & Zone Fill** | **`pcbnew`** (API Python embarquée KiCad, headless) | `ExportSpecctraDSN`, `ImportSpecctraSES`, `ZONE_FILLER` (absent de `kicad-cli`). |
| **Empreintes normalisées** | Dépôt officiel **`kicad-footprints`** + `easyeda2kicad` | Des milliers d'empreintes IPC-7351 fiables, complétées par LCSC au besoin. |

> [!TIP]
> **Le seul cas où un build KiCad personnalisé a du sens** : pour packager l'outil dans un conteneur **Docker / CI léger**, on peut compiler uniquement la partie headless de KiCad (`kicad-cli` + bindings `_pcbnew`) sans l'interface graphique complète.

---

## 6. Les 3 nouveaux skills à concevoir

1. **`netlist-gen` (ex-`sch-gen`)** :
   - Traduit l'architecture issue du prompt (`semantics.json`) en `netlist.json` + `bom.json` via **SKiDL** ou un DSL Python dédié, avec ERC programmatique intégré.
   - Optionnel : génère un diagramme fonctionnel SVG/Mermaid (ou un `.kicad_sch` par blocs + net labels) pour la revue humaine.
2. **`lcsc-lib` (Pont composants JLCPCB → KiCad)** :
   - Résout chaque composant de `bom.json` en priorité depuis **`kicad-footprints`** (empreintes officielles), et utilise `easyeda2kicad` / `JLC2KiCadLib` pour les références LCSC spécifiques (`Cxxxxx`), avec vérification de l'orientation 0° JLCPCB.
3. **`kicad-bridge` (Adaptateur CLI & `pcbnew` headless)** :
   - Remplace `easyeda-api` en exposant une interface unifiée aux skills existants : `init_pcb_from_netlist`, `export_board_geometry`, `apply_placement`, `export_dsn`, `import_ses`, `fill_zones`, `run_drc`, `export_manufacturing`.

---

## 7. Pistes ouvertes à creuser

1. **Format d'entrée de `netlist-gen`** : Utiliser **SKiDL** (existant, Python pur, ERC intégré) ou **Atopile**, ou un générateur maison sur `semantics.json` ?
2. **Architecture du dépôt** : Introduire une abstraction `BoardBackend` dans ce dépôt pour supporter EasyEDA **et** KiCad, ou créer un outil/dépôt dédié uniquement au pipeline KiCad headless ?
3. **Contraintes critiques avant FreeRouting** : Étendre `dsn_patcher.py` pour protéger (`type protect`) les pré-routages critiques (boucles Buck, paires différentielles USB/Ethernet) avant le passage de l'autorouteur.
4. **Contour mécanique automatique** : Exploiter `size_estimator.py` (`pcb-placer`) pour générer automatiquement `Edge.Cuts` et les trous de montage M2/M3 dans `.kicad_pcb` depuis `constraints.json`.
