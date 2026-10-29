---
name: scmm-latent-space-umap-visualization
description: Use when you have a trained scMM latent representation from paired transcriptome
  & surface protein (CITE-seq) counts and need to visualize the joint space and its
  high-resolution cluster partition in 2D.
when_to_use_negative:
- Input is raw count data that has not been encoded by scMM — the UMAP should be computed
  on the learned joint latent representation, not directly on RNA/protein counts
- Only a single modality is available (e.g. scRNA-seq alone); scMM's joint latent
  space requires paired transcriptome & surface protein (or chromatin accessibility)
  data
- Goal is coarse cell-type annotation rather than the paper's high-resolution (~54
  cluster) regime; Rphenograph at tutorial-default neighborhood size will over-partition
  simple datasets
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scMM
  role: Produces the joint transcriptome-protein latent representation that is embedded
    with UMAP
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: 'Hosts the downstream analysis: Rphenograph clustering, umap embedding, and
    pheatmap rendering of scMM outputs'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:42fa940e3444b3bf501a91791aa2b51874559114b4d938012763ed11a582eecb
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# scMM latent-space UMAP visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embeds scMM's joint transcriptome-protein latent representation with UMAP in R and colors cells by Rphenograph cluster, enabling visual assessment of high-resolution multimodal clustering on CITE-seq data such as the PBMC analysis.

## When to use

Use when you have a trained scMM latent representation from paired transcriptome & surface protein (CITE-seq) counts and need to visualize the joint space and its high-resolution cluster partition in 2D.

## When NOT to use

- Input is raw count data that has not been encoded by scMM — the UMAP should be computed on the learned joint latent representation, not directly on RNA/protein counts
- Only a single modality is available (e.g. scRNA-seq alone); scMM's joint latent space requires paired transcriptome & surface protein (or chromatin accessibility) data
- Goal is coarse cell-type annotation rather than the paper's high-resolution (~54 cluster) regime; Rphenograph at tutorial-default neighborhood size will over-partition simple datasets

## Inputs

- scMM output directory with joint transcriptome-protein latent representations
- CITE-seq Protein_count.mtx with protein.tsv and barcode.tsv
- Transcriptome RNA_count.mtx with gene.tsv and barcode.tsv
- Rphenograph cluster assignments per cell

## Outputs

- 2D UMAP embedding of the scMM latent space colored by Rphenograph cluster
- Cluster-annotated UMAP figure for interpretability (optionally paired pheatmap z-score panels)

## How to apply

First cluster cells in the scMM latent space with Rphenograph (k-nearest-neighbor graph plus Louvain community detection at tutorial-default neighborhood size) to recover the high-resolution partition (~54 clusters in the PBMC CITE-seq analysis). Then embed the latent variables with the umap R package to visualize the joint transcriptome-protein latent space, coloring each cell by its Rphenograph cluster assignment. Load the scMM output directory in R, reading gene.tsv / barcode.tsv and the CITE-seq Protein_count.mtx matrix to align cells across modalities. Optionally complement the UMAP with per-cluster z-scored heatmaps (pheatmap) of transcriptome and surface-protein markers to confirm that UMAP clusters correspond to coherent multimodal marker patterns. Validation: the UMAP should resolve ~54 distinct clusters matching the paper's reported regime and published figure.

## Related tools

- **scMM** (Produces the joint transcriptome-protein latent representation that is embedded with UMAP) — https://github.com/kodaim1115/scMM
- **R** (Hosts the downstream analysis: Rphenograph clustering, umap embedding, and pheatmap rendering of scMM outputs)

## Examples

```
source("R/tutorial.R")  # Rphenograph on scMM latent space, then umap embedding colored by cluster, per R/tutorial.R
```

## Evaluation signals

- Rphenograph on the scMM latent space recovers the paper's high-resolution regime (~54 clusters) on PBMC CITE-seq data
- UMAP plot shows spatially separated clusters, each colored by a distinct Rphenograph community
- Per-cluster z-scored heatmap panels (pheatmap) show coherent, cluster-specific multimodal marker patterns matching the published figure
- Barcodes in gene.tsv / barcode.tsv and Protein_count.mtx align one-to-one across modalities before embedding

## Limitations

- The tutorial's downstream R analysis (`R/tutorial.R`) is demonstrated on toy BMNC CITE-seq data with 15,000 randomly subsampled cells and the most variable 5,000 genes; other datasets may need tuning
- Number of Rphenograph clusters depends on neighborhood size; the ~54-cluster partition is specific to the PBMC CITE-seq analysis settings
- Deep-learning latent representations have limited interpretability on their own; scMM compensates via pseudocell generation, so the UMAP alone does not explain regulatory programs

## Evidence

- [results] Load the scMM output directory for the PBMC analysis using R, reading gene.tsv / barcode.tsv and the CITE-seq Protein_count.mtx matrix: [evidence span withheld — anchor status: not_found]
- [results] Embed the latent variables with umap in R to visualize the joint transcriptome-protein latent space colored by Rphenograph cluster: [evidence span withheld — anchor status: not_found]
