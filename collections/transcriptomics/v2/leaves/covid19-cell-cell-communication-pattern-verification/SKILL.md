---
name: covid19-cell-cell-communication-pattern-verification
description: Use when you have pre-computed CCI datasets (e.g., CellChat/CellPhoneDB/ICELLNET/SingleCellSignalR
  output) for multiple conditions (control, moderate, critical COVID-19) and need
  to compare per-cell-type interaction profiles across conditions to verify condition-specific
  communication patterns.
when_to_use_negative:
- You need to re-infer interactions from raw scRNA-seq data — InterCellar builds upon
  pre-computed, inferred ligand-receptor interactions uploaded as CCI data and does
  not perform the inference itself.
- Only a single condition is available — the radar-plot comparison module requires
  multiple conditions/datasets to be meaningful.
- You need a spatial or bulk-RNA-seq cell-cell communication analysis — CCCExplorer-style
  bulk tools and this scRNA-seq workflow do not transfer directly.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: InterCellar
  role: R/Shiny app used to load the three CCI datasets, compute relative interaction
    numbers per cell type per condition, and render the multi-condition radar plots
  repo: https://github.com/martaint/InterCellar
- name: InterCellar-reproducibility
  role: Repository supplying the pre-computed COVID-19 (control, moderate, critical)
    CCI data and walkthrough needed to reproduce Fig. 4b
  repo: https://github.com/martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:b83c1c878c7bb217885870772f6306b107f9c7abc8d450d4f02de857ce75740d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - InterCellar
  - InterCellar-reproducibility
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# covid19-cell-cell-communication-pattern-verification

## Summary

Reproduce InterCellar's multiple-condition radar plots of relative interaction numbers per viewpoint cell type to verify that critical COVID-19 cases show a consistent communication pattern in which all cell types interact more with mast cells (MC) and proliferating NKT (NKT-p) cells than in moderate/control conditions.

## When to use

Use when you have pre-computed CCI datasets (e.g., CellChat/CellPhoneDB/ICELLNET/SingleCellSignalR output) for multiple conditions (control, moderate, critical COVID-19) and need to compare per-cell-type interaction profiles across conditions to verify condition-specific communication patterns.

## When NOT to use

- You need to re-infer interactions from raw scRNA-seq data — InterCellar builds upon pre-computed, inferred ligand-receptor interactions uploaded as CCI data and does not perform the inference itself.
- Only a single condition is available — the radar-plot comparison module requires multiple conditions/datasets to be meaningful.
- You need a spatial or bulk-RNA-seq cell-cell communication analysis — CCCExplorer-style bulk tools and this scRNA-seq workflow do not transfer directly.

## Inputs

- Three pre-computed cell-cell interaction (CCI) datasets for COVID-19 BALF samples: control, moderate, and critical conditions (ligand-receptor interaction tables from InterCellar-supported tools such as CellChat, CellPhoneDB, ICELLNET, or SingleCellSignalR)
- Cell cluster annotations (immune cell types e.g. MoMa, nrMa, CTL, Treg, MC, NKT-p; epithelial cell types e.g. ciliated, secretory)
- InterCellar Shiny app installed via Bioconductor/Bioconda/Docker

## Outputs

- Radar plots per viewpoint cell type overlaying relative interaction-number profiles across control, moderate, and critical conditions
- Verified pattern: consistently higher number of interactions between all cell types and MC or NKT-p cells in critical COVID-19 (Fig. 4b)
- Downloadable tables/figures for reproducibility via InterCellar's download options

## How to apply

Load the three pre-computed CCI datasets (control, moderate, critical COVID-19) from the martaint/InterCellar-reproducibility repository into the InterCellar Shiny app, launched with reproducible = TRUE so results are reproducible across R sessions. In InterCellar's multiple-condition comparison module, compute for each viewpoint cell type the relative number of interactions with all other clusters within each condition. Render radar plots that overlay the control, moderate, and critical profiles for each cell type (immune cells such as MoMa, nrMa, CTL, Treg and epithelial cells such as ciliated and secretory) so the per-condition communication shapes are directly comparable. Inspect the radar profiles and confirm the verification target: in critical cases, all cell types show a consistently higher number of interactions with MC and NKT-p cells, matching the paper's finding that 'the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells'.

## Related tools

- **InterCellar** (R/Shiny app used to load the three CCI datasets, compute relative interaction numbers per cell type per condition, and render the multi-condition radar plots) — https://github.com/martaint/InterCellar
- **InterCellar-reproducibility** (Repository supplying the pre-computed COVID-19 (control, moderate, critical) CCI data and walkthrough needed to reproduce Fig. 4b) — https://github.com/martaint/InterCellar-reproducibility

## Examples

```
library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- Radar plots render for each viewpoint cell type with three overlaid condition profiles (control, moderate, critical) without errors.
- In the critical-condition radar profile, every cell type's spoke toward MC and NKT-p exceeds the corresponding control and moderate values (consistent higher communication with MC/NKT-p).
- Both immune (MoMa, nrMa, CTL, Treg) and epithelial (ciliated, secretory) cell types show the same pattern, not just a subset.
- App launched with reproducible = TRUE, so re-running the session reproduces the same interaction counts and radar shapes.

## Limitations

- InterCellar relies on pre-computed interactions whose validity depends on the upstream prediction tool's reference database and statistical assumptions; predicted interactions still require experimental validation (e.g., smFISH or flow cytometry co-occurrence).
- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); other published tools are not accepted.
- CellChat v1 repository (sqjin/CellChat) is archived and no longer maintained; the project moved to jinworks/CellChat, so pre-computed inputs may need regeneration from the v2 codebase.

## Evidence

- [abstract] InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [discussion] the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells: "the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells"
- [intro] functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [intro] The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data): "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [readme] InterCellar::run_app( reproducible = TRUE ) ensures reproducible results across R sessions (README): "The flag `reproducible = TRUE` ensures that your results will be reproducible across R sessions."
