---
name: autocrine-paracrine-interaction-classification
description: Use when you have pre-computed ligand-receptor interaction data (e.g.,
  CellPhoneDB output) for two or more conditions and want to find cell types whose
  total autocrine/paracrine interaction burden shifts between conditions.
when_to_use_negative:
- The input is raw scRNA-seq expression data rather than pre-computed, inferred ligand-receptor
  interactions — InterCellar's workflow builds upon pre-computed CCI data
- The goal is inference of new interactions rather than downstream comparison — use
  the generating tool (e.g., CellPhoneDB, CellChat, ICELLNET) first
- Only a single condition/dataset is available with no second condition to compare
  against — the back-to-back barplot comparison requires multiple datasets
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: R/Shiny app used to upload CCI data, classify interactions as autocrine/paracrine
    in the cluster-verse, aggregate per-cluster counts, and render the back-to-back
    barplot
  repo: martaint/InterCellar
- name: CellPhoneDB
  role: Source tool that produced the pre-computed ligand-receptor interaction datasets
    uploaded into InterCellar
- name: InterCellar-reproducibility
  role: Repository with supplementary data and code (input_data/, intercellar_walkthrough/)
    to reproduce the COVID-19 analyses shown in the manuscript
  repo: martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:ad4f2d8d63a85f6a61d5702c58da30d023c53da9b011f8b85287875d0c8cc2bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellPhoneDB
  - InterCellar
  - InterCellar-reproducibility
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# autocrine-paracrine-interaction-classification

## Summary

Classify predicted ligand-receptor interactions in InterCellar's cluster-verse as autocrine (source cluster = target cluster) or paracrine (source ≠ target cluster), then aggregate per-cell-type interaction counts and compare them across conditions (e.g., critical vs. moderate COVID-19) via a back-to-back barplot.

## When to use

Use when you have pre-computed ligand-receptor interaction data (e.g., CellPhoneDB output) for two or more conditions and want to find cell types whose total autocrine/paracrine interaction burden shifts between conditions.

## When NOT to use

- The input is raw scRNA-seq expression data rather than pre-computed, inferred ligand-receptor interactions — InterCellar's workflow builds upon pre-computed CCI data
- The goal is inference of new interactions rather than downstream comparison — use the generating tool (e.g., CellPhoneDB, CellChat, ICELLNET) first
- Only a single condition/dataset is available with no second condition to compare against — the back-to-back barplot comparison requires multiple datasets

## Inputs

- CellPhoneDB-derived CCI (cell-cell interaction) dataset(s), one per condition, containing pre-computed ligand-receptor interactions with source and target cluster annotations
- Cell cluster annotations for the scRNA-seq data underlying each condition

## Outputs

- Per-cluster autocrine and paracrine interaction counts for each condition dataset
- Back-to-back barplot juxtaposing per-cluster total interaction counts between conditions (e.g., critical vs. moderate COVID-19), exported as back_to_back_barplot.png

## How to apply

Upload each CellPhoneDB-derived CCI dataset into InterCellar as pre-computed ligand-receptor interaction data, then move to the cluster-verse where each interaction can be classified by comparing the source and target cluster fields: identical clusters yield autocrine, differing clusters yield paracrine. Aggregate interaction counts per cell cluster (per flow, or summed across all flows) for each condition dataset. Render InterCellar's back-to-back barplot to juxtapose per-cluster total interaction counts between conditions. The rationale is that condition-specific gains in a cell type's interaction count (e.g., mast cells (MC) and proliferating NKT cells (NKT-p) in critical COVID-19) reveal cell types with altered cell-cell communication, following the paper's comparison of interactions across multiple conditions.

## Related tools

- **InterCellar** (R/Shiny app used to upload CCI data, classify interactions as autocrine/paracrine in the cluster-verse, aggregate per-cluster counts, and render the back-to-back barplot) — martaint/InterCellar
- **CellPhoneDB** (Source tool that produced the pre-computed ligand-receptor interaction datasets uploaded into InterCellar)
- **InterCellar-reproducibility** (Repository with supplementary data and code (input_data/, intercellar_walkthrough/) to reproduce the COVID-19 analyses shown in the manuscript) — martaint/InterCellar-reproducibility

## Examples

```
library(InterCellar); InterCellar::run_app( reproducible = TRUE )
```

## Evaluation signals

- Each interaction is assigned to exactly one of autocrine (source cluster equals target cluster) or paracrine (source and target clusters differ); no interactions are left unclassified
- Per-cluster total counts are computed across all interaction flows for each of the three datasets before comparison
- The back-to-back barplot reproduces the reported pattern: mast cells (MC) and proliferating NKT cells (NKT-p) gain interactions in critical vs. moderate COVID-19
- Macrophage interaction counts are higher and T cell counts lower in critical versus moderate cases in the rendered plot
- The figure is exported as back_to_back_barplot.png via InterCellar's download options

## Limitations

- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); data from other prediction methods require conversion or are unsupported
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence); the autocrine/paracrine split reflects inference, not confirmed biology
- Methods rely on different reference databases and statistical assumptions, so per-cluster count patterns depend on the upstream prediction tool chosen

## Evidence

- [abstract] InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [intro] In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data.: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [discussion] the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells: "the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells"
- [readme] Starting from pre-computed ligand-receptor interactions, InterCellar provides filtering options, annotations and multiple visualizations to explore clusters, genes and functions.: [evidence span withheld — anchor status: not_found]
