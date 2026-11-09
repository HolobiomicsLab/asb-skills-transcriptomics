---
name: malignant-cell-communication-analysis-melanoma
description: Use when analyzing predicted ligand-receptor interactions from a tumor
  scRNA-seq dataset from the viewpoint of a specific malignant-cell population, and
  you need functionally grouped interaction modules (int-pair modules) rather than
  a flat interaction list.
when_to_use_negative:
- You need de novo inference of ligand-receptor interactions from raw scRNA-seq counts
  — InterCellar builds on pre-computed CCI data (e.g., from CellChat), not raw data.
- You need a fully scripted/non-interactive pipeline — InterCellar's int-pair module
  workflow is performed interactively in the Shiny app.
- Your CCI data comes from a tool outside InterCellar's supported set (CellChat, CellPhoneDB,
  ICELLNET, SingleCellSignalR).
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: InterCellar
  role: Interactive R/Shiny platform that ingests the CCI data, provides cluster-verse
    subsetting to malignant-cell outgoing interactions, function-verse annotation,
    and int-pair module definition via UMAP + hierarchical clustering
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Generates the predicted ligand-receptor CCI data on the melanoma dataset that
    serves as InterCellar's input
  repo: https://github.com/sqjin/CellChat
- name: R/Shiny
  role: Implementation/runtime environment for launching the InterCellar app locally
  repo: https://github.com/martaint/InterCellar
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:c8ed44cc64c7e97fc318c9bc0d478490567b9071e69820aaab043c7be13d21a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  - R/Shiny
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# malignant-cell-communication-analysis-melanoma

## Summary

Applies InterCellar's int-pair module analysis to CellChat-predicted cell-cell interactions, subset to directed-outgoing interactions of malignant cells in the Tirosh metastatic melanoma dataset (GSE72056), yielding nine modules that reveal ECM/integrin signaling (module #6) and ANXA1–FPR malignant-to-macrophage signaling (module #8) relevant to melanoma invasiveness.

## When to use

Use when analyzing predicted ligand-receptor interactions from a tumor scRNA-seq dataset from the viewpoint of a specific malignant-cell population, and you need functionally grouped interaction modules (int-pair modules) rather than a flat interaction list.

## When NOT to use

- You need de novo inference of ligand-receptor interactions from raw scRNA-seq counts — InterCellar builds on pre-computed CCI data (e.g., from CellChat), not raw data.
- You need a fully scripted/non-interactive pipeline — InterCellar's int-pair module workflow is performed interactively in the Shiny app.
- Your CCI data comes from a tool outside InterCellar's supported set (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR).

## Inputs

- CellChat-predicted CCI dataset for the Tirosh metastatic melanoma dataset (GSE72056)
- Malignant-cell annotations/clusters from the melanoma scRNA-seq data
- InterCellar-supported CCI input (CellChat, CellPhoneDB, ICELLNET, or SingleCellSignalR output)

## Outputs

- Nine int-pair modules of malignant-cell directed-outgoing interactions (module assignment table)
- UMAP embedding and hierarchical-clustering dendrogram of interaction pairs
- Module #6: ECM organization/integrin signaling module (collagen/laminin genes with integrin complexes)
- Module #8: ANXA1 (malignant cell) – FPR (macrophage) interaction
- Downloaded tables and figures from InterCellar

## How to apply

Load the CellChat-derived CCI data for the Tirosh metastatic melanoma dataset (GSE72056) into the InterCellar Shiny app (launched with InterCellar::run_app(reproducible = TRUE)). In the cluster-verse, interactively filter and subset the CCI data to directed outgoing interactions from the malignant-cell viewpoint. In the function-verse, annotate the enriched interaction pairs using InterCellar's functional annotation resources (Gene Ontology and pathway databases), then run the int-pair module definition, which embeds interaction pairs via UMAP and groups them by hierarchical clustering into modules of interactions with similar functional patterns. Inspect the dendrogram/modules to verify nine modules are recovered, confirming that module #6 groups collagen- and laminin-encoding genes from malignant cells interacting with integrin complexes (ECM organization/integrin signaling) and that module #8 contains the ANXA1 (malignant cell) to FPR (macrophage-exclusive) interaction. Export the module assignment table and UMAP/hierarchy visualizations using InterCellar's download options.

## Related tools

- **InterCellar** (Interactive R/Shiny platform that ingests the CCI data, provides cluster-verse subsetting to malignant-cell outgoing interactions, function-verse annotation, and int-pair module definition via UMAP + hierarchical clustering) — https://github.com/martaint/InterCellar
- **CellChat** (Generates the predicted ligand-receptor CCI data on the melanoma dataset that serves as InterCellar's input) — https://github.com/sqjin/CellChat
- **R/Shiny** (Implementation/runtime environment for launching the InterCellar app locally) — https://github.com/martaint/InterCellar

## Examples

```
R -e 'library(InterCellar); InterCellar::run_app(reproducible = TRUE)'
```

## Evaluation signals

- The int-pair module definition recovers exactly nine modules from the malignant-cell outgoing-interaction subset.
- Module #6 groups collagen- and laminin-encoding genes from malignant cells interacting with integrin-complexes (ECM organization/integrin signaling).
- Module #8 contains the ANXA1 (malignant cell) with FPR interaction, with FPR exclusively expressed by macrophages.
- The exported module assignment table is consistent across R sessions when the app is run with reproducible = TRUE.
- UMAP and dendrogram visualizations downloaded from InterCellar show module grouping consistent with the module assignment table.

## Limitations

- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); other published tools are planned future work.
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence).
- Results of the run rely on the CellChat reference database and statistical assumptions, which differ from other prediction tools.

## Evidence

- [intro] int-pair modules, which are defined as groups of interactions with similar functional patterns: "the analysis of the so-called int-pair modules, which are defined as groups of interactions with similar functional patterns"
- [intro] Interactive filtering and subsetting: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] Function-verse annotation: "in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [readme] Reproducible app launch: "InterCellar::run_app( reproducible = TRUE )"
- [intro] Supported input tools: "either CCI data generated by InterCellar-supported tools (CellChat12, CellPhoneDB11, ICELLNET15, or SingleCellSignalR16)"
- [intro] Download/export for reproducibility: "InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures."
- [discussion] Validation limitation: "we want to stress the importance of further experimental validation of the predicted interactions"
