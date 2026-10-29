---
name: cci-interaction-filtering-by-cell-type-viewpoint
description: Use when starting from CellChat/CellPhoneDB/ICELLNET/SingleCellSignalR-inferred
  CCI data and the analysis goal requires isolating interactions sent or received
  by one cell type (e.g., outgoing interactions of malignant cells) before module
  definition.
when_to_use_negative:
- The goal is to compare cell-cell communication across multiple conditions/datasets
  rather than inspect one viewpoint's interactions — use InterCellar's multiple-condition
  comparison instead.
- Interactions have not yet been inferred from the scRNA-seq data — InterCellar builds
  on pre-computed, inferred ligand-receptor interactions and does not itself predict
  them.
- Analysis requires the full undirected communication network without focusing on
  a sender/receiver cell type; filtering to one viewpoint would discard the rest of
  the network.
edam_operation: http://edamontology.org/operation_3696
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: Interactive R/Shiny app whose cluster-verse provides the filtering and visualization
    options used to subset CCI data by cell-type viewpoint
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Source of the CellChat-predicted CCI data that is loaded into InterCellar
    and then subsetted to malignant-cell directed-outgoing interactions
  repo: https://github.com/sqjin/CellChat
- name: R
  role: Runtime environment for launching the InterCellar Shiny app (requires R 4.1
    / Bioconductor 3.14)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:82af54c874870101e5cfebd80ed3b0eed16a1f6d5c20e2aa737cd103b3e295c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  edam_operation: http://edamontology.org/operation_3696
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cci-interaction-filtering-by-cell-type-viewpoint

## Summary

Interactively filter and subset a pre-computed CCI dataset to the directed interactions of a single cell-type viewpoint (e.g., malignant-cell directed-outgoing interactions) inside InterCellar's cluster-verse, so that downstream int-pair module analysis reflects a sender- or receiver-specific communication pattern.

## When to use

Use when starting from CellChat/CellPhoneDB/ICELLNET/SingleCellSignalR-inferred CCI data and the analysis goal requires isolating interactions sent or received by one cell type (e.g., outgoing interactions of malignant cells) before module definition.

## When NOT to use

- The goal is to compare cell-cell communication across multiple conditions/datasets rather than inspect one viewpoint's interactions — use InterCellar's multiple-condition comparison instead.
- Interactions have not yet been inferred from the scRNA-seq data — InterCellar builds on pre-computed, inferred ligand-receptor interactions and does not itself predict them.
- Analysis requires the full undirected communication network without focusing on a sender/receiver cell type; filtering to one viewpoint would discard the rest of the network.

## Inputs

- Pre-computed, inferred ligand-receptor interactions as a CCI dataset (e.g., CellChat output; supported tools: CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- Cell cluster/annotation information for the scRNA-seq dataset (e.g., malignant cells vs. immune/stromal cells in the Tirosh metastatic melanoma dataset GSE72056)

## Outputs

- Subset of the CCI data restricted to directed interactions of the selected cell-type viewpoint (e.g., malignant-cell directed-outgoing interactions)
- Filtered cluster-verse view that feeds the function-verse annotation and int-pair module definition

## How to apply

Load the pre-computed, inferred ligand-receptor interactions as a CCI dataset into the InterCellar Shiny app (e.g., CellChat output for the Tirosh metastatic melanoma dataset, GSE72056). In the cluster-verse, use the interactive filtering and visualization options to subset the data to the directed interactions of a chosen cell-type viewpoint — for the melanoma invasiveness task, filter to malignant-cell directed-outgoing interactions so only ligand-receptor pairs sent by malignant cells are retained. This subsetting is done before annotating enriched interaction pairs in the function-verse and before running int-pair module definition (UMAP embedding followed by hierarchical clustering), ensuring the resulting modules reflect the sender-cell perspective. Rationale: each universe (cluster-verse, gene-verse, function-verse) provides filtering options to interactively subset the data, and viewpoint selection determines which functional patterns the modules capture. Set the reproducible flag when launching the app so results are stable across R sessions.

## Related tools

- **InterCellar** (Interactive R/Shiny app whose cluster-verse provides the filtering and visualization options used to subset CCI data by cell-type viewpoint) — https://github.com/martaint/InterCellar
- **CellChat** (Source of the CellChat-predicted CCI data that is loaded into InterCellar and then subsetted to malignant-cell directed-outgoing interactions) — https://github.com/sqjin/CellChat
- **R** (Runtime environment for launching the InterCellar Shiny app (requires R 4.1 / Bioconductor 3.14))

## Examples

```
library(InterCellar); InterCellar::run_app( reproducible = TRUE )
```

## Evaluation signals

- After subsetting, the retained interaction set should contain only pairs with the chosen cell type in the directed role (e.g., every ligand-receptor pair in the melanoma subset is sent by malignant cells).
- Downstream int-pair module definition on the subset should recover nine modules for the melanoma directed-outgoing case, with module #6 grouping ECM-organization/integrin-signaling interactions and module #8 containing the ANXA1 (malignant cell) & FPR (macrophage) interaction.
- The filtered subset should be inspectable in all three universes (cluster-verse, gene-verse, function-verse) with consistent cluster/ligand/receptor assignments.
- Results should be reproducible across R sessions when the app is launched with reproducible = TRUE.

## Limitations

- InterCellar supports only a limited set of input CCI tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); output from other published tools must be converted or is unsupported.
- InterCellar runs locally to avoid data-privacy issues, but it is an interactive Shiny app — no scripted/batch API for this filtering step is described.
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry), so the filtered subset reflects predicted, not validated, communication.
- Installation may fail on missing dependencies ('golem', 'ComplexHeatmap'); Bioconda/Docker installs may need a browser option set (options(browser="firefox")).

## Evidence

- [intro] cluster-verse filtering supports viewpoint subsetting: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] workflow builds on pre-computed CCI data: "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [intro] int-pair modules are groups of interactions with similar functional patterns: "the analysis of the so-called int-pair modules, which are defined as groups of interactions with similar functional patterns"
- [intro] InterCellar supports CellChat as CCI input: "either CCI data generated by InterCellar-supported tools (CellChat12, CellPhoneDB11, ICELLNET15, or SingleCellSignalR16)"
- [readme] launching the app with reproducible results: "The flag `reproducible = TRUE` ensures that your results will be reproducible across R sessions."
