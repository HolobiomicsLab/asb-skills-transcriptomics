---
name: cluster-pair-interaction-score-plotting
description: Use when pre-computed ligand-receptor interactions (CCI data) need to
  be filtered to specific genes and cluster-pairs and visualized as interaction-score
  dot plots across multiple datasets/conditions, e.g. to test whether a communication
  pattern is disease-specific.
when_to_use_negative:
- Input is raw scRNA-seq count data rather than pre-computed ligand-receptor interactions
  — InterCellar's workflow 'builds upon pre-computed, inferred ligand-receptor interactions'
- The question requires functional annotation or pathway enrichment of interactions
  — use the function-verse instead of gene-verse dot plotting
- The analysis requires programming-level customization beyond interactive subsetting
  — InterCellar is an interactive Shiny app, not a scripting API
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: InterCellar
  role: Interactive R/Shiny platform used to upload CCI data, filter int-pairs in
    the gene-verse, and render interaction-score dot plots per condition
  repo: github.com__martaint__InterCellar
- name: R/Shiny
  role: Implementation framework of InterCellar; the app is launched via library(InterCellar)
    and InterCellar::run_app()
- name: martaint/InterCellar-reproducibility
  role: Supplementary repository containing walkthrough of InterCellar's analysis
    on the COVID-19 datasets to reproduce manuscript results
  repo: martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:da46fa3827c26e55d730d4b7181e31705a26bb37576addabf47d9b814b73895a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - InterCellar
  - R/Shiny
  - martaint/InterCellar-reproducibility
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Cluster-pair interaction score plotting

## Summary

A skill for rendering dot plots of interaction scores for filtered int-pairs within InterCellar's gene-verse, restricted to selected cluster-pairs, to visualize condition-specific cell-cell communication (e.g., chemokine ligand-receptor pairs between secretory/secretory-diff cells and neutrophils in COVID-19 datasets).

## When to use

Use when pre-computed ligand-receptor interactions (CCI data) need to be filtered to specific genes and cluster-pairs and visualized as interaction-score dot plots across multiple datasets/conditions, e.g. to test whether a communication pattern is disease-specific.

## When NOT to use

- Input is raw scRNA-seq count data rather than pre-computed ligand-receptor interactions — InterCellar's workflow 'builds upon pre-computed, inferred ligand-receptor interactions'
- The question requires functional annotation or pathway enrichment of interactions — use the function-verse instead of gene-verse dot plotting
- The analysis requires programming-level customization beyond interactive subsetting — InterCellar is an interactive Shiny app, not a scripting API

## Inputs

- Pre-computed, inferred ligand-receptor interaction datasets (CCI data) from InterCellar-supported tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR), one per condition
- Set of target ligand genes to filter int-pairs (e.g. CXCL1, CXCL3, CXCL6, CXCL16, CXCL17)
- Selected cell cluster-pairs (e.g. secretory and secretory-diff clusters vs neutrophils)

## Outputs

- Dot plots of interaction scores for the filtered int-pairs, one per dataset/condition
- Exported interaction score table
- Exported dot plot figure via InterCellar's download options

## How to apply

Launch the InterCellar Shiny app with `InterCellar::run_app(reproducible = TRUE)` and upload the pre-computed, inferred ligand-receptor interactions as CCI data in the first workflow step. In the gene-verse, use the interactive filtering options to subset the int-pairs to the target ligand genes (in the demonstrated case, the chemokines CXCL1, CXCL3, CXCL6, CXCL16, CXCL17), then restrict the filtered int-pairs to the cluster-pairs of interest (secretory and secretory-diff cell clusters communicating with neutrophils). Render dot plots of interaction scores for the filtered int-pairs, one plot per dataset/condition, so that communication patterns can be compared across conditions and those uniquely found in each condition highlighted (e.g. chemokine-driven communication with neutrophils enriched in COVID-19 and absent in healthy controls). Finally, use InterCellar's download options to export the dot plot figure and the interaction score table for reproducibility.

## Related tools

- **InterCellar** (Interactive R/Shiny platform used to upload CCI data, filter int-pairs in the gene-verse, and render interaction-score dot plots per condition) — github.com__martaint__InterCellar
- **R/Shiny** (Implementation framework of InterCellar; the app is launched via library(InterCellar) and InterCellar::run_app())
- **martaint/InterCellar-reproducibility** (Supplementary repository containing walkthrough of InterCellar's analysis on the COVID-19 datasets to reproduce manuscript results) — martaint/InterCellar-reproducibility

## Examples

```
library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- Dot plots are rendered one per dataset/condition, so the number of plots matches the number of uploaded CCI datasets
- The plotted int-pairs are restricted to the selected chemokine genes (CXCL1, CXCL3, CXCL6, CXCL16, CXCL17) and the selected cluster-pairs (secretory/secretory-diff vs neutrophils)
- The dot plots across conditions show chemokine-driven communication involving neutrophils enriched in COVID-19 disease conditions and absent in healthy controls
- Dot plot figure and interaction score table are successfully exported via InterCellar's download options
- Running the app with reproducible = TRUE yields reproducible results across R sessions

## Limitations

- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence)
- Comparison across conditions relies on the uploaded CCI datasets being derived from the same reference/assumptions; prediction methods rely on different reference databases and statistical assumptions

## Evidence

- [intro] In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] Workflow builds on pre-computed CCI data: "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [intro] Comparing multiple conditions highlights condition-specific patterns: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [intro] Download options support reproducibility and result collection: "InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures."
- [readme] reproducible = TRUE ensures cross-session reproducibility: "The flag `reproducible = TRUE` ensures that your results will be reproducible across R sessions."
