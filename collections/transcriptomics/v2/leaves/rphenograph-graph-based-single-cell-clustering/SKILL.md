---
name: rphenograph-graph-based-single-cell-clustering
description: Use when you have scMM latent representations of multimodal single-cell
  data (e.g. CITE-seq RNA + surface protein counts stored as RNA_count.mtx/Protein_count.mtx
  with gene.tsv/barcode.tsv) and need high-resolution, interpretable cluster assignments
  for downstream heatmap visualization.
when_to_use_negative:
- Input is raw count matrices without an scMM-trained latent representation — cluster
  on the latent space, not directly on counts
- Data are single-modality or non-CITE-seq formats whose folder layout (RNA-seq/CITE-seq/ATAC-seq
  with gene.tsv/protein.tsv/peak.tsv) does not match the scMM expected structure without
  reorganization
- The goal is low-level coarse annotation rather than the high-resolution fine-grained
  partition the scMM paper targets
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMM
  role: Provides the joint transcriptome-surface-protein latent representation that
    Rphenograph clusters, and the sampled surface-protein measurements (from NB distributions)
    used in the per-cluster heatmaps
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Runtime for the downstream analysis of scMM outputs (Rphenograph clustering,
    umap embedding, pheatmap rendering) as scripted in R/tutorial.R
  repo: https://github.com/kodaim1115/scMM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:c98a4f829dca9e44db456ce27c2841cc33a97d021cd917a868aea83872f71ab7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kodaim1115/scMM
  repo_url: https://github.com/kodaim1115/scMM
  tools:
  - R
  - scMM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rphenograph-graph-based-single-cell-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters cells in the scMM joint transcriptome–surface-protein latent space using Rphenograph (k-nearest-neighbor graph + Louvain community detection) to recover the paper's high-resolution ~54-cluster partition on PBMC CITE-seq data, enabling cluster-level interpretability.

## When to use

Use when you have scMM latent representations of multimodal single-cell data (e.g. CITE-seq RNA + surface protein counts stored as RNA_count.mtx/Protein_count.mtx with gene.tsv/barcode.tsv) and need high-resolution, interpretable cluster assignments for downstream heatmap visualization.

## When NOT to use

- Input is raw count matrices without an scMM-trained latent representation — cluster on the latent space, not directly on counts
- Data are single-modality or non-CITE-seq formats whose folder layout (RNA-seq/CITE-seq/ATAC-seq with gene.tsv/protein.tsv/peak.tsv) does not match the scMM expected structure without reorganization
- The goal is low-level coarse annotation rather than the high-resolution fine-grained partition the scMM paper targets

## Inputs

- scMM latent-space representation of cells (joint transcriptome-protein embedding)
- scMM output folder: gene.tsv, barcode.tsv, CITE-seq/Protein_count.mtx (and optionally RNA-seq/RNA_count.mtx, protein.tsv)
- Rphenograph neighborhood size parameter (tutorial-default k)

## Outputs

- Per-cell Rphenograph cluster assignments (~54 clusters on PBMC CITE-seq data)
- UMAP embedding of latent variables colored by Rphenograph cluster
- Z-scored per-cluster mean expression matrices for transcriptome genes and surface proteins
- Side-by-side transcriptome and surface-protein heatmap panels rendered with pheatmap

## How to apply

Load the scMM output directory in R, reading gene.tsv / barcode.tsv and the CITE-seq Protein_count.mtx matrix following the folder structure documented in the scMM README. Run Rphenograph on the latent-space embedding (k-nearest-neighbor graph construction followed by Louvain community detection at the tutorial-default neighborhood size) to obtain the fine-grained partition (~54 clusters) reported for the PBMC CITE-seq analysis. Embed the latent variables with umap in R to visualize the joint transcriptome-protein latent space colored by Rphenograph cluster. Then compute per-cluster mean values for transcriptome genes and surface proteins, scale each feature across clusters to z-scores, and render side-by-side transcriptome and surface-protein heatmap panels with pheatmap to demonstrate cluster-specific multimodal marker patterns. The code path for downstream analysis of scMM outputs is the repository's R/tutorial.R script and its vignette.

## Related tools

- **scMM** (Provides the joint transcriptome-surface-protein latent representation that Rphenograph clusters, and the sampled surface-protein measurements (from NB distributions) used in the per-cluster heatmaps) — https://github.com/kodaim1115/scMM
- **R** (Runtime for the downstream analysis of scMM outputs (Rphenograph clustering, umap embedding, pheatmap rendering) as scripted in R/tutorial.R) — https://github.com/kodaim1115/scMM

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- Clustering reaches the paper's reported high-resolution regime of approximately 54 clusters on the PBMC CITE-seq data
- Heatmap panels show coherent, cluster-specific multimodal marker patterns matching the published figure
- Transcriptome and surface-protein panels are each scaled to z-scores across clusters, with marker features selected per modality
- UMAP of latent variables colored by Rphenograph cluster shows spatially contiguous, separated communities
- Per-cluster mean values are computed for both modalities before scaling, and clusters are annotated to demonstrate interpretability

## Limitations

- The scMM tutorial and toy data use randomly subsampled 15,000 CITE-seq BMNC cells (Stuart and Butler et al., 2018) with the 5000 most variable genes; other datasets may require re-tuning the neighborhood size
- Deep generative models like scMM have limited interpretability, which the paper compensates for via pseudocell generation and downstream visualization rather than direct latent-dimension reading
- Louvain community detection on kNN graphs is stochastic and resolution-dependent, so cluster counts can vary between runs unless seeds/parameters are fixed

## Evidence
