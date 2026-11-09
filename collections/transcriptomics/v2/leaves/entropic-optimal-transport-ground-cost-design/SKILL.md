---
name: entropic-optimal-transport-ground-cost-design
description: Use when training Mowgli on paired single-cell multi-omics data (RNA,
  ADT, ATAC) and you need to configure the OT loss — ground-cost metric, entropic
  regularization, sparsity weights, and latent dimension k — or extend it, e.g. with
  batch-aware cost reweighting.
when_to_use_negative:
- The data are unpaired multi-omics or single-omics only — Mowgli is designed for
  paired multi-omics integration with any type and number of omics, so an embedding
  like Seurat's, which the silhouette score cannot even be computed on, calls for
  a different method
- The dataset is very large and no GPU is available — the paper notes GPU availability
  is required for Mowgli to scale because entropic OT is inherently expensive to compute
- A deterministic k is required without experimentation — the paper states Mowgli
  does not offer a straightforward approach to defining the number of latent dimensions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Mowgli
  role: Core Python package implementing integrative NMF + entropic OT (Sinkhorn)
    training; installed via pip install mowgli
  repo: github.com/cantinilab/mowgli
- name: Muon
  role: 'Preprocessing of the Muon object: filter_var QC, TF-IDF for ATAC, CLR for
    ADT'
- name: Scanpy
  role: normalize_total, log1p, highly_variable_genes feature selection, and downstream
    clustering/UMAP on the Mowgli embedding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:1699ac3e465cbfff523cfbea941950c8893301a5f51e3baf938d37a8f0191032
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mowgli
  - Muon
  - Python
  - Scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Entropic optimal transport ground-cost design for paired multi-omics integration

## Summary

Designs the optimal transport component of Mowgli's integrative NMF objective: each omics matrix is decomposed into shared cell embeddings W and modality-specific H factors, with an entropically regularized OT term computed via the GPU-enabled Sinkhorn algorithm on a cosine ground cost between cells. This is what distinguishes Mowgli from plain integrative NMF and improves clustering and interpretability.

## When to use

Use when training Mowgli on paired single-cell multi-omics data (RNA, ADT, ATAC) and you need to configure the OT loss — ground-cost metric, entropic regularization, sparsity weights, and latent dimension k — or extend it, e.g. with batch-aware cost reweighting.

## When NOT to use

- The data are unpaired multi-omics or single-omics only — Mowgli is designed for paired multi-omics integration with any type and number of omics, so an embedding like Seurat's, which the silhouette score cannot even be computed on, calls for a different method
- The dataset is very large and no GPU is available — the paper notes GPU availability is required for Mowgli to scale because entropic OT is inherently expensive to compute
- A deterministic k is required without experimentation — the paper states Mowgli does not offer a straightforward approach to defining the number of latent dimensions

## Inputs

- Paired multi-omics Muon/AnnData object (scRNA, scADT/CITE, scATAC modalities) with per-cell metadata (e.g. batch labels, cell-type annotations)
- Preprocessed feature matrices per modality (normalized, log1p, TF-IDF or CLR, top 2500/1500 HVGs selected)
- Hyperparameter configuration: number of latent dimensions k, sparsity parameters rho per modality and mu, ground-cost metric (cosine)

## Outputs

- Shared low-dimensional cell embedding W (k latent dimensions, e.g. 50)
- Modality-specific factor matrices H(p) for each omics
- Mowgli factor loadings usable for clustering, UMAP visualization, GSEA (top 150 genes per factor) and motif enrichment (top 100 peaks per factor)

## How to apply

Load paired data as a Muon object and preprocess per modality (QC filtering with Muon's filter_var, Scanpy normalize_total to 10,000 counts plus log1p, TF-IDF for ATAC, Center Log Ratio for ADT, top 2500 highly variable genes or 1500 for the Liu dataset). Build the cosine ground cost over cells and compute the entropic regularization of OT with the fast, GPU-enabled Sinkhorn algorithm — needed because OT is inherently expensive compared to Euclidean distance. Train by minimizing the reconstruction-plus-OT objective alternating between H(p) and W with L-BFGS, using sparsity parameters such as rho_rna=0.01, rho_adt=0.01, rho_atac=0.1 and mu=0.001, and the overall best-performing number of latent dimensions k (the paper notes k must be tested empirically). Mowgli combines integrative NMF with OT, which the paper shows enhances both clustering performance and factor interpretability. As a future extension, the discussion flags batch correction of the OT objective — e.g. batch-aware ground-cost reweighting — as evaluable with the existing silhouette score, ARI, and kNN purity metric framework.

## Related tools

- **Mowgli** (Core Python package implementing integrative NMF + entropic OT (Sinkhorn) training; installed via pip install mowgli) — github.com/cantinilab/mowgli
- **Muon** (Preprocessing of the Muon object: filter_var QC, TF-IDF for ATAC, CLR for ADT)
- **Scanpy** (normalize_total, log1p, highly_variable_genes feature selection, and downstream clustering/UMAP on the Mowgli embedding)

## Examples

```
pip install mowgli
```

## Evaluation signals

- The objective converges under alternating L-BFGS minimization on H(p) and W without divergence or negative entries in the nonnegative factors
- Embedding quality metrics (silhouette score, Adjusted Rand Index across Leiden resolutions, kNN purity score) for Mowgli are competitive with or better than Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate on the same dataset
- Learned factors show cell-type specificity: mean weight within each annotated cell type exceeds weight outside it
- If batch correction is added, corrected vs. uncorrected embeddings are compared on the same silhouette/ARI/purity metric set (batch_correction_comparison.csv)
- Sinkhorn converges (entropic regularization keeps the OT problem solvable by the GPU-enabled Sinkhorn algorithm at the chosen regularization strength)

## Limitations

- Mowgli does not handle batch correction when integrating paired multi-omics data; the discussion flags this as future work
- Optimal transport is inherently expensive to compute compared to the Euclidean distance; GPU availability is required for Mowgli to scale to the larger datasets presented
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values of k should be tested

## Evidence

- [abstract] Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing at the same time the clustering performance and interpretability of integrative Nonnegative Matrix: "Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing at the same time the clustering performance and interpretability of integrative Nonnegative Matrix"
- [methods] The entropic regularization of OT, described in (Eq.4) below is computed using the fast and GPU-enabled Sinkhorn algorithm: [evidence span withheld — anchor status: not_found]
- [methods] we alternate between minimizing (Eq. 5) on H(p) and W: "we alternate between minimizing (Eq. 5) on H p ð Þ and W"
- [discussion] it would be interesting to deal with batch correction once integrating paired multi-omics data: "it would be interest-
ingtodealwithbatchcorrectiononceintegratingpairedmulti-
omics data"
- [readme] Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy"
