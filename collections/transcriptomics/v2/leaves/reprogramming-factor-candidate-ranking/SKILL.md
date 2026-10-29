---
name: reprogramming-factor-candidate-ranking
description: Use when single-cell RNA-seq data with annotated subpopulations (e.g.,
  GSE75478 human HSPC data) is available and you need ranked candidate regulators
  to overexpress or knock down to convert one cell type (B-cells, mono/DCs) into another
  (HSPCs).
when_to_use_negative:
- Trajectory structure must be inferred de novo without pre-annotated source and target
  subpopulations — CellRouter's reprogramming scoring is subpopulation-aware and requires
  defined start and end states
- Data types other than single-cell transcriptomics (e.g., single-cell chromatin accessibility,
  DNA methylation, mass cytometry) — the article only envisions these as future applications,
  not demonstrated ones
- The goal is functional validation of engraftable HSC-like cells — the article explicitly
  defers thorough characterization of each identified gene beyond the scope of the
  computational ranking
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellRouter
  role: 'Core platform: reconstructs GRNs, identifies multi-state trajectories via
    flow networks, and scores genes/regulators along transitions to rank reprogramming
    candidates'
  repo: https://github.com/edroaldo/cellrouter
- name: igraph
  role: Community detection used to identify subpopulations from learned data structure
    before trajectory reconstruction
- name: mclust
  role: Model-based clustering supporting subpopulation identification combined with
    community detection
- name: STEMNET
  role: Embedding of cells used prior to subpopulation identification in the reprogramming
    candidate workflow
- name: Rtsne
  role: Dimensionality reduction dependency for the CellRouter R environment
- name: pheatmap
  role: Annotated heatmap visualization of gene signature/clustering results in the
    CellRouter pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:917ec3f16ac0cff52e8dd0ebf8747623cd34bbcfadae2611efda924193bb29cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - Rtsne
  - STEMNET
  - cccd
  - ggplot2
  - igraph
  - mclust
  - pheatmap
  - reshape
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# reprogramming-factor-candidate-ranking

## Summary

Rank overexpression and knockdown candidate transcriptional regulators for cell reprogramming by scoring genes along CellRouter flow-network trajectories between source and target subpopulations, integrating the reconstructed gene regulatory network with transition-specific expression dynamics.

## When to use

Use when single-cell RNA-seq data with annotated subpopulations (e.g., GSE75478 human HSPC data) is available and you need ranked candidate regulators to overexpress or knock down to convert one cell type (B-cells, mono/DCs) into another (HSPCs).

## When NOT to use

- Trajectory structure must be inferred de novo without pre-annotated source and target subpopulations — CellRouter's reprogramming scoring is subpopulation-aware and requires defined start and end states
- Data types other than single-cell transcriptomics (e.g., single-cell chromatin accessibility, DNA methylation, mass cytometry) — the article only envisions these as future applications, not demonstrated ones
- The goal is functional validation of engraftable HSC-like cells — the article explicitly defers thorough characterization of each identified gene beyond the scope of the computational ranking

## Inputs

- Single-cell RNA-seq expression matrix (e.g., GSE75478 human HSPC data)
- Subpopulation annotations or cluster labels for each cell
- Optional: published reprogramming factor list used as validation reference (67 factors)

## Outputs

- reprogramming_candidates.csv containing ranked overexpression and knockdown candidate lists per lineage transition
- Lineage-specific knockdown candidate rankings (e.g., DNMT1, H3F3A, JARID2, KMT2D for B-cell to HSPC)
- Transition-specific gene expression scores along reconstructed trajectories

## How to apply

Load the single-cell expression matrix and subpopulation annotations into R, source the CellRouter class with libdir pointing to the CellRouter Java libraries folder, and identify subpopulations by combining learning the structure of the data with community detection (igraph/mclust). Reconstruct the gene regulatory network from the single cells to define putative gene regulatory relationships, then run CellRouter flow-network trajectory reconstruction between chosen source and target subpopulations (e.g., subpopulation 10 B-cells and 17 mono/DCs to subpopulation 12 HSPCs in GSE75478). Score genes along each transition trajectory to rank overexpression candidates — validated by recovery of 11 of the 67 published reprogramming factors — and knockdown candidates such as DNMT1/H3F3A/JARID2/KMT2D for B-cell-to-HSPC and DNMT1/H2AFZ/TRIM28/KMT2D/H2AFV for mono/DC-to-HSPC. Emit both ranked lists (e.g., reprogramming_candidates.csv) and verify expected regulators appear in the lineage-specific knockdown lists.

## Related tools

- **CellRouter** (Core platform: reconstructs GRNs, identifies multi-state trajectories via flow networks, and scores genes/regulators along transitions to rank reprogramming candidates) — https://github.com/edroaldo/cellrouter
- **igraph** (Community detection used to identify subpopulations from learned data structure before trajectory reconstruction)
- **mclust** (Model-based clustering supporting subpopulation identification combined with community detection)
- **STEMNET** (Embedding of cells used prior to subpopulation identification in the reprogramming candidate workflow)
- **Rtsne** (Dimensionality reduction dependency for the CellRouter R environment)
- **pheatmap** (Annotated heatmap visualization of gene signature/clustering results in the CellRouter pipeline)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'; list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')
```

## Evaluation signals

- reprogramming_candidates.csv exists and contains both overexpression and knockdown ranked lists
- Exactly 11 of the 67 published reprogramming factors are recovered among the overexpression candidates
- B-cell-to-HSPC knockdown list contains DNMT1, H3F3A, JARID2, and KMT2D
- Mono/DC-to-HSPC knockdown list contains DNMT1, H2AFZ, TRIM28, KMT2D, and H2AFV
- Trajectories were reconstructed from subpopulation 10 (B-cells) and 17 (mono/DCs) to subpopulation 12 (HSPCs) without error

## Limitations

- Functional characterization of genes identified for generating engraftable HSC-like cells with long-term multilineage potential is deferred to future work and beyond the scope of the manuscript
- Application to single-cell chromatin accessibility, DNA methylation, and mass cytometry data is only envisioned as future work, not demonstrated
- CellRouter was tested in Unix-based systems only; the README strongly recommends Unix/Mac and states it was not tested in Windows

## Evidence

- [intro] CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions: "CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions"
- [methods] a gene regulatory network is reconstructed to identify putative gene regulatory relationships: "a gene regulatory network is reconstructed to identify putative gene regulatory relationships"
- [methods] source the CellRouter class and make sure you provide the correct directory for the Java libraries: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [readme] CellRouter was tested in Unix-based systems. Therefore, we strongly recommend to use CellRouter in Unix-based systems or Mac.: "CellRouter was tested in Unix-based systems. Therefore, we strongly recommend to use CellRouter in Unix-based systems or Mac. CellRouter was not tested in Windows."
- [readme] list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd'): "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
