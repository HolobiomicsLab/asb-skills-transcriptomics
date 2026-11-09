---
name: gene-verse-int-pair-filtering
description: Use when you have pre-computed ligand-receptor interaction (CCI) data
  loaded in InterCellar and need to test whether a specific ligand family (e.g. chemokines)
  or set of cluster-pairs (e.g. secretory-to-neutrophil) drives communication in one
  condition versus another.
when_to_use_negative:
- The input is raw scRNA-seq count matrices rather than pre-computed, inferred ligand-receptor
  interactions — InterCellar's workflow builds upon pre-computed CCI data.
- The goal is functional annotation of enriched interactions or int-pair module detection
  — those belong to the function-verse, not the gene-verse filtering workflow.
- The tool used to predict interactions is not among the InterCellar-supported inputs
  (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); support for other tools is
  planned future work.
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: InterCellar
  role: R/Shiny app providing the gene-verse universe and interactive filtering/visualization
    options used to subset int-pairs and render dot plots
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: One of the InterCellar-supported upstream tools whose CCI output can be loaded
    as input for gene-verse filtering
  repo: https://github.com/sqjin/CellChat
- name: ICELLNET
  role: One of the InterCellar-supported upstream tools whose ligand-receptor predictions
    can be uploaded as CCI data
  repo: https://github.com/soumelis-lab/ICELLNET
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:4512fe3e991dddc6b6a0d02ffd15bb73b6688f60005e37fbdc9840f10ccf1184
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - ICELLNET
  - InterCellar
  - R/Shiny
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# gene-verse-int-pair-filtering

## Summary

Interactively filter and subset predicted ligand-receptor interactions in InterCellar's gene-verse — restricting int-pairs by ligand genes (e.g. chemokines CXCL1, CXCL3, CXCL6, CXCL16, CXCL17) and cluster-pairs — then render dot plots of interaction scores to reveal gene-centric communication patterns across conditions.

## When to use

Use when you have pre-computed ligand-receptor interaction (CCI) data loaded in InterCellar and need to test whether a specific ligand family (e.g. chemokines) or set of cluster-pairs (e.g. secretory-to-neutrophil) drives communication in one condition versus another.

## When NOT to use

- The input is raw scRNA-seq count matrices rather than pre-computed, inferred ligand-receptor interactions — InterCellar's workflow builds upon pre-computed CCI data.
- The goal is functional annotation of enriched interactions or int-pair module detection — those belong to the function-verse, not the gene-verse filtering workflow.
- The tool used to predict interactions is not among the InterCellar-supported inputs (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); support for other tools is planned future work.

## Inputs

- Pre-computed, inferred ligand-receptor interactions uploaded as cell-cell interactions (CCI data), e.g. from InterCellar-supported tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- Multiple CCI datasets representing different conditions (e.g. COVID-19 severe/critical and healthy control)
- A ligand gene selection list (e.g. chemokines CXCL1, CXCL3, CXCL6, CXCL16, CXCL17)
- Cell cluster annotation shared across the datasets

## Outputs

- Dot plots of interaction scores for filtered int-pairs, one per dataset/condition
- Filtered interaction score table
- Downloadable figure and table files exported from InterCellar

## How to apply

Upload the pre-computed, inferred ligand-receptor interaction dataset(s) as CCI data in the InterCellar Shiny app (R/Shiny, launched via InterCellar::run_app(reproducible = TRUE)). In the gene-verse, use the interactive filtering options to subset int-pairs to a selected ligand gene list — for the COVID-19 chemokine analysis this was CXCL1, CXCL3, CXCL6, CXCL16 and CXCL17. Restrict the filtered int-pairs to cluster-pairs involving the cell clusters of interest (secretory and secretory-diff cells vs neutrophils). Render dot plots of interaction scores for the filtered int-pairs, one per dataset/condition, and compare them across conditions to highlight communication patterns uniquely found in each condition. Export the dot plot figure and the interaction score table via InterCellar's download options for reproducibility.

## Related tools

- **InterCellar** (R/Shiny app providing the gene-verse universe and interactive filtering/visualization options used to subset int-pairs and render dot plots) — https://github.com/martaint/InterCellar
- **CellChat** (One of the InterCellar-supported upstream tools whose CCI output can be loaded as input for gene-verse filtering) — https://github.com/sqjin/CellChat
- **ICELLNET** (One of the InterCellar-supported upstream tools whose ligand-receptor predictions can be uploaded as CCI data) — https://github.com/soumelis-lab/ICELLNET

## Examples

```
library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- The filtered int-pairs set contains only interactions whose ligand genes match the selection (e.g. CXCL1, CXCL3, CXCL6, CXCL16, CXCL17).
- The dot plot restricts to cluster-pairs involving the chosen clusters (secretory and secretory-diff cells vs neutrophils) and no others.
- Comparing dot plots across conditions reproduces the expected pattern: chemokine-driven communication involving neutrophils is enriched in COVID-19 datasets and absent in healthy controls.
- The exported interaction score table contains interaction-score values consistent with the rendered dot plots and is reproducible across R sessions when run with reproducible = TRUE.

## Limitations

- InterCellar supports only a limited set of input CCI tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work.
- Interactions are predicted, not experimentally validated; findings should be confirmed experimentally, e.g. by smFISH or flow cytometry co-occurrence.
- InterCellar runs locally, so unlike remote web-based tools there is no server-side sharing, but analysis is limited to supported CCI input formats.

## Evidence

- [intro] the analysis focuses on three biological domains, named universes: cell clusters, genes, and biological functions. In each universe (cluster-verse, gene-verse, and function-verse), different: [evidence span withheld — anchor status: not_found]
- [intro] In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [readme] Starting from pre-computed ligand-receptor interactions, `InterCellar` provides filtering options, annotations and multiple visualizations to explore clusters, genes and functions.: [evidence span withheld — anchor status: external_doc]
- [intro] InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures.: "InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures."
