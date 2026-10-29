---
name: per-cluster-interaction-count-aggregation
description: Use when you have pre-computed CCI datasets (e.g., CellPhoneDB output)
  for two or more conditions and want to identify cell clusters whose total interaction
  counts differ strikingly between conditions.
when_to_use_negative:
- Input is raw scRNA-seq data or cluster assignments without pre-computed ligand-receptor
  interactions — InterCellar's workflow starts from CCI data, not expression matrices
- The CCI data comes from a prediction tool outside InterCellar's supported input
  set (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- The question concerns which genes or pathways drive the interactions — this belongs
  to the gene-verse or function-verse, not per-cluster count aggregation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3366
tools:
- name: InterCellar
  role: Interactive R/Shiny app used to upload CCI data, aggregate per-cluster interaction
    counts in the cluster-verse, and render the back-to-back barplot
  repo: https://github.com/martaint/InterCellar
- name: CellPhoneDB
  role: Source of the pre-computed ligand-receptor interaction data uploaded into
    InterCellar for the COVID-19 critical vs. moderate comparison
- name: R
  role: Runtime environment for the InterCellar Bioconductor package (requires R 4.1
    / Bioconductor 3.14)
  repo: https://github.com/martaint/InterCellar
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:0732e3ce4a874f89b25bd7fd5a727dbc1415f67c38b805ad3753ce16ad7a5a15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellPhoneDB
  - InterCellar
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# per-cluster-interaction-count-aggregation

## Summary

Aggregate predicted ligand-receptor interaction counts per cell cluster in InterCellar's cluster-verse, classifying each interaction as autocrine or paracrine, then render a back-to-back barplot to compare per-cluster totals between conditions (e.g., critical vs. moderate COVID-19).

## When to use

Use when you have pre-computed CCI datasets (e.g., CellPhoneDB output) for two or more conditions and want to identify cell clusters whose total interaction counts differ strikingly between conditions.

## When NOT to use

- Input is raw scRNA-seq data or cluster assignments without pre-computed ligand-receptor interactions — InterCellar's workflow starts from CCI data, not expression matrices
- The CCI data comes from a prediction tool outside InterCellar's supported input set (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- The question concerns which genes or pathways drive the interactions — this belongs to the gene-verse or function-verse, not per-cluster count aggregation

## Inputs

- Pre-computed, inferred ligand-receptor interaction datasets (CCI data, e.g., CellPhoneDB-derived) for each condition/dataset
- Cell cluster annotations per interaction (source and target clusters)

## Outputs

- Per-cluster total interaction counts per dataset, split by interaction flow (autocrine/paracrine)
- Back-to-back barplot comparing per-cluster interaction counts between conditions (exported as back_to_back_barplot.png)

## How to apply

Upload the pre-computed ligand-receptor interaction files as CCI datasets into InterCellar (R/Shiny, launched with reproducible=TRUE for cross-session reproducibility). In the cluster-verse, classify each interaction as autocrine when the source cluster equals the target cluster and paracrine when they differ, then aggregate interaction counts per cell cluster across all interaction flows for each dataset. Render the back-to-back barplot juxtaposing per-cluster interaction counts between the two conditions — in the InterCellar COVID-19 analysis this comparison highlighted a gain of interactions for mast cells (MC) and proliferating natural killer T cells (NKT-p) in critical versus moderate cases, with higher macrophage and lower T cell counts in critical cases. Export the figure via InterCellar's download options to collect and finalize results.

## Related tools

- **InterCellar** (Interactive R/Shiny app used to upload CCI data, aggregate per-cluster interaction counts in the cluster-verse, and render the back-to-back barplot) — https://github.com/martaint/InterCellar
- **CellPhoneDB** (Source of the pre-computed ligand-receptor interaction data uploaded into InterCellar for the COVID-19 critical vs. moderate comparison)
- **R** (Runtime environment for the InterCellar Bioconductor package (requires R 4.1 / Bioconductor 3.14)) — https://github.com/martaint/InterCellar

## Examples

```
library(InterCellar); InterCellar::run_app( reproducible = TRUE )
```

## Evaluation signals

- The generated back-to-back barplot reproduces the reported per-cluster count pattern: gain for mast cells (MC) and proliferating NKT cells (NKT-p) in critical vs. moderate
- Macrophage interaction counts are higher and T cell counts lower in critical versus moderate cases
- Every interaction in each dataset is classified as exactly one of autocrine (source cluster equals target cluster) or paracrine (source and target differ)
- Per-cluster totals equal the sum across all interaction flows and are consistent across the three uploaded datasets
- The plot is exported (back_to_back_barplot.png) using InterCellar's reproducible=TRUE mode so results match across R sessions

## Limitations

- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); other published tools are planned as future work
- Predicted interactions await experimental validation, e.g., by single-molecule fluorescence in situ hybridization or flow cytometry co-occurrence
- Benchmark comparison excluded 13 methods requiring programming skills and was a snapshot as of April 2021, so downstream-analysis coverage is not exhaustive

## Evidence

- [intro] The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data): "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [intro] In each universe, filtering and visualization options let users interactively subset the data.: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [abstract] InterCellar supports comparison of cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [discussion] Critical COVID-19 patients show a consistent communication pattern involving mast cells.: "the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells"
- [readme] InterCellar runs locally and each analysis step can be performed interactively without programming skills.: "Every step of the analysis can be performed interactively, thus not requiring any programming skills. Moreover, `InterCellar` runs on your
local machine, avoiding issues related to data privacy."
