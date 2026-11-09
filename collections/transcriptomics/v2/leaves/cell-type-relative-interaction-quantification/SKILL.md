---
name: cell-type-relative-interaction-quantification
description: Use when several pre-computed cell-cell communication (CCI) datasets
  from different conditions must be compared cell-type-wise, to test whether a cell
  type (e.g., mast cells or NKT-p) consistently mediates more interactions in one
  condition.
when_to_use_negative:
- Only a single condition/dataset is available — the multiple-condition comparison
  and radar overlays require at least two CCI datasets to contrast
- The goal is de novo inference of ligand-receptor interactions from a scRNA-seq expression
  matrix — InterCellar builds on pre-computed, inferred ligand-receptor interactions,
  not raw counts
- A tool outside InterCellar's supported set was used to compute the CCI data — input
  options cover CellChat, CellPhoneDB, ICELLNET and SingleCellSignalR
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: Shiny app that loads pre-computed CCI datasets, computes per-cell-type relative
    interaction numbers in its multiple-condition module, and renders overlaid radar
    plots
  repo: https://github.com/martaint/InterCellar-reproducibility
- name: InterCellar-reproducibility
  role: Provides the pre-computed COVID-19 (control, moderate, critical) CCI datasets
    and walkthrough used to reproduce the radar plots (Fig. 4b)
  repo: https://github.com/martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:4d8c3c925e3b0e3471fe64e6ddca7a565b633d7d5eb67939fb85bf7a82ee7f41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar-reproducibility
  tools:
  - InterCellar
  - InterCellar-reproducibility
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cell-type-relative-interaction-quantification

## Summary

Quantify, for each cell type within a condition, the relative number of predicted ligand-receptor interactions with all other clusters, and overlay profiles across multiple conditions (e.g., control, moderate, and critical COVID-19) using InterCellar's multiple-condition radar plots to reveal condition-specific communication patterns.

## When to use

Use when several pre-computed cell-cell communication (CCI) datasets from different conditions must be compared cell-type-wise, to test whether a cell type (e.g., mast cells or NKT-p) consistently mediates more interactions in one condition.

## When NOT to use

- Only a single condition/dataset is available — the multiple-condition comparison and radar overlays require at least two CCI datasets to contrast
- The goal is de novo inference of ligand-receptor interactions from a scRNA-seq expression matrix — InterCellar builds on pre-computed, inferred ligand-receptor interactions, not raw counts
- A tool outside InterCellar's supported set was used to compute the CCI data — input options cover CellChat, CellPhoneDB, ICELLNET and SingleCellSignalR

## Inputs

- One or more pre-computed CCI datasets (inferred ligand-receptor interactions) from InterCellar-supported tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR), one per condition (e.g., control, moderate, critical COVID-19)

## Outputs

- Per-cell-type relative interaction-number profiles within each condition
- Overlaid radar plots comparing condition-specific communication profiles per cell type
- Condition-specific pattern identification (e.g., cell types with consistently elevated interactions with MC or NKT-p in critical cases)

## How to apply

Load each condition's pre-computed inferred ligand-receptor interactions as a CCI dataset into the InterCellar Shiny app (launched with reproducible=TRUE for reproducible sessions). In InterCellar's multiple-condition comparison module, compute for every cell type the relative number of interactions with all other clusters within each condition rather than absolute counts, so datasets with different interaction totals are comparable. Render radar plots that overlay the per-condition profiles (e.g., control vs. moderate vs. critical COVID-19) for each cell type so the communication 'shapes' are directly comparable. Inspect whether any cell type shows consistently elevated relative interaction numbers with a specific partner across cell types in one condition; in the COVID-19 application this revealed higher communication of both immune cells (MoMa, nrMa, CTL, Treg) and epithelial cells (ciliated, secretory) with mast cells (MC) or proliferating NKT cells (NKT-p) in critical cases. Use InterCellar's download options to export the tables and figures for reproducibility.

## Related tools

- **InterCellar** (Shiny app that loads pre-computed CCI datasets, computes per-cell-type relative interaction numbers in its multiple-condition module, and renders overlaid radar plots) — https://github.com/martaint/InterCellar-reproducibility
- **InterCellar-reproducibility** (Provides the pre-computed COVID-19 (control, moderate, critical) CCI datasets and walkthrough used to reproduce the radar plots (Fig. 4b)) — https://github.com/martaint/InterCellar-reproducibility

## Examples

```
library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- Each condition's CCI dataset loads in InterCellar without errors and the condition labels (control/moderate/critical) are correctly assigned in the multiple-condition module
- Relative interaction numbers are normalized within each condition, so per-condition totals do not bias cross-condition comparisons
- The rendered radar plot for a given cell type shows distinct overlaid polygons (shapes) for each condition, all anchored on the same set of partner cell types
- The critical-COVID-19 radar profiles reproduce the reported pattern: elevated relative interaction numbers toward MC (mast cells) and NKT-p for both immune (MoMa, nrMa, CTL, Treg) and epithelial (ciliated, secretory) cell types
- Tables and figures are exported via InterCellar's download options so results are reproducible across sessions

## Limitations

- InterCellar supports only a limited set of input CCI tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence), so relative interaction numbers are inference-based signals
- Prediction methods rely on different reference databases and statistical assumptions, so relative interaction profiles may differ depending on the CCI tool used upstream

## Evidence

- [abstract] InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [intro] functionalities to compare interactions across multiple datasets: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [discussion] critical COVID-19 mast cell pattern: "the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells"
- [readme] reproducible app launch: "The flag `reproducible = TRUE` ensures that your results will be reproducible across R sessions."
