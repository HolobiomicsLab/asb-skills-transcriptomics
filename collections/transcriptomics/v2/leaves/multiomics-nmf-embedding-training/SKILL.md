---
name: multiomics-nmf-embedding-training
description: Use when you have paired single-cell multi-omics data (e.g., a Muon object
  with RNA, ATAC and/or protein modalities) and need a shared low-dimensional cell
  embedding with interpretable factors, as in the Mowgli workflow.
when_to_use_negative:
- Unpaired or non-paired single-cell data where cell-level correspondence between
  modalities is unavailable
- When the number of latent dimensions k must be set automatically — Mowgli does not
  offer a straightforward approach to defining k, so avoid expecting auto-selection
- Datasets that require batch correction across centers — Mowgli does not handle batch
  correction when integrating paired multi-omics data
tools:
- name: Mowgli
  role: Core Python package that implements integrative NMF + entropic OT training
    and exposes model.train(mdata) within the scverse ecosystem
  repo: https://github.com/cantinilab/Mowgli
- name: PyTorch
  role: Backend used to implement the fast, GPU-enabled Sinkhorn algorithm and L-BFGS
    optimization of the dual problems
- name: Muon
  role: Provides the Muon object container for paired multi-omics data and per-modality
    preprocessing utilities (filter_var, tf_idf, clr)
- name: Scanpy
  role: Used for preprocessing (normalize_total, log1p, highly_variable_genes) and
    downstream analysis of the trained embedding
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:eac429ef7bb84f6cd6c561043a844d0ffad365140474ed0ddafe2e8dd936029d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/Mowgli
  tools:
  - Mowgli
  - Muon
  - PyTorch
  - Python
  - Scanpy
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multiomics-nmf-embedding-training

## Summary

Train the Mowgli model, which decomposes paired multi-omics matrices (scRNA, scATAC, ADT) via integrative Nonnegative Matrix Factorization combined with entropic-regularized Optimal Transport to yield modality-specific dictionaries H(p) and a shared cell embedding W. This skill covers building the cosine ground cost, running Sinkhorn-based OT, and alternating L-BFGS optimization on the dual problems.

## When to use

Use when you have paired single-cell multi-omics data (e.g., a Muon object with RNA, ATAC and/or protein modalities) and need a shared low-dimensional cell embedding with interpretable factors, as in the Mowgli workflow.

## When NOT to use

- Unpaired or non-paired single-cell data where cell-level correspondence between modalities is unavailable
- When the number of latent dimensions k must be set automatically — Mowgli does not offer a straightforward approach to defining k, so avoid expecting auto-selection
- Datasets that require batch correction across centers — Mowgli does not handle batch correction when integrating paired multi-omics data

## Inputs

- Muon object (.h5mu) with paired single-cell multi-omics modalities (scRNA-seq, scATAC-seq, ADT/CITE-seq)
- Preprocessed feature matrices per modality (QC-filtered, normalized, log1p/TF-IDF/CLR, top 2500 (1500 for Liu) highly variable genes)
- Number of latent dimensions k (e.g., 15 for the 10X Multiome demo dataset)

## Outputs

- Shared cell embedding W with shape n_cells × k, stored in mdata.obsm['W_OT'] with rows respecting the simplex constraint
- Modality-specific dictionaries H(p) stored in mdata.uns with simplex-constrained columns
- Trained model checkpoint (mowgli_model.pt)

## How to apply

Load the paired data as a Muon object (e.g., mu.load_h5mu('my_data.h5mu')) after per-modality preprocessing (QC filtering, Scanpy normalize_total to 10000 and log1p, TF-IDF for ATAC, CLR for ADT, top 2500/1500 highly variable features). For each modality, build the data-driven ground cost matrix as pairwise cosine distances between feature rows (C_kl = 1 - u_k·u_l/(||u_k|| ||u_l||), Eq. 3), then compute the entropic-regularized OT objective (Eq. 4) with the fast GPU-enabled Sinkhorn algorithm in PyTorch. Minimize the loss (Eq. 5) by alternating between the smooth dual problems for H(p) (Eqs. 7-9, column-wise softmax of -G(p)W/ρ_p) and W (Eqs. 10-11, softmax of (1/(dμ)) Σ_p H(p)' G(p)), each solved with L-BFGS, enforcing simplex constraints and entropic sparsity with ρ̃_rna = ρ̃_adt = 0.01, ρ̃_atac = 0.1, and μ̃ = 0.001. Call model.train(mdata) to fit; this stores the shared embedding in mdata.obsm['W_OT'] and the modality-specific dictionaries in mdata.uns, and save the checkpoint as mowgli_model.pt.

## Related tools

- **Mowgli** (Core Python package that implements integrative NMF + entropic OT training and exposes model.train(mdata) within the scverse ecosystem) — https://github.com/cantinilab/Mowgli
- **PyTorch** (Backend used to implement the fast, GPU-enabled Sinkhorn algorithm and L-BFGS optimization of the dual problems)
- **Muon** (Provides the Muon object container for paired multi-omics data and per-modality preprocessing utilities (filter_var, tf_idf, clr))
- **Scanpy** (Used for preprocessing (normalize_total, log1p, highly_variable_genes) and downstream analysis of the trained embedding)

## Examples

```
pip install mowgli; import mowgli; model.train(mdata); W = mdata.obsm['W_OT']
```

## Evaluation signals

- The trained embedding in mdata.obsm['W_OT'] has shape n_cells × k (e.g., n_cells × 15 for the demo)
- Rows of W respect the simplex constraint (nonnegativity and rows/columns summing to 1 for W and H(p) respectively)
- Silhouette score, Adjusted Rand Index, and purity score computed on the embedding are competitive with the paper's benchmark against Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate
- On controlled cell line datasets (except 90% sparse), Mowgli's metrics should be greater or equal to other methods; in 96% sparse, the three cell populations should be correctly separated
- Installation/training succeeds in the tested environment (Python 3.10 via pip install mowgli, continuously tested on Ubuntu)

## Limitations

- Optimal transport is inherently expensive to compute compared to Euclidean distance; GPU availability is required for Mowgli to scale to the larger datasets presented
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values of k should be tested
- Mowgli does not handle batch correction when integrating paired multi-omics data profiled across centers
- Benchmarking on Seurat v4 comparisons is partially restricted because Seurat does not provide an embedding, so the silhouette score cannot be computed for it

## Evidence

- [abstract] Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport: "Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing at the same time the clustering performance and interpretability of integrative Nonnegative Matrix"
- [methods] GPU-enabled Sinkhorn: [evidence span withheld — anchor status: not_found]
- [methods] Alternating L-BFGS optimization: "we alternate between minimizing (Eq. 5) on H p ð Þ and W"
- [results] scverse integration: [evidence span withheld — anchor status: not_found]
- [readme] Installation via PyPI: [evidence span withheld — anchor status: external_doc]
- [results] Benchmark metrics: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance"
