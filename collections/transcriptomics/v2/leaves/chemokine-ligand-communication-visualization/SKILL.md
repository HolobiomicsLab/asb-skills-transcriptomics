---
name: chemokine-ligand-communication-visualization
description: Use when pre-computed ligand-receptor interactions from scRNA-seq data
  (e.g., COVID-19 CCI datasets) need targeted visualization of a chemokine ligand
  family's communication between defined cell clusters, to test enrichment in disease
  vs. healthy controls.
when_to_use_negative:
- Input is raw scRNA-seq count data or clustering output without pre-computed ligand-receptor
  interactions — InterCellar's workflow builds upon pre-computed, inferred interactions
  that must be inferred first with a supported tool
- Interactions were inferred by a tool outside InterCellar's supported input set (only
  CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR are supported)
- The goal is de novo prediction of ligand-receptor interactions or statistical inference
  rather than interactive downstream exploration and visualization of existing CCI
  results
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: InterCellar
  role: R/Shiny app used to upload CCI data, interactively filter int-pairs by chemokine
    ligand genes and cluster-pairs in the gene-verse, render dot plots of interaction
    scores, compare conditions, and export figures/tables
  repo: https://github.com/martaint/InterCellar
- name: R/Shiny
  role: Implementation framework in which the InterCellar app runs; installed as a
    Bioconductor package (requires R 4.1, Bioconductor 3.14)
  repo: https://github.com/martaint/InterCellar
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:5702bb6e6a4bd28eba015944a3861e1d07ba89929ea4e035055e6f38cea28517
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - InterCellar
  - R/Shiny
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chemokine-ligand-communication-visualization

## Summary

Visualize chemokine-mediated cell-cell communication by filtering InterCellar's gene-verse dot plots to selected CXCL ligand-receptor pairs (CXCL1, CXCL3, CXCL6, CXCL16, CXCL17) and specific cluster-pairs, then comparing plots across disease and control conditions.

## When to use

Use when pre-computed ligand-receptor interactions from scRNA-seq data (e.g., COVID-19 CCI datasets) need targeted visualization of a chemokine ligand family's communication between defined cell clusters, to test enrichment in disease vs. healthy controls.

## When NOT to use

- Input is raw scRNA-seq count data or clustering output without pre-computed ligand-receptor interactions — InterCellar's workflow builds upon pre-computed, inferred interactions that must be inferred first with a supported tool
- Interactions were inferred by a tool outside InterCellar's supported input set (only CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR are supported)
- The goal is de novo prediction of ligand-receptor interactions or statistical inference rather than interactive downstream exploration and visualization of existing CCI results

## Inputs

- Pre-computed, inferred ligand-receptor interaction datasets (CCI data) from supported tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR), one per condition/dataset
- Cluster annotation linking clusters to cell types (e.g., secretory, secretory-diff, neutrophils)
- Target ligand gene list (CXCL1, CXCL3, CXCL6, CXCL16, CXCL17)

## Outputs

- Gene-verse dot plots of interaction scores for filtered chemokine int-pairs, one per dataset/condition
- Exported dot plot figure files
- Exported interaction score table for the filtered int-pairs

## How to apply

Launch the InterCellar Shiny app (InterCellar::run_app with reproducible = TRUE) and upload the pre-computed ligand-receptor interaction CCI data, one dataset per condition. In the gene-verse, use the interactive filtering options to subset int-pairs to the selected chemokine ligand genes CXCL1, CXCL3, CXCL6, CXCL16, CXCL17. Restrict the filtered int-pairs to cluster-pairs involving the secretory and secretory-diff cell clusters. Render dot plots of interaction scores for the filtered int-pairs, one plot per dataset/condition, and compare across conditions to confirm that chemokine-driven communication involving neutrophils is enriched in COVID-19 disease conditions and absent in healthy controls. Finally, use InterCellar's download options to export both the dot plot figure and the interaction score table for reproducibility.

## Related tools

- **InterCellar** (R/Shiny app used to upload CCI data, interactively filter int-pairs by chemokine ligand genes and cluster-pairs in the gene-verse, render dot plots of interaction scores, compare conditions, and export figures/tables) — https://github.com/martaint/InterCellar
- **R/Shiny** (Implementation framework in which the InterCellar app runs; installed as a Bioconductor package (requires R 4.1, Bioconductor 3.14)) — https://github.com/martaint/InterCellar

## Examples

```
BiocManager::install("InterCellar"); library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- The filtered int-pair set contains only interactions whose ligands are among CXCL1, CXCL3, CXCL6, CXCL16, CXCL17
- Dot plots are restricted to cluster-pairs involving secretory and secretory-diff cells on the ligand side and neutrophils as partners
- One dot plot of interaction scores is rendered per dataset/condition, enabling cross-condition comparison
- The COVID-19 dot plots show clear chemokine ligand-receptor enrichment in secretory/secretory-diff to neutrophil communication, while the healthy-control plot lacks this enrichment
- Both the dot plot figure and the interaction score table are exported via InterCellar's download options, and the app was launched with reproducible = TRUE

## Limitations

- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- The workflow is limited to downstream interactive analysis of pre-computed interactions — it does not itself infer ligand-receptor interactions
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence)

## Evidence

- [intro] In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] Multiple-condition comparison rationale: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [abstract] COVID-19 analysis motivation: "By analyzing COVID-19 and melanoma cell-cell interactions, we show that InterCellar resolves data-driven patterns of communication and highlights molecular signals"
- [readme] Tool description from README: "an R/Shiny app for interactive analysis and exploration of cell-cell communication based on single-cell transcriptomics data"
- [readme] App launch command from README: "InterCellar::run_app( reproducible = TRUE )"
