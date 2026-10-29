---
name: multi-method-integration-comparison-analysis
description: Use when a batch-corrected single-cell embedding must be quantitatively
  compared to other integration methods on cell type conservation (higher silhouette
  better) and batch mixing (lower silhouette better), e.g. replicating a Fig. 3d-style
  benchmark table.
when_to_use_negative:
- The dataset lacks expert-annotated cell type labels — silhouette-with-respect-to-cell-type
  is undefined without them
- A method's embedding mixes biological and batch factors such that silhouette comparison
  to supervised scANVI would be misleading without careful interpretation (scANVI
  was fully supervised on cell type labels for this benchmark)
- The goal is batch-confounded cell type discovery rather than integration benchmarking
  — use the CODAL disentangling workflow instead of a comparison table
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: silhouette_score computation of per-cell silhouette widths against cell type
    and joint batch+cell type labels
- name: scanpy
  role: preprocessing (normalize_total target_sum 10000, log+1, highly variable gene
    selection) and PCA baseline embedding
- name: CODAL
  role: method under evaluation; variational autoencoder with mutual information regularization
    disentangling technical and biological effects
  repo: https://github.com/cistrome/MIRA
- name: scVI
  role: baseline VAE-based integration embedding
- name: scANVI
  role: semi-supervised VAE baseline; fully supervised on cell type labels for this
    benchmark
- name: Harmony
  role: baseline applied to scanpy PCA embedding; removes technical effects via cross-dataset
    fuzzy clustering
- name: Scanorama
  role: baseline integration embedding
- name: MIRA
  role: baseline topic-modeling embedding; CODAL's architecture extends MIRA and all
    MIRA topic models use CODAL inference
  repo: https://github.com/cistrome/MIRA
- name: Seurat v3
  role: baseline integration embedding
- name: FastMNN
  role: baseline integration embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:98772c14b5183e37b39464d3a64cf305fbec23cc1726b29e9b3b21d796ab757f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# multi-method-integration-comparison-analysis

## Summary

Benchmark a batch-corrected latent space against eight alternative integration methods (CODAL, scVI, scANVI, Harmony, Scanorama, MIRA, PCA, Seurat v3/FastMNN) by computing cell type and batch average silhouette widths (ASW) with scikit-learn on expert-annotated single-cell data.

## When to use

Use when a batch-corrected single-cell embedding must be quantitatively compared to other integration methods on cell type conservation (higher silhouette better) and batch mixing (lower silhouette better), e.g. replicating a Fig. 3d-style benchmark table.

## When NOT to use

- The dataset lacks expert-annotated cell type labels — silhouette-with-respect-to-cell-type is undefined without them
- A method's embedding mixes biological and batch factors such that silhouette comparison to supervised scANVI would be misleading without careful interpretation (scANVI was fully supervised on cell type labels for this benchmark)
- The goal is batch-confounded cell type discovery rather than integration benchmarking — use the CODAL disentangling workflow instead of a comparison table

## Inputs

- Preprocessed NEURIPS bone marrow gene expression data (cells with <400 counts and genes with <30 counts filtered, normalized to target_sum 10000, log+1 transformed, 3500 highly variable genes)
- Expert-annotated cell type labels per cell
- Batch (site/donor) covariate labels per cell
- Latent space embeddings from each integration method: CODAL, scVI, scANVI, scanpy PCA+Harmony, Scanorama, MIRA, PCA, Seurat v3/FastMNN

## Outputs

- Per-method cell type ASW and batch ASW values compiled into asw_benchmark_table.csv
- Benchmark table comparing CODAL against eight baseline methods (cell type silhouette higher = better; batch silhouette lower = better)

## How to apply

Load the preprocessed dataset (cells <400 counts and genes <30 counts filtered, normalize_total to target_sum 10000, log+1 transformed, 3500 highly variable genes) with expert-annotated cell type labels and batch (site/donor) covariates. Load each method's latent embeddings — CODAL, scVI, scANVI, scanpy PCA+Harmony, Scanorama, MIRA, PCA, and Seurat v3/FastMNN. For each embedding, compute per-cell silhouette widths with scikit-learn's silhouette_score twice: once with respect to the expert cell type labels and once with respect to the joint batch+cell type label, where lower scores indicate better batch intermixing. Average silhouette widths across all cells per metric per method, compile into a benchmark table (asw_benchmark_table.csv), and interpret: cell type ASW higher is better, batch ASW lower is better; CODAL should be comparable to scANVI, which was fully supervised on cell type labels.

## Related tools

- **scikit-learn** (silhouette_score computation of per-cell silhouette widths against cell type and joint batch+cell type labels)
- **scanpy** (preprocessing (normalize_total target_sum 10000, log+1, highly variable gene selection) and PCA baseline embedding)
- **CODAL** (method under evaluation; variational autoencoder with mutual information regularization disentangling technical and biological effects) — https://github.com/cistrome/MIRA
- **scVI** (baseline VAE-based integration embedding)
- **scANVI** (semi-supervised VAE baseline; fully supervised on cell type labels for this benchmark)
- **Harmony** (baseline applied to scanpy PCA embedding; removes technical effects via cross-dataset fuzzy clustering)
- **Scanorama** (baseline integration embedding)
- **MIRA** (baseline topic-modeling embedding; CODAL's architecture extends MIRA and all MIRA topic models use CODAL inference) — https://github.com/cistrome/MIRA
- **Seurat v3** (baseline integration embedding)
- **FastMNN** (baseline integration embedding)

## Examples

```
from sklearn.metrics import silhouette_score; asw_ct = silhouette_score(embedding, celltype_labels); asw_batch = silhouette_score(embedding, batch_labels)
```

## Evaluation signals

- asw_benchmark_table.csv contains both silhouette metrics for all nine listed methods
- Cell type ASW direction check: higher is better (better cell type colocalization); batch ASW direction check: lower is better (better batch intermixing)
- CODAL's cell type and batch ASW scores are comparable to fully supervised scANVI, as reported in Fig. 3d
- Per-cell silhouette widths are averaged across all cells separately for the cell type and batch metrics — no pooling of the two metrics

## Limitations

- New batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out under-correction of technical effects
- scANVI was fully supervised on cell type labels for this benchmark, so its silhouette scores are a favorable upper comparison for unsupervised methods like CODAL
- CODAL does not remove the need for good experimental design and appropriate controls; experimentalists must include appropriate controls
- Further improvements might be achieved through better characterization of the technical sources of bias in conjunction with the CODAL model

## Evidence

- [results] CODAL achieved batch mixing and cell type colocalization comparable to scANVI, which previously showed best-in-class atlas-level integration and was fully supervised on cell type labels for this benchmark.: [evidence span withheld — anchor status: not_found]
- [methods] cells with fewer than 400 counts and genes with fewer than 30 counts filtered out: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [methods] cell read depths normalized using the normalize_total function with target_sum set to 10000: "cell read depths normalized using the normalize_total function with target_sum set to 10000"
- [methods] highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes: "highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes"
- [intro] This includes VAE-based scVI20, which has been shown to be one of the most effective approaches in single-cell RNA-seq atlas benchmarking tests21.: "This includes VAE-based scVI20, which has been shown to be one of the most effective approaches in single-cell RNA-seq atlas benchmarking tests21."
- [intro] CODAL is available as an open-source Python package at https://mira-multiome.readthedocs.io.: "CODAL is available as an open-source Python package at https://mira-multiome.readthedocs.io."
- [readme] All MIRA topic models use the new CODAL algorithm for inference.: [evidence span withheld — anchor status: external_doc]
