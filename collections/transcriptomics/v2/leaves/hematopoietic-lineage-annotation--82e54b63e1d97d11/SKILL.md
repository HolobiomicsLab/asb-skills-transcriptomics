---
name: hematopoietic-lineage-annotation
description: Use when raw mouse bone marrow or HSPC scRNA-seq count data (e.g., GSE76983)
  must be segmented into subpopulations and transitions from HSCs toward mature neutrophil/erythroid
  fates must be annotated with lineage-specific gene dynamics.
when_to_use_negative:
- The analysis goal is a strictly linear pseudotime ordering or a single-branch-point
  trajectory — Waterfall, Monocle 1, or Wishbone target those cases, and CellRouter's
  multi-state flow-network design is unnecessary overhead
- Trajectories must be reconstructed between two pre-specified states using user-defined
  metadata (e.g., sorted populations) rather than de novo community-detected subpopulations
  — CellRouter's updated workflow supports user-defined clusters directly instead
- The data are not single-cell transcriptomics from a hematopoietic context; marker
  dynamics like Cebpe/Mxd1 and Gata1/Scna are lineage-specific to myelo-erythroid
  differentiation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: CellRouter
  role: 'Core platform: subpopulation identification via community detection, flow-network
    trajectory reconstruction, and transition-specific gene dynamics scoring'
  repo: https://github.com/edroaldo/cellrouter
- name: Rtsne
  role: t-SNE embedding of cells following StemID-style dimensionality reduction before
    subpopulation learning
- name: igraph
  role: Community detection on the cccd-based k-nearest-neighbor graph to identify
    subpopulations
- name: cccd
  role: Builds the k-nearest-neighbor graph used for community detection
- name: pheatmap
  role: Annotated heatmap visualization of clustering and gene signatures (listed
    dependency)
- name: ggplot2
  role: Visualization of trajectories and gene expression dynamics (listed dependency)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:6cd524f77624f1022b7f9a11f6857f12f4ed365c2908bd26523e8e69db18c2c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Monocle3
  - R
  - Rtsne
  - SingCellaR
  - cccd
  - ggplot2
  - igraph
  - pheatmap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# hematopoietic-lineage-annotation

## Summary

Annotate hematopoietic subpopulations and lineage trajectories in single-cell RNA-seq data by combining CellRouter's flow-network subpopulation identification with marker-gene dynamics (e.g., Cebpe, Mxd1 for neutrophil; Gata1, Scna for erythroid) along reconstructed transitions from HSC-enriched states.

## When to use

Use when raw mouse bone marrow or HSPC scRNA-seq count data (e.g., GSE76983) must be segmented into subpopulations and transitions from HSCs toward mature neutrophil/erythroid fates must be annotated with lineage-specific gene dynamics.

## When NOT to use

- The analysis goal is a strictly linear pseudotime ordering or a single-branch-point trajectory — Waterfall, Monocle 1, or Wishbone target those cases, and CellRouter's multi-state flow-network design is unnecessary overhead
- Trajectories must be reconstructed between two pre-specified states using user-defined metadata (e.g., sorted populations) rather than de novo community-detected subpopulations — CellRouter's updated workflow supports user-defined clusters directly instead
- The data are not single-cell transcriptomics from a hematopoietic context; marker dynamics like Cebpe/Mxd1 and Gata1/Scna are lineage-specific to myelo-erythroid differentiation

## Inputs

- Raw single-cell RNA-seq count matrix from mouse bone marrow / myeloid progenitors (GEO accession GSE76983, Paul et al. Cell 2015)
- CellRouter class R source file (CellRouter_Class.R) and path to CellRouter Java libraries (libdir)
- Choice of starting subpopulation (HSC-enriched subpopulation 20) and trajectory count (k=10)

## Outputs

- Set of identified subpopulations (24 in the reference run: HSCs, erythroblasts, neutrophils)
- Multi-state transition trajectories from subpopulation 20 to neutrophil and erythroid fates
- Transition-specific gene dynamics table (Cebpe/Mxd1 on neutrophil; Rnf10/Gata1/Scna on erythroid) written to cellrouter_trajectories.csv

## How to apply

Load the Paul et al. (Cell 2015) mouse myeloid progenitor count data (GSE76983) in R, source the CellRouter class, and set 'libdir' to the CellRouter Java library directory. Embed cells with t-SNE (Rtsne) following the StemID-style dimensionality reduction, then learn the data structure and detect subpopulations with CellRouter's community detection (igraph) on the cccd-based k-nearest-neighbor graph (k=5 in the reference task). Reconstruct k=10 high-resolution multi-state trajectories with CellRouter's flow network algorithm starting from the HSC-enriched subpopulation 20, which yields 24 subpopulations spanning HSCs, erythroblasts, and neutrophils. Score transition-specific gene dynamics along each trajectory: on the neutrophil trajectory Cebpe is upregulated early and Mxd1 later; on the erythroid trajectory Gata1 is upregulated earlier than Scna (with Rnf10 also dynamic). Write the trajectory and gene dynamics table to cellrouter_trajectories.csv for downstream validation.

## Related tools

- **CellRouter** (Core platform: subpopulation identification via community detection, flow-network trajectory reconstruction, and transition-specific gene dynamics scoring) — https://github.com/edroaldo/cellrouter
- **Rtsne** (t-SNE embedding of cells following StemID-style dimensionality reduction before subpopulation learning)
- **igraph** (Community detection on the cccd-based k-nearest-neighbor graph to identify subpopulations)
- **cccd** (Builds the k-nearest-neighbor graph used for community detection)
- **pheatmap** (Annotated heatmap visualization of clustering and gene signatures (listed dependency))
- **ggplot2** (Visualization of trajectories and gene expression dynamics (listed dependency))

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'; cr <- CellRouter(expdata=counts); cr <- findsubpopulations(cr, k=5); cr <- findtrajproperties(cr, path.mode='all_paths', pop=20, k=10)
```

## Evaluation signals

- The run identifies 24 subpopulations when trajectories are reconstructed from the HSC-enriched subpopulation 20
- On the neutrophil trajectory, Cebpe is upregulated early and Mxd1 is upregulated later, matching the reported transition-specific dynamics
- On the erythroid trajectory, Gata1 is upregulated earlier than Scna, with Rnf10 also showing expected dynamics
- Subpopulation detection uses k=5 communities on the cccd kNN graph and k=10 high-resolution trajectories, as in the reference workflow
- The trajectory and gene dynamics table is produced as cellrouter_trajectories.csv with entries for both lineage trajectories

## Limitations

- CellRouter was tested in Unix-based systems/Mac and not in Windows; Unix or Mac is strongly recommended
- Thorough functional characterization of genes identified for generating engraftable HSC-like cells is deferred to future work and beyond the manuscript's scope
- The discussion section provides no dataset accession numbers, software versions, or parameter settings by itself; parameters must come from the tutorial workflow
- Application to single-cell chromatin accessibility, DNA methylation, and mass cytometry data is only envisioned, not demonstrated

## Evidence

- [methods] Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [methods] High-resolution multi-state trajectories from any subpopulation: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [methods] Source the CellRouter class and set libdir: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [intro] CellRouter integrates subpopulation identification, multi-state trajectories, and GRNs: "CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions"
- [readme] Analysis of Paul et al. mouse myeloid progenitor data: "Analysis of mouse myeloid progenitor single-cell RNA-seq data generated by Paul et al.(Cell 2015)"
