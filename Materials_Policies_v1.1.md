# Materials_Policies_v1.1

## Objectif
Normaliser les propriétés globales d’un matériau PCB en JSON `properties` → 4 familles, chacune avec `metrics[]` (objet **Metric** générique et extensible).

## Périmètre
- **IN**: Valeurs globales (Tg, Td, CTE, résistivités, peel strength, etc.), résumés Dk/Df ponctuels (ex: Dk@1 GHz typ).
- **OUT**: Détails Dk/Df vs fréquence par *stack-up* (aller dans `forms`, pas ici).

## Schéma de référence (rappel)
Voir `metric_schema_v1.1.json`. Les clés JSON sont **anglais snake_case**. Les unités ne vont **jamais** dans les ids.

## Règles d’extraction
1. **Ne pas inventer**: si absente → omettre.
2. **Unités**: conserver celles du PDF ou convertir en SI si clair (ex: kV/mm ↔ V/µm). Si conversion: ≤ 4 décimales.
3. **Seuils**: capturer `≥`, `≤`, `Pass`, classes CTI → `qualifier` = "≥" / "Pass" / "Class …".
4. **Conditions**:
   - `frequency_Hz` (numérique, Hz), `temperature_C`, `resin_content_percent`.
   - `axis` = x|y|z ; `phase` = pre_Tg|post_Tg ; `copper_state` = with_cu|copper_removed ; `conditioning` = after_thermal_stress|after_moisture_resistance|…
   - `weight_loss_percent` pour Td (ex: 5).
5. **Multi‑valeurs**: MD/TD, length/cross, pre/post → `values[]` avec `label`.
6. **Méthodes & normes**: remplir `method`/`standard` si visibles (ex: IPC‑TM‑650 2.5.5.9, ASTM D149, UL 94).
7. **PDF “collés”**: re-split de nombres (ex: `0.0513.89`), supprimer espaces parasites, respecter signes.
8. **Arrondis**: conserver précision raisonnable (≤ 4 décimales) sans troncature agressive.

## Mappings usuels (non exhaustif)
- **Thermal**
  - Tg (DSC/TMA/DMA) → `glass_transition_temperature` (°C), `statistic` selon la fiche, `method` si dispo.
  - Td (5% WL, TGA) → `thermal_decomposition_temperature` (°C), `conditions.weight_loss_percent=5`.
  - CTE Z → `z_axis_cte` (ppm/°C) avec labels `pre_Tg`/`post_Tg`.
  - Conductivité thermique → `thermal_conductivity` (W/m·K).
  - Solder float/thermal stress → `solder_float_resistance`/`thermal_stress` avec `qualifier` "Pass".
- **Electrical**
  - Dk/Df résumé explicite → `dielectric_constant` / `dissipation_factor` + `conditions.frequency_Hz`.
  - Electric strength → `electric_strength` (kV/mm) ; Dielectric breakdown → `dielectric_breakdown` (kV).
  - Volume/surface resistivity → `volume_resistivity` (Ω·cm) / `surface_resistivity` (Ω).
  - CTI → `comparative_tracking_index` (texte/grade).
- **Physical**
  - Moisture absorption → `moisture_absorption` (%).
  - Dimensional stability (MD/TD) → `dimensional_stability` (%) avec `label`.
  - Flammability (UL) → `flammability_rating` (ex: "UL‑94 V0" ou "UL‑94 VTM‑0").
- **Mechanical**
  - Peel strength (N/mm, kgf/cm) → `peel_strength` + labels: `as_received`, `after_solder_float`, `after_thermal_stress`, `after_process_solutions`.
  - Flexural endurance (MIT) → `flexural_endurance` (cycles) + `conditions` (rayon, charge).
  - Tensile strength/modulus/elongation → `tensile_strength` (MPa ou kgf/mm²), `tensile_modulus`, `elongation` (%), `specimen="base_film"` si précisé.

## Bonnes pratiques
- Grouper logiquement les metrics (éviter doublons).
- Garder la **cohérence d’unités** à l’intérieur d’une même metric.
- `series` uniquement si la fiche publie plusieurs points de la même courbe EN PROPRIÉTÉS (cas rare). Sinon, ne pas utiliser `series`.

## Erreurs structurées
- Document vide ou tableau absent → `{ "error":"No extractable properties", "missing":["Properties table"] }`
- Valeurs illisibles → les omettre, ne jamais interpoler en "properties".
``