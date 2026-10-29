---
name: ligand-receptor-interaction-functional-annotation
description: Use when you have pre-computed ligand-receptor CCI data (e.g. from CellChat
  on scRNA-seq) subset to a specific viewpoint such as directed outgoing interactions
  of one cell cluster, and you need to annotate enriched interaction pairs by biological
  function before grouping them into modules.
when_to_use_negative:
- You need to predict ligand-receptor interactions from raw scRNA-seq expression —
  InterCellar's workflow builds upon pre-computed, inferred ligand−receptor interactions
  and does not perform the prediction itself
- The CCI data come from a tool outside InterCellar's supported input set (CellChat,
  CellPhoneDB, ICELLNET, SingleCellSignalR); the article notes extension to other
  published tools is planned future work
- You require a command-line, scripted, or remotely hosted analysis — InterCellar
  is an interactive local Shiny app, and remote web-based tools raise data privacy
  issues for unpublished patient data
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: InterCellar
  role: R/Shiny platform providing the function-verse for functional annotation of
    enriched interactions and the int-pair module definition (UMAP + hierarchical
    clustering)
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Upstream tool that generates the CCI input data (predicted ligand-receptor
    interactions on the melanoma scRNA-seq dataset) that are then annotated and module-organized
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:9aa6612510d3fc31a9ad73321ff58c772bb2da73911e588f5de8983c0b2608c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ligand-receptor-interaction-functional-annotation

## Summary

Functional annotation of enriched ligand-receptor interaction pairs in InterCellar's function-verse, linking predicted CCI pairs to Gene Ontology and pathway resources so that groups of interactions with similar functional patterns (int-pair modules) can be defined. This is the step that turns raw predicted interactions into biologically interpretable modules, e.g. ECM/integrin signaling or ANXA1–FPR signaling in melanoma.

## When to use

Use when you have pre-computed ligand-receptor CCI data (e.g. from CellChat on scRNA-seq) subset to a specific viewpoint such as directed outgoing interactions of one cell cluster, and you need to annotate enriched interaction pairs by biological function before grouping them into modules.

## When NOT to use

- You need to predict ligand-receptor interactions from raw scRNA-seq expression — InterCellar's workflow builds upon pre-computed, inferred ligand−receptor interactions and does not perform the prediction itself
- The CCI data come from a tool outside InterCellar's supported input set (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); the article notes extension to other published tools is planned future work
- You require a command-line, scripted, or remotely hosted analysis — InterCellar is an interactive local Shiny app, and remote web-based tools raise data privacy issues for unpublished patient data

## Inputs

- Pre-computed ligand-receptor interaction results as a CCI dataset from an InterCellar-supported tool (CellChat, CellPhoneDB, ICELLNET, or SingleCellSignalR)
- CellChat-predicted CCI data on the Tirosh metastatic melanoma dataset (GSE72056)
- User-selected subset of interactions, e.g. directed outgoing interactions of malignant cells chosen interactively in the cluster-verse

## Outputs

- Function-annotated enriched interaction pairs (functional annotation from Gene Ontology and pathway databases)
- Int-pair module assignments (e.g. nine modules for melanoma malignant-cell outgoing interactions), via UMAP embedding and hierarchical clustering
- Dendrogram/UMAP module visualizations and downloadable tables and figures

## How to apply

Load the pre-computed CCI dataset (e.g. CellChat-derived results on the Tirosh metastatic melanoma dataset GSE72056) into the InterCellar Shiny app, then in the cluster-verse interactively filter and subset to the desired viewpoint (e.g. directed outgoing interactions of malignant cells). Switch to the function-verse, where InterCellar provides multiple resources to perform functional annotation of the enriched interaction pairs, drawing on Gene Ontology and pathway databases to link biological functions and pathways to ligand-receptor interactions. The annotated pairs are then embedded via UMAP and grouped by hierarchical clustering into int-pair modules — groups of interactions with similar functional patterns. On melanoma malignant-cell outgoing data this yields nine modules, with module #6 (ECM organization/integrin signaling) composed of collagen- and laminin-encoding genes from malignant cells interacting with integrin complexes, and module #8 containing ANXA1 (sent by malignant cells) with FPR exclusively expressed by macrophages. Export the module assignment table and UMAP/hierarchy visualizations using InterCellar's download options; launch the app with reproducible = TRUE to keep results reproducible across R sessions.

## Related tools

- **InterCellar** (R/Shiny platform providing the function-verse for functional annotation of enriched interactions and the int-pair module definition (UMAP + hierarchical clustering)) — https://github.com/martaint/InterCellar
- **CellChat** (Upstream tool that generates the CCI input data (predicted ligand-receptor interactions on the melanoma scRNA-seq dataset) that are then annotated and module-organized) — https://github.com/sqjin/CellChat

## Examples

```
R> library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- The number of int-pair modules recovered matches expectation: nine modules for melanoma malignant-cell directed-outgoing interactions
- Module #6 groups ECM-organization/integrin-signaling interactions composed of collagen- and laminin-encoding genes from malignant cells interacting with integrin complexes
- Module #8 contains exactly the ANXA1 (malignant cell) — FPR (macrophage) interaction, with FPR exclusively expressed by macrophages
- Annotated pairs are consistent with the chosen viewpoint: only directed outgoing interactions of malignant cells are retained after cluster-verse subsetting
- App was launched with reproducible = TRUE so module assignments and UMAP layouts are reproducible across R sessions

## Limitations

- InterCellar currently supports only a limited set of input CCI tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- Predicted interactions require further experimental validation, e.g. by single-molecule fluorescence in situ hybridization or flow cytometry co-occurrence
- Prediction methods rely on different reference databases and statistical assumptions, making upstream method choice critical for the annotated results

## Evidence

- [intro] Step 3a: data-driven analysis via int-pair modules: "The first enables a focused inspection of cell−cell communication through the analysis of the so-called int-pair modules, which are defined as groups of interactions with similar functional patterns"
- [intro] Function-verse annotation: "For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [intro] Interactive subsetting: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [readme] Reproducible launch: "The flag `reproducible = TRUE` ensures that your results will be reproducible
across R sessions."
- [readme] GO/pathway basis of annotation: "Moreover, based on functional annotation from Gene Ontology
and pathway databases, `InterCellar` implements data-driven analyses to
investigate cell-cell communication in one or multiple conditions."
