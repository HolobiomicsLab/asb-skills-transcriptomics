---
name: signal-robustness-perturbation-testing
description: Use when an RMT-based noise-filtering pass (e.g., on ZhengMix PBMC or
  similar sparse scRNA-seq data) has flagged candidate signal dimensions and you need
  a data-driven threshold to keep only robust, biologically meaningful signals before
  UMAP embedding.
when_to_use_negative:
- Input data has not been preprocessed with log normalization plus L2 normalization
  — signal distortion from conventional log normalization alone (e.g., 84 signals
  on ZhengMix) invalidates the robustness comparison baseline
- The goal is imputation of dropout zeros rather than signal filtering — the robustness
  test removes dropout-driven signals without imputing values
- The dataset is too large for full eigenvalue decomposition of the cell-to-cell similarity
  matrix, which requires significant memory resources
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scLENS
  role: Implements the RMT-based noise filtering plus the signal robustness test (10
    sparse perturbations, robustness >0.5) and exposes plot_stability for inspection
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python environment packaging of scLENS with PyTorch CUDA support for GPU execution
    of the perturbation pipeline
  repo: Mathbiomed/scLENSpy
- name: UMAP.jl
  role: Produces the 2D embedding from the selected robust signals (n_neighbors=15,
    min_dist=0.01, cosine metric) via scLENS.apply_umap!
- name: Julia
  role: Runtime for the scLENS package executing QC, perturbation testing, and embedding
- name: CUDA
  role: Optional GPU acceleration for scLENS; falls back to CPU automatically if unavailable
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:0b5e058c7e2e2efc3a4346b486401f89e75bd6dae2309b3e77156a35dab0454f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CUDA
  - Julia
  - UMAP.jl
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# signal-robustness-perturbation-testing

## Summary

scLENS's signal robustness test filters low-quality signal dimensions identified by RMT-based noise filtering by checking each signal's stability against sparse perturbations of the scRNA-seq data, enabling data-driven, imputation-free removal of signals arising from stochastic dropout.

## When to use

Use when an RMT-based noise-filtering pass (e.g., on ZhengMix PBMC or similar sparse scRNA-seq data) has flagged candidate signal dimensions and you need a data-driven threshold to keep only robust, biologically meaningful signals before UMAP embedding.

## When NOT to use

- Input data has not been preprocessed with log normalization plus L2 normalization — signal distortion from conventional log normalization alone (e.g., 84 signals on ZhengMix) invalidates the robustness comparison baseline
- The goal is imputation of dropout zeros rather than signal filtering — the robustness test removes dropout-driven signals without imputing values
- The dataset is too large for full eigenvalue decomposition of the cell-to-cell similarity matrix, which requires significant memory resources

## Inputs

- Preprocessed scRNA-seq expression matrix (CSV or JLD2 DataFrame with :cell column) after QC, log normalization (L=1), and L2 normalization
- Candidate signal eigenvalues/eigenvectors from RMT-based noise filtering of the cell-to-cell similarity matrix
- Perturbation parameters: number of perturbed datasets (10) and binary sparse perturbation sparsity level (>0.97)

## Outputs

- Robustness value per candidate signal dimension
- Final set of robust signal dimensions (e.g., 13 on ZhengMix)
- Low-dimensional embedding coordinates suitable for UMAP and export to CSV (e.g., zhengmix_signal_counts.csv, out/pca.csv, out/umap.csv)

## How to apply

After log normalization (scaling factor 1), L2 normalization of cell vector lengths, and RMT-based noise filtering on the cell similarity matrix G=XX^T/N, generate 10 perturbed datasets by adding a binary sparse perturbation matrix (sparsity level selected iteratively, typically >0.97). For each candidate signal, compute the average maximum absolute correlation between its eigenvector and the eigenvectors of the perturbed datasets, and retain signals with robustness value >0.5. This removes low-quality signals from stochastic dropout events without requiring imputation; on ZhengMix it reduced 42 log+L2 signals to 13 robust signals whose 2D UMAP embedding (n_neighbors=15, min_dist=0.01, cosine metric) clearly separated T-cell subtypes. In the Julia implementation, the robustness behavior can be inspected via scLENS.plot_stability on the sclens_embedding object returned by scLENS.sclens.

## Related tools

- **scLENS** (Implements the RMT-based noise filtering plus the signal robustness test (10 sparse perturbations, robustness >0.5) and exposes plot_stability for inspection) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python environment packaging of scLENS with PyTorch CUDA support for GPU execution of the perturbation pipeline) — Mathbiomed/scLENSpy
- **UMAP.jl** (Produces the 2D embedding from the selected robust signals (n_neighbors=15, min_dist=0.01, cosine metric) via scLENS.apply_umap!)
- **Julia** (Runtime for the scLENS package executing QC, perturbation testing, and embedding)
- **CUDA** (Optional GPU acceleration for scLENS; falls back to CPU automatically if unavailable)

## Examples

```
using scLENS; pre_df = scLENS.preprocess(scLENS.read_file("data/Z8eq.csv.gz")); emb = scLENS.sclens(pre_df, device_="gpu"); scLENS.plot_stability(emb); scLENS.apply_umap!(emb)
```

## Evaluation signals

- Signal count cascade reproduced on ZhengMix: 84 (log only) -> 42 (log+L2) -> 13 robust signals after the robustness test
- Every retained signal has average maximum absolute correlation (robustness value) >0.5 across the 10 perturbed datasets
- Perturbation sparsity level selected iteratively and exceeds 0.97
- UMAP (n_neighbors=15, min_dist=0.01, cosine metric) of the 13-signal embedding clearly separates T-cell subtypes, unlike the 84- or 42-signal embeddings
- scLENS.plot_stability output shows stable clustering stability for retained signals

## Limitations

- RMT-based filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources; large datasets with whole genes need a high-specification computer
- The binary sparse perturbation sparsity level must be selected iteratively, adding a tuning step to the pipeline
- scLENS supports saving results as a Seurat object only in version 1.0.0; current versions output CSV and AnnData

## Evidence

- [abstract] RMT-based noise filtering and a signal robustness test enable data-driven determination of the threshold for signal dimensions: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] Robustness test removes dropout signals without imputation: "scLENS removes low-quality signals arising from stochastic dropout events using a signal robustness test which does not require imputation"
- [abstract] L2 normalization rationale: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [readme] QC thresholds in preprocess function: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [discussion] Full eigenvalue decomposition memory demand: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
- [readme] Stability plotting API: "scLENS.plot_stability(sclens_embedding)"
