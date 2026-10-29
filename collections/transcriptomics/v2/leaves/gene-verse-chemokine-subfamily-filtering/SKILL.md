---
name: gene-verse-chemokine-subfamily-filtering
description: Use when you have pre-computed cell-cell interaction data from multiple
  biological conditions and need to isolate one ligand-receptor subfamily (e.g., CXC/CC
  chemokines) and compute which interactions are unique to each condition.
when_to_use_negative:
- The input is raw scRNA-seq count data rather than pre-computed, inferred ligand-receptor
  interactions — InterCellar's workflow builds upon pre-computed CCI data, so inference
  must first be done with CellChat or a supported tool
- The analysis goal is functional annotation of enriched interactions rather than
  subfamily-level filtering and multi-condition couplet comparison — the function-verse
  pathway is the appropriate route
- The dataset is from a single condition with no multi-condition comparison possible
  — condition-unique couplet percentages cannot be computed
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: InterCellar
  role: R/Shiny platform used to upload CCI data, interactively filter the gene-verse
    to CXC/CC chemokine subfamilies, run multi-condition comparison of int-pair/cluster-pair
    couplets, and generate dot plots and pie charts
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Used upstream to infer the ligand-receptor interactions (CCI data) for critical
    and non-critical COVID-19 conditions that are uploaded into InterCellar
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:9aa3356ce242014419d28c7beb2975a960b17bd3d4d01250f9a81eed99b275c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gene-verse-chemokine-subfamily-filtering

## Summary

In InterCellar's gene-verse, interactively filter a CellChat-derived CCI dataset down to the CXC- and CC-chemokine ligand-receptor subfamilies, then run the multi-condition comparison to quantify phenotype-unique int-pair/cluster-pair couplets (e.g., critical vs. non-critical COVID-19) per cell cluster.

## When to use

Use when you have pre-computed cell-cell interaction data from multiple biological conditions and need to isolate one ligand-receptor subfamily (e.g., CXC/CC chemokines) and compute which interactions are unique to each condition.

## When NOT to use

- The input is raw scRNA-seq count data rather than pre-computed, inferred ligand-receptor interactions — InterCellar's workflow builds upon pre-computed CCI data, so inference must first be done with CellChat or a supported tool
- The analysis goal is functional annotation of enriched interactions rather than subfamily-level filtering and multi-condition couplet comparison — the function-verse pathway is the appropriate route
- The dataset is from a single condition with no multi-condition comparison possible — condition-unique couplet percentages cannot be computed

## Inputs

- Pre-computed CellChat-derived CCI datasets for each condition (e.g., critical and non-critical COVID-19 patients), as InterCellar CCI data
- Cell cluster annotations for the cell types of interest (epithelial, macrophage, NK, T cell clusters)

## Outputs

- Phenotype-unique CXC/CC-chemokine int-pair and cluster-pair couplets per cell cluster
- Proportions of unique couplets favoring each condition (e.g., 47% epithelial and 57% immune CXCL couplets unique to critical; 56% macrophage, 41% NK, 20% T cell favoring critical)
- Dot plots of phenotype-unique chemokine couplets per cell cluster
- Pie charts summarizing the proportion of unique couplets favoring each condition

## How to apply

Upload the pre-computed CellChat-derived CCI datasets (one per condition, e.g., critical and non-critical COVID-19 patients) into InterCellar as CCI data. In the gene-verse, use the interactive filtering options to subset the interactions to the CXC- and CC-chemokine subfamilies before any further analysis, so downstream couplet counts are chemokine-specific. Then run the multi-condition comparison functionality to compute int-pair and cluster-pair couplets uniquely found in each phenotype within defined clusters (e.g., epithelial, macrophage/MoMa, NK, T cells, nrMa, rMa). Generate dot plots of phenotype-unique chemokine couplets per cell cluster and pie charts summarizing the proportion of unique couplets favoring each condition. Rationale: InterCellar's universes (cluster-verse, gene-verse, function-verse) each provide interactive subsetting, and the multi-condition comparison is specifically designed to highlight patterns of communication uniquely found in each condition.

## Related tools

- **InterCellar** (R/Shiny platform used to upload CCI data, interactively filter the gene-verse to CXC/CC chemokine subfamilies, run multi-condition comparison of int-pair/cluster-pair couplets, and generate dot plots and pie charts) — https://github.com/martaint/InterCellar
- **CellChat** (Used upstream to infer the ligand-receptor interactions (CCI data) for critical and non-critical COVID-19 conditions that are uploaded into InterCellar) — https://github.com/sqjin/CellChat

## Examples

```
BiocManager::install("InterCellar"); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- Reproduced proportions match the reported values: 47% epithelial and 57% immune CXCL couplets unique to critical, 56% macrophage, 41% NK, and 20% T cell couplets favoring critical
- Validated ligand usage: CCL4L2 usage in MoMa, CCL7 usage in nrMa, and absence of rMa CCL3 in COVID-19
- Filtered interaction table contains only CXC- and CC-chemokine subfamily ligand-receptor pairs, with all other subfamilies excluded
- Every couplet is classified as unique to exactly one phenotype (control, moderate, critical) per cell cluster; couplets shared across conditions are excluded from uniqueness counts
- Downloaded dot plots and pie charts render per cell cluster with the condition-unique couplet proportions labeled

## Limitations

- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); other CCI prediction tools must be extended upstream
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence), so chemokine couplet assignments are predictions
- Because InterCellar runs locally in R/Shiny rather than remotely, large multi-condition datasets must be processed on the analyst's own machine

## Evidence

- [intro] In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] multi-condition comparison highlights condition-unique communication patterns: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [intro] InterCellar takes pre-computed CCI data from supported tools including CellChat: "either CCI data generated by InterCellar-supported tools (CellChat12, CellPhoneDB11, ICELLNET15, or SingleCellSignalR16)"
- [readme] InterCellar launches as a reproducible Shiny app in R: "InterCellar::run_app( reproducible = TRUE )"
- [readme] The reproducibility repository contains the COVID-19 and melanoma walkthrough data: [evidence span withheld — anchor status: external_doc]
