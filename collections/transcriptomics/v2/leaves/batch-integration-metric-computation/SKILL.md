---
name: batch-integration-metric-computation
description: Use when you have latent embeddings from one or more single-cell batch-integration
  methods plus per-cell batch and cell type labels, and need quantitative ASW scores
  to rank integration quality.
when_to_use_negative:
- The embeddings have not yet been generated — silhouette scores quantify an existing
  latent space and cannot be computed from raw counts alone
- You need scRNA-seq/scATAC-seq peak calling rather than integration benchmarking
  (e.g. MACS3 is a ChIP-seq peak caller, not a batch-correction scorer)
- Only a single batch with no batch covariate exists — batch ASW against the joint
  batch-and-cell-type label is undefined in a meaningful way
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Provides silhouette_score used to compute per-cell silhouette widths for cell
    type and joint batch-cell type labels
- name: scanpy
  role: Preprocessing workflow (filtering, normalize_total, highly variable gene selection)
    and PCA baseline embeddings
- name: CODAL
  role: Batch-correction method whose latent space is benchmarked; uses mutual information
    regularization to disentangle technical and biological effects
  repo: github.com/cistrome/MIRA
- name: MIRA
  role: Topic-modeling framework whose architecture CODAL extends; provides a baseline
    embedding
  repo: https://github.com/cistrome/MIRA
- name: scVI
  role: VAE-based batch-integration baseline embedding
- name: scANVI
  role: Semi-supervised VAE baseline; fully supervised on cell type labels for this
    benchmark and best-in-class comparator
- name: Harmony
  role: Baseline producing scanpy PCA+Harmony embeddings for ASW comparison
- name: Scanorama
  role: Baseline batch-integration embedding
- name: Seurat v3
  role: Baseline integration embedding
- name: FastMNN
  role: Baseline integration embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:422f4b47456977421a5b89bed206faa48d5ebbf2ed15fcc7683a27f1015ce6d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - FastMNN
  - Harmony
  - MIRA
  - Scanorama
  - Seurat
  - Seurat v3
  - scANVI
  - scVI
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# batch-integration-metric-computation

## Summary

Computes average silhouette width (ASW) metrics for cell type conservation and batch mixing to benchmark latent-space embeddings from batch-correction methods such as CODAL against baselines like scVI, scANVI, Harmony, and Scanorama on multi-batch single-cell atlases.

## When to use

Use when you have latent embeddings from one or more single-cell batch-integration methods plus per-cell batch and cell type labels, and need quantitative ASW scores to rank integration quality.

## When NOT to use

- The embeddings have not yet been generated — silhouette scores quantify an existing latent space and cannot be computed from raw counts alone
- You need scRNA-seq/scATAC-seq peak calling rather than integration benchmarking (e.g. MACS3 is a ChIP-seq peak caller, not a batch-correction scorer)
- Only a single batch with no batch covariate exists — batch ASW against the joint batch-and-cell-type label is undefined in a meaningful way

## Inputs

- Latent-space embedding matrices (one per integration method: CODAL, scVI, scANVI, PCA+Harmony, Scanorama, MIRA, PCA, Seurat v3/FastMNN)
- Expert-annotated cell type labels per cell
- Batch (site/donor) covariate labels per cell
- Preprocessed NEURIPS bone marrow gene expression data (filtered <400 counts/cell and <30 counts/gene, normalize_total target_sum=10000, log+1, 3500 highly variable genes)

## Outputs

- Per-method cell type ASW (higher is better) and batch ASW (lower is better)
- Benchmark table (asw_benchmark_table.csv) with both silhouette metrics for all compared methods

## How to apply

Load the preprocessed dataset (for the NEURIPS bone marrow benchmark: cells with fewer than 400 counts and genes with fewer than 30 counts filtered, normalized to target_sum 10000, log+1 transformed, 3500 highly variable genes) along with expert-annotated cell type labels and batch (site/donor) covariates. For each method's latent embedding (CODAL, scVI, scANVI, scanpy PCA+Harmony, Scanorama, MIRA, PCA, Seurat v3/FastMNN), use scikit-learn's silhouette_score to compute per-cell silhouette widths against the cell type labels, where higher scores indicate better cell type colocalization. Separately compute silhouette widths against the joint batch-and-cell-type label so that lower scores indicate better intermixing of batches. Average silhouette widths across all cells for each metric per method, then compile both metrics into a benchmark table (e.g. asw_benchmark_table.csv) comparing CODAL against the baselines.

## Related tools

- **scikit-learn** (Provides silhouette_score used to compute per-cell silhouette widths for cell type and joint batch-cell type labels)
- **scanpy** (Preprocessing workflow (filtering, normalize_total, highly variable gene selection) and PCA baseline embeddings)
- **CODAL** (Batch-correction method whose latent space is benchmarked; uses mutual information regularization to disentangle technical and biological effects) — github.com/cistrome/MIRA
- **MIRA** (Topic-modeling framework whose architecture CODAL extends; provides a baseline embedding) — https://github.com/cistrome/MIRA
- **scVI** (VAE-based batch-integration baseline embedding)
- **scANVI** (Semi-supervised VAE baseline; fully supervised on cell type labels for this benchmark and best-in-class comparator)
- **Harmony** (Baseline producing scanpy PCA+Harmony embeddings for ASW comparison)
- **Scanorama** (Baseline batch-integration embedding)
- **Seurat v3** (Baseline integration embedding)
- **FastMNN** (Baseline integration embedding)

## Examples

```
from sklearn.metrics import silhouette_score; asw_ct = silhouette_score(latent_embedding, cell_type_labels); asw_batch = silhouette_score(latent_embedding, batch_celltype_joint_labels)
```

## Evaluation signals

- asw_benchmark_table.csv contains both cell type ASW and batch ASW rows/columns for all listed methods (CODAL, scVI, scANVI, PCA+Harmony, Scanorama, MIRA, PCA, Seurat v3/FastMNN)
- Cell type ASW is oriented so higher is better; batch ASW is oriented so lower is better — check the sign/orientation of computed scores
- CODAL's batch mixing and cell type colocalization scores should be comparable to scANVI's, consistent with Fig. 3d of the paper
- Each silhouette score is an average across all cells, so verify the per-cell silhouette width vector length equals the number of cells in the embedding
- Embeddings are computed on the same preprocessed feature space (3500 highly variable genes) across methods so scores are comparable

## Limitations

- ASW scores require trustworthy cell type annotations; CODAL's authors caution that new batch-confounded or perturbation-induced cell states should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- Batch ASW of the joint batch-and-cell-type label conflates biological and technical separation, so it must be interpreted with the correct orientation (lower is better)
- Authors note further improvements may require better characterization of technical sources of bias, and that computational correction does not remove the need for good experimental design and appropriate controls

## Evidence

- [methods] cells with fewer than 400 counts and genes with fewer than 30 counts filtered out: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [methods] highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes: "highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes"
- [abstract] Here we propose CODAL, a variational autoencoder-based statistical model which uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological: "Here we propose CODAL, a variational autoencoder-based statistical model which uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological"
- [readme] All MIRA topic models use the new CODAL algorithm for inference.: [evidence span withheld — anchor status: external_doc]
