---
name: rmt-eigenvalue-noise-filtering
description: Use when preprocessing scRNA-seq count data (e.g., ZhengMix PBMC) and
  you need a data-driven, non-manual threshold to separate signal dimensions from
  noise before dimensionality reduction or UMAP embedding.
when_to_use_negative:
- When the input is already a low-dimensional feature embedding or a feature table
  of highly variable genes — RMT filtering operates on the full gene-by-cell count
  matrix, and selecting highly variable genes first can compromise local structure
  in low-variance genes
- When the dataset is too large for available hardware — the approach requires computing
  all eigenvalues of the large cell-to-cell similarity matrix, demanding significant
  memory resources and a computer with high specifications
- When the user needs an interactive, subjective dimension cutoff — this skill is
  designed to replace manual threshold decisions, so do not apply it expecting user-tunable
  signal counts
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scLENS
  role: Implements log normalization with scaling factor 1, L2 normalization, RMT-based
    noise filtering with Marchenko-Pastur fitting and Tracy-Widom thresholding, and
    the signal robustness test
  repo: https://github.com/Mathbiomed/scLENS
- name: UMAP.jl
  role: Produces 2D embeddings (n_neighbors=15, min_dist=0.01, cosine metric) from
    the RMT-filtered signal dimensions for downstream validation
- name: CUDA
  role: Optional GPU acceleration for the eigenvalue computations; the package automatically
    falls back to CPU when CUDA is unavailable
- name: scLENSpy
  role: Python packaging of scLENS with conda environment (environment.yml) and PyTorch
    CUDA 12.8 installation instructions
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:c4b3824b4b0b71dcee12679b7c114e6f0dca820926f229c4a690fd22fc827c09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# rmt-eigenvalue-noise-filtering

## Summary

Random matrix theory (RMT)-based noise filtering, as implemented in scLENS, builds the cell-to-cell similarity matrix G=XX^T/N, computes all eigenvalues, fits the Marchenko-Pastur distribution, and counts eigenvalues exceeding the Tracy-Widom threshold (p=0.05) to provide a data-driven signal/noise cutoff for scRNA-seq data.

## When to use

Use when preprocessing scRNA-seq count data (e.g., ZhengMix PBMC) and you need a data-driven, non-manual threshold to separate signal dimensions from noise before dimensionality reduction or UMAP embedding.

## When NOT to use

- When the input is already a low-dimensional feature embedding or a feature table of highly variable genes — RMT filtering operates on the full gene-by-cell count matrix, and selecting highly variable genes first can compromise local structure in low-variance genes
- When the dataset is too large for available hardware — the approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources and a computer with high specifications
- When the user needs an interactive, subjective dimension cutoff — this skill is designed to replace manual threshold decisions, so do not apply it expecting user-tunable signal counts

## Inputs

- scRNA-seq cell-by-gene count matrix (CSV with genes in the first row and cell IDs in the first column, or JLD2 DataFrame with a ':cell' column)
- 10x formatted data (matrix.mtx.gz, features.tsv.gz, barcodes.tsv.gz) convertible via tenx2jld2

## Outputs

- Count of detected signal eigenvalues exceeding the Tracy-Widom threshold (p=0.05), e.g., 42 on ZhengMix with log+L2 preprocessing
- Selected robust signal dimensions with robustness values >0.5 from the signal robustness test
- Low-dimensional embedding (PCA coordinates) and 2D UMAP coordinates exportable to CSV or AnnData (.h5ad)

## How to apply

Load the cell-by-gene count matrix (CSV or 10x gz converted to JLD2 via tenx2jld2) and run quality control, filtering cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells. Preprocess with log normalization (scaling factor set to 1) plus L2 normalization of cell vector lengths — L2 normalization is essential because conventional log normalization alone distorts signals, inflating detected signals (e.g., 84 on ZhengMix vs. 42 with log+L2). Build the cell similarity matrix G=XX^T/N and compute all eigenvalues, then fit the Marchenko-Pastur distribution and count eigenvalues exceeding the Tracy-Widom threshold at p=0.05 as candidate signals. Optionally refine via the signal robustness test: generate 10 perturbed datasets by adding a binary sparse perturbation matrix (sparsity >0.97), compute each signal's average maximum absolute correlation with perturbed eigenvectors, and retain signals with robustness >0.5 (e.g., 13 robust signals on ZhengMix). Construct embeddings from the selected eigenvalues/vectors and apply UMAP (UMAP.jl, n_neighbors=15, min_dist=0.01, cosine metric) to verify downstream separation.

## Related tools

- **scLENS** (Implements log normalization with scaling factor 1, L2 normalization, RMT-based noise filtering with Marchenko-Pastur fitting and Tracy-Widom thresholding, and the signal robustness test) — https://github.com/Mathbiomed/scLENS
- **UMAP.jl** (Produces 2D embeddings (n_neighbors=15, min_dist=0.01, cosine metric) from the RMT-filtered signal dimensions for downstream validation)
- **CUDA** (Optional GPU acceleration for the eigenvalue computations; the package automatically falls back to CPU when CUDA is unavailable)
- **scLENSpy** (Python packaging of scLENS with conda environment (environment.yml) and PyTorch CUDA 12.8 installation instructions) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; pre_df = scLENS.preprocess(scLENS.read_file("data/ZhengMix.csv.gz")); emb = scLENS.sclens(pre_df, device_="gpu"); scLENS.apply_umap!(emb)
```

## Evaluation signals

- On ZhengMix PBMC data, detected signals should decrease from 84 (log normalization alone) to 42 (log + L2 normalization), confirming L2 normalization is suppressing log-induced signal distortion
- After the signal robustness test, the count should drop to 13 robust signals on ZhengMix (robustness value >0.5 across 10 binary sparse perturbations, sparsity >0.97)
- The eigenvalue histogram should be fit by a Marchenko-Pastur distribution, and the Marchenko-Pastur and clustering stability plots (plot_mpdist, plot_stability) should render without errors
- Only the full-pipeline (13-signal) embedding should show clear separation of T-cell subtypes in the 2D UMAP, quantitatively outperforming the 84- and 42-signal embeddings
- QC filtering must remove cells with <200 expressed genes and genes expressed in <15 cells before similarity matrix construction

## Limitations

- Computing all eigenvalues of the large cell-to-cell similarity matrix requires significant memory resources; operation on large datasets with whole genes still requires a computer with high specifications
- RMT-based filtering without L2 normalization can over-count signals due to distortion introduced by conventional log normalization (84 vs. 42 signals on ZhengMix)
- GPU support requires an NVIDIA GPU with appropriate drivers; the package automatically falls back to CPU if CUDA is unavailable

## Evidence

- [abstract] RMT with L2 normalization: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [abstract] RMT noise filtering and robustness test: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] Scaling factor of 1: "we set a scaling factor of 1 during the first step of log normalization"
- [discussion] Memory demand of RMT approach: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
- [readme] QC thresholds in preprocess function: "During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
