---
name: single-cell-trajectory-inference-with-flow-networks
description: Use when snapshot single-cell RNA-seq data spans multiple cell states
  (e.g., HSC-to-neutrophil/erythroid differentiation) and you need multi-state, transition-centered
  trajectories between any two chosen subpopulations without knowing branch counts
  a priori.
when_to_use_negative:
- The analysis goal is a single linear pseudotime ordering or one predefined branch
  point — linear tools (Waterfall, Monocle 1) or Wishbone may suffice and CellRouter's
  flow-network machinery is unnecessary overhead.
- Data are single-cell chromatin accessibility, DNA methylation, or mass cytometry
  — the article only envisions these applications as future work and does not demonstrate
  them.
- You need validated functional enrichment or functional characterization of genes
  along trajectories for engineering engraftable cells — the manuscript explicitly
  defers this beyond its scope.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: 'Core platform: flow-network reconstruction of multi-state transition trajectories
    between any subpopulations'
  repo: https://github.com/edroaldo/cellrouter
- name: Rtsne
  role: t-SNE embedding following StemID-style dimensionality reduction prior to subpopulation
    learning
- name: igraph
  role: Community detection to identify subpopulations on the k-nearest-neighbor graph
- name: cccd
  role: Builds the k-nearest-neighbor graph on which community detection runs
- name: pheatmap
  role: Annotated heatmaps of gene signatures per cluster
- name: ggplot2
  role: Visualization of trajectories and gene dynamics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:0bdb8413ff16f49944575606ebaa21c50209cd93c04c024f52ac880d09f4345d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - Rtsne
  - cccd
  - ggplot2
  - igraph
  - pheatmap
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# single-cell-trajectory-inference-with-flow-networks

## Summary

Reconstruct multi-state, branched single-cell trajectories by applying CellRouter's flow network algorithm to subpopulation structure identified on a k-nearest-neighbor graph (cccd/igraph), then score transition-specific gene dynamics along each trajectory.

## When to use

Use when snapshot single-cell RNA-seq data spans multiple cell states (e.g., HSC-to-neutrophil/erythroid differentiation) and you need multi-state, transition-centered trajectories between any two chosen subpopulations without knowing branch counts a priori.

## When NOT to use

- The analysis goal is a single linear pseudotime ordering or one predefined branch point — linear tools (Waterfall, Monocle 1) or Wishbone may suffice and CellRouter's flow-network machinery is unnecessary overhead.
- Data are single-cell chromatin accessibility, DNA methylation, or mass cytometry — the article only envisions these applications as future work and does not demonstrate them.
- You need validated functional enrichment or functional characterization of genes along trajectories for engineering engraftable cells — the manuscript explicitly defers this beyond its scope.

## Inputs

- Raw single-cell RNA-seq count/UMI matrix (e.g., GSE76983 mouse myeloid progenitor counts)
- Cell metadata or user-defined cluster labels (optional; enhanced flexibility allows sorted populations)
- libdir path to CellRouter Java libraries

## Outputs

- Subpopulation assignments and community-detected cluster structure
- Multi-state trajectories between subpopulations with divergence of single-cell transcriptomes along progression
- Transition-specific gene expression dynamics table (e.g., cellrouter_trajectories.csv)

## How to apply

Load raw count data (e.g., Paul et al. GSE76983 mouse myeloid progenitors) and source the CellRouter class in R, setting 'libdir' to the Java library directory. Perform StemID-style dimensionality reduction with t-SNE (Rtsne), then identify subpopulations (e.g., k=5 clusters resolving 24 subpopulations) via community detection (igraph) on the cccd-based k-nearest-neighbor graph. Reconstruct high-resolution multi-state trajectories with CellRouter's flow network algorithm from a chosen start subpopulation (e.g., subpopulation 20, the HSC-enriched fraction). Finally, score transition-specific gene dynamics along each trajectory and export results (e.g., cellrouter_trajectories.csv).

## Related tools

- **CellRouter** (Core platform: flow-network reconstruction of multi-state transition trajectories between any subpopulations) — https://github.com/edroaldo/cellrouter
- **Rtsne** (t-SNE embedding following StemID-style dimensionality reduction prior to subpopulation learning)
- **igraph** (Community detection to identify subpopulations on the k-nearest-neighbor graph)
- **cccd** (Builds the k-nearest-neighbor graph on which community detection runs)
- **pheatmap** (Annotated heatmaps of gene signatures per cluster)
- **ggplot2** (Visualization of trajectories and gene dynamics)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- Confirm the expected number of subpopulations is recovered (e.g., 24 subpopulations for the Paul et al. GSE76983 myeloid data).
- Verify trajectories originate from the chosen start state (e.g., subpopulation 20, HSC-enriched) and include the expected lineage endpoints (neutrophils, erythroblasts).
- Check transition-specific dynamics: Cebpe upregulated early and Mxd1 late along the neutrophil trajectory; Gata1 (and Rnf10/Scna) upregulated earlier than Scna along the erythroid trajectory.
- Confirm the trajectory and gene dynamics table is written to cellrouter_trajectories.csv with populated rows for each trajectory.
- Confirm CellRouter runs require the 'libdir' variable pointing to the CellRouter Java library directory; missing libdir causes failure.

## Limitations

- CellRouter was tested only in Unix-based systems/Mac and not in Windows; the README strongly recommends Unix.
- Functional characterization of genes identified for generating engraftable HSC-like cells is beyond the manuscript scope.
- Application to chromatin accessibility, DNA methylation, and mass cytometry data is envisioned but not demonstrated.
- Requires several R packages (reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd) plus Vennerable from R-Forge, which may complicate installation.

## Evidence

- [abstract] flow network trajectories: "identifies complex cell-state transition trajectories by using flow networks to explore the subpopulation structure of multi-dimensional, single-cell omics data"
- [intro] no a priori branch knowledge: "CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches"
- [methods] subpopulation identification by community detection: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [methods] multi-state trajectories between any subpopulations: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [readme] libdir setup: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [readme] input flexibility and QC: "Raw count/UMI data can be provided as input. 2. Perform quality control on genes and cells 3. Normalize the data"
