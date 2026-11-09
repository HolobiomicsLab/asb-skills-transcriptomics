---
name: flow-network-trajectory-reconstruction
description: Use when snapshot single-cell RNA-seq data span multiple cell states
  (e.g., HSPC differentiation to erythroid, myeloid, and lymphoid lineages) and you
  need multi-lineage trajectories between user-chosen source and target subpopulations,
  plus transition-specific genes and regulators.
when_to_use_negative:
- The analysis goal is a simple linear pseudotime ordering from a single root to a
  single endpoint — Waterfall or Monocle 1-style approaches suffice and CellRouter's
  multi-state machinery is unnecessary.
- The analysis goal is chromatin accessibility, DNA methylation, or mass cytometry
  trajectory reconstruction — the article only envisions these applications as future
  work and does not demonstrate them.
- The goal is generating or functionally validating engraftable HSC-like cells — the
  article explicitly defers thorough characterization of CellRouter-identified genes
  for this purpose to future work.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: 'Core platform: flow-network reconstruction of multi-state trajectories between
    subpopulations, integrating GRNs and regulator scoring'
  repo: https://github.com/edroaldo/cellrouter
- name: igraph
  role: Builds the kNN graph from diffusion components and performs community detection
    to define subpopulations
- name: mclust
  role: Learns the structure of the data prior to community-based subpopulation identification
- name: cccd
  role: Constructs the class-covering k-nearest-neighbor graph from the 3D diffusion
    components
- name: R
  role: Implementation environment; CellRouter is sourced as an R class with R dependencies
    (reshape2, pheatmap, tsne, ggplot2, Rtsne, etc.)
  repo: https://github.com/edroaldo/cellrouter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:4af16daf1bb6936e0dd7ddf575c2cb478f57609e1a098dcd9d78730b0d2d6d2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - cccd
  - igraph
  - mclust
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# flow-network-trajectory-reconstruction

## Summary

Reconstruct high-resolution, multi-state single-cell differentiation trajectories using CellRouter's flow-network algorithm, which is subpopulation-aware and requires no a priori knowledge of the number of branches or cell fates, unlike linear methods (Waterfall, Monocle 1), single-branch methods (Wishbone), or DPT.

## When to use

Use when snapshot single-cell RNA-seq data span multiple cell states (e.g., HSPC differentiation to erythroid, myeloid, and lymphoid lineages) and you need multi-lineage trajectories between user-chosen source and target subpopulations, plus transition-specific genes and regulators.

## When NOT to use

- The analysis goal is a simple linear pseudotime ordering from a single root to a single endpoint — Waterfall or Monocle 1-style approaches suffice and CellRouter's multi-state machinery is unnecessary.
- The analysis goal is chromatin accessibility, DNA methylation, or mass cytometry trajectory reconstruction — the article only envisions these applications as future work and does not demonstrate them.
- The goal is generating or functionally validating engraftable HSC-like cells — the article explicitly defers thorough characterization of CellRouter-identified genes for this purpose to future work.

## Inputs

- Single-cell RNA-seq raw count/UMI matrix (e.g., Paul et al. Cell 2015 mouse myeloid progenitor scRNA-seq data)
- Low-dimensional embedding of cells, e.g., 3D diffusion map coordinates used to build the kNN graph
- Subpopulation metadata or user-defined cluster/sorted-population labels (optional, for trajectory endpoints)

## Outputs

- Subpopulation assignments from mclust + igraph community detection (e.g., 15 subpopulations)
- Multi-state trajectory assignments from source (long-term HSC) to target subpopulations (erythroid, GM, LMPP)
- Lineage-specific marker gene dynamics along each trajectory (e.g., Gata1/Klf1/Gfi1b, Ctsg/Gfi1, Ccl3)
- Transition-specific regulator scores at gene and network level (e.g., Satb1, Irf8, Notch1, Esr1), exportable as cellrouter_trajectories.csv

## How to apply

