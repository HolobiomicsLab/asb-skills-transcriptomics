---
name: multiomics-cluster-interpretability-annotation
description: Use when scMM latent representations from CITE-seq (transcriptome + surface
  protein) data have been clustered at high resolution and clusters must be characterized/annotated
  with coherent multimodal marker patterns.
when_to_use_negative:
- Input is raw CITE-seq counts that have not yet been encoded by scMM into a joint
  latent representation
- Single-modality data (RNA-only or ATAC-only) where surface-protein marker panels
  cannot be computed
- Aims requiring only coarse cell-type labels rather than high-resolution (~54-cluster)
  interpretable characterization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scMM
  role: Provides the joint transcriptome-protein latent representation and NB-distribution
    sampled surface protein measurements that the clustering and heatmaps annotate
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Environment for downstream analysis of scMM outputs via R/tutorial.R, including
    Rphenograph clustering, umap embedding, and pheatmap rendering
  repo: https://github.com/kodaim1115/scMM
- name: Rphenograph
  role: Builds a k-nearest-neighbor graph over the scMM latent space with Louvain
    community detection to recover the high-resolution cluster partition
- name: umap
  role: Embeds scMM latent variables in 2D for cluster-colored visualization of the
    joint transcriptome-protein space
- name: pheatmap
  role: Renders the side-by-side z-scored transcriptome and surface-protein heatmap
    panels used for cluster annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:bfc90b25d3a026568883e8b9fe4af917b37ff9d2772a145eed3de1abb1dc543a
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
  - Rphenograph
  - pheatmap
  - scMM
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multiomics-cluster-interpretability-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate and interpret high-resolution clusters from scMM's joint transcriptome–surface-protein latent space by building side-by-side z-scored marker heatmaps (pheatmap) of genes and proteins per Rphenograph cluster, as demonstrated on PBMC CITE-seq data (~54 clusters).

## When to use

Use when scMM latent representations from CITE-seq (transcriptome + surface protein) data have been clustered at high resolution and clusters must be characterized/annotated with coherent multimodal marker patterns.

## When NOT to use

- Input is raw CITE-seq counts that have not yet been encoded by scMM into a joint latent representation
- Single-modality data (RNA-only or ATAC-only) where surface-protein marker panels cannot be computed
- Aims requiring only coarse cell-type labels rather than high-resolution (~54-cluster) interpretable characterization

## Inputs

- scMM output latent representation for PBMC CITE-seq cells
- gene.tsv (transcriptome feature list)
- barcode.tsv (single-cell barcodes)
- CITE-seq Protein_count.mtx (surface protein count matrix)

## Outputs

- Rphenograph cluster assignments (~54 clusters) over the scMM latent space
- UMAP embedding of latent variables colored by Rphenograph cluster
- Per-cluster z-scored mean feature matrices for genes and surface proteins
- Side-by-side transcriptome and surface-protein heatmap panels rendered with pheatmap

## How to apply

Load the scMM output directory in R by reading gene.tsv / barcode.tsv and the CITE-seq Protein_count.mtx matrix, following the folder layout documented in the scMM README. Cluster cells in the scMM latent space with Rphenograph (k-nearest-neighbor graph plus Louvain community detection at tutorial-default neighborhood size) to recover the ~54-cluster partition reported in the paper. Embed latent variables with umap in R to visualize the joint transcriptome-protein latent space colored by Rphenograph cluster. Compute per-cluster mean values for transcriptome genes and surface proteins, scale each feature across clusters to z-scores, and select marker features per modality. Render the z-score panels with pheatmap as side-by-side transcriptome and surface-protein heatmap panels so clusters can be annotated from jointly interpretable marker signatures.

## Related tools

- **scMM** (Provides the joint transcriptome-protein latent representation and NB-distribution sampled surface protein measurements that the clustering and heatmaps annotate) — https://github.com/kodaim1115/scMM
- **R** (Environment for downstream analysis of scMM outputs via R/tutorial.R, including Rphenograph clustering, umap embedding, and pheatmap rendering) — https://github.com/kodaim1115/scMM
- **Rphenograph** (Builds a k-nearest-neighbor graph over the scMM latent space with Louvain community detection to recover the high-resolution cluster partition)
- **umap** (Embeds scMM latent variables in 2D for cluster-colored visualization of the joint transcriptome-protein space)
- **pheatmap** (Renders the side-by-side z-scored transcriptome and surface-protein heatmap panels used for cluster annotation)

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- Rphenograph clustering reaches the paper's reported high-resolution regime of approximately 54 clusters on PBMC CITE-seq data
- Heatmap panels show coherent, cluster-specific multimodal marker patterns matching the published figure
- Per-feature z-scores are computed by scaling each gene/protein across the full set of clusters, yielding comparable rows across modalities
- UMAP embedding of latent variables reproduces the joint transcriptome-protein structure colored consistently with Rphenograph cluster assignments
- Cluster annotations are supported by marker features selected independently from each modality (genes and surface proteins)

## Limitations

- Deep generative models such as scMM have limited intrinsic interpretability, which is why the pseudocell generation strategy and downstream heatmap panels are needed
- Interpretability depends on recovering the high-resolution clustering regime; different neighborhood sizes in Rphenograph may not reproduce the ~54-cluster partition
- The downstream R tutorial is designed around the scMM output folder structure (RNA-seq / CITE-seq directories with mtx, gene/protein.tsv, barcode.tsv files); other layouts require adaptation

## Evidence

- [results] Sampling protein measurements for cluster heatmaps: [evidence span withheld — anchor status: not_found]