Load the single-cell count data (raw count/UMI data is supported in the updated version) and low-dimensional embeddings (e.g., 3D diffusion map coordinates) into R, sourcing the CellRouter class with libdir pointed at the CellRouter Java libraries. Build a k-nearest-neighbor graph from the embedding with the cccd/igraph packages, then learn the data structure with mclust and apply igraph community detection to partition cells into subpopulations (e.g., 15 subpopulations on the Paul et al. 2015 myeloid progenitor data). Designate a subpopulation such as long-term HSCs as the source and run CellRouter's flow-network algorithm to reconstruct multi-state trajectories to chosen target subpopulations; the flow-network approach relaxes the requirement of identifying branching points and is independent of dimensionality reduction technique. Score lineage marker dynamics along each trajectory (e.g., Klf1/Gata1/Gfi1b along erythroid, Ctsg/Gfi1 along GM, Ccl3 along LMPP) and rank transition-specific regulators such as Satb1, Irf8, Notch1, and Esr1 by integrating gene regulatory networks with the trajectories. Write trajectory assignments, marker dynamics, and regulator scores to an output table such as cellrouter_trajectories.csv.

## Related tools

- **CellRouter** (Core platform: flow-network reconstruction of multi-state trajectories between subpopulations, integrating GRNs and regulator scoring) — https://github.com/edroaldo/cellrouter
- **igraph** (Builds the kNN graph from diffusion components and performs community detection to define subpopulations)
- **mclust** (Learns the structure of the data prior to community-based subpopulation identification)
- **cccd** (Constructs the class-covering k-nearest-neighbor graph from the 3D diffusion components)
- **R** (Implementation environment; CellRouter is sourced as an R class with R dependencies (reshape2, pheatmap, tsne, ggplot2, Rtsne, etc.)) — https://github.com/edroaldo/cellrouter

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- The expected number of subpopulations is recovered from community detection (e.g., 15 subpopulations on the BloodNet/Paul et al. dataset built from 3D diffusion components).
- Each reconstructed trajectory terminates in the expected mature lineage population (erythroid, granulocyte-monocyte, or LMPP) when long-term HSCs (subpopulation 2) are designated as the source.
- Lineage-specific markers show the expected dynamics along their trajectories: Gata1/Klf1/Gfi1b rising along erythroid, Ctsg/Gfi1 along GM, and Ccl3 along LMPP.
- Regulator ranking reproduces published transition-specific regulators (Satb1, Irf8, Notch1, Esr1 as top lymphoid/LMPP regulators) at both gene and network level.
- Output files (e.g., cellrouter_trajectories.csv) contain per-cell trajectory assignments plus marker dynamics and regulator scores without missing trajectory assignments.

## Limitations

- CellRouter was tested only in Unix-based systems or Mac and was not tested in Windows.
- Thorough functional characterization of genes identified by CellRouter for generating engraftable HSC-like cells is beyond the scope of the manuscript and deferred to future work.
- Application to single-cell chromatin accessibility, DNA methylation, and mass cytometry data is only envisioned as future work, not demonstrated.
- Before the May 2018 update, CellRouter expected preprocessed input; the updated version adds raw count/UMI input, quality control on genes and cells, and normalization steps.

## Evidence

- [abstract] CellRouter, a multifaceted single-cell analysis platform that identifies complex cell-state transition trajectories by using flow networks: "Here, we present CellRouter, a multifaceted single-cell analysis platform that identifies complex cell-state transition trajectories by using flow networks"
- [intro] CellRouter relaxes branching-point requirement and implements flow network algorithm: "CellRouter relaxes the requirement of identifying branching points during cell-fate transitions and implements a flow network algorithm to flexibly reconstruct multi-state transition trajectories"
- [methods] Subpopulations identified by learning structure of data and community detection: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [methods] Source the CellRouter class and set libdir for Java libraries: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [readme] Required R packages for CellRouter installation: "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
- [intro] CellRouter integrates subpopulation identification, multi-state trajectories, and GRNs: "CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions"
