---
name: mouse-cortex-cell-type-grouping-analysis
description: Use when denoising a sparse single-cell RNA-seq dataset (e.g., 3,005-cell
  mouse cortex from Zeisel et al.) with known cell-type labels and you need to quantify
  cluster resolution via mean silhouette score after RMT feature selection versus
  baseline denoising methods.
when_to_use_negative:
- The dataset has no known phenotype/cell-type labels, so silhouette scores against
  marked populations cannot be computed.
- The input matrix is already a denoised or low-rank feature embedding rather than
  a raw sparse cell-gene count matrix.
- The goal is de novo cluster discovery rather than benchmarking denoising methods
  against known marked cell populations.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Randomly
  role: 'RMT-based denoising and feature selection: removes genes causing spurious
    eigenvector localization in the randomized matrix and extracts a low-rank signal
    representation'
  repo: https://github.com/luisaparicio/randomly
- name: MAGIC
  role: Imputation baseline used for silhouette-score comparison against RMT
- name: scImpute
  role: Imputation baseline used for silhouette-score comparison against RMT
- name: ZIFA
  role: Zero-inflated dimensionality-reduction baseline used for comparison
- name: ZIMB-WaVE
  role: Zero-inflated dimensionality-reduction baseline used for comparison
- name: scVI
  role: Deep-learning (neural network) denoising baseline used for comparison
- name: DCA
  role: Deep-learning (neural network) denoising baseline used for comparison
- name: pip
  role: Installation channel for the Randomly package (Python 3 only)
  repo: https://github.com/luisaparicio/randomly
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:9dade3ea8edd612dae6b19539b56acc7203a156dd3caa8d38631ec489ea9b044
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  repo_url: https://github.com/luisaparicio/randomly
  tools:
  - DCA
  - MAGIC
  - Python 3
  - Randomly
  - ZIFA
  - ZIMB-WaVE
  - pip
  - scImpute
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mouse-cortex-cell-type-grouping-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the silhouette-score benchmark from Aparicio et al. (Patterns, 2020) in which RMT-based denoising of the 3,005-cell mouse cortex dataset (Zeisel et al.) is compared against imputation, zero-inflated, and deep-learning baselines by computing mean silhouette scores of known phenotype groupings (7, 15, and 26 groups) in latent spaces of varying dimension.

## When to use

Use when denoising a sparse single-cell RNA-seq dataset (e.g., 3,005-cell mouse cortex from Zeisel et al.) with known cell-type labels and you need to quantify cluster resolution via mean silhouette score after RMT feature selection versus baseline denoising methods.

## When NOT to use

- The dataset has no known phenotype/cell-type labels, so silhouette scores against marked populations cannot be computed.
- The input matrix is already a denoised or low-rank feature embedding rather than a raw sparse cell-gene count matrix.
- The goal is de novo cluster discovery rather than benchmarking denoising methods against known marked cell populations.

## Inputs

- 3,005-cell mouse cortex single-cell expression matrix (Zeisel et al.), ~95% sparse cell-gene count matrix
- Phenotype/cell-type label vector for the mouse cortex cells at 7, 15, and 26 grouping resolutions
- Denoised expression matrices produced by Randomly (RMT, 1,500 signal-like genes) and each baseline method (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw)

## Outputs

- Reduced latent-space cell embeddings per method and per latent dimension
- silhouette_scores.csv: mean silhouette score per method, per latent dimension, and per phenotype grouping (7/15/26 groups)

## How to apply

Load the 3,005-cell mouse cortex expression matrix from Zeisel et al. together with its phenotype labels at three grouping granularities (7, 15, and 26 groups). Denoise the matrix with the Randomly package, which models the data as a random matrix plus a low-rank perturbation and selects the most signal-like genes (1,500 in the benchmark) by discarding genes responsible for eigenvector localization in the per-gene-randomized (sparsity-only) matrix. Run each baseline — MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 highly variable genes, and the raw unfiltered matrix — under identical settings. Project each denoised result into a reduced latent space while sweeping over latent dimensions, then compute the mean silhouette coefficient of phenotype clusters on the embedded cell coordinates. Tabulate mean silhouette per method, per latent dimension, and per grouping; the validation expectation from Figure 6 of the paper is that RMT attains the highest mean silhouette score, with its advantage growing as the number of phenotype groups and latent dimensions increase.

## Related tools

- **Randomly** (RMT-based denoising and feature selection: removes genes causing spurious eigenvector localization in the randomized matrix and extracts a low-rank signal representation) — https://github.com/luisaparicio/randomly
- **MAGIC** (Imputation baseline used for silhouette-score comparison against RMT)
- **scImpute** (Imputation baseline used for silhouette-score comparison against RMT)
- **ZIFA** (Zero-inflated dimensionality-reduction baseline used for comparison)
- **ZIMB-WaVE** (Zero-inflated dimensionality-reduction baseline used for comparison)
- **scVI** (Deep-learning (neural network) denoising baseline used for comparison)
- **DCA** (Deep-learning (neural network) denoising baseline used for comparison)
- **pip** (Installation channel for the Randomly package (Python 3 only)) — https://github.com/luisaparicio/randomly

## Examples

```
pip install randomly; import randomly; model = randomly.Denoiser(); genes, X_lowrank = model.fit_transform(cortex_matrix); silhouette_score(X_emb, cortex_labels)
```

## Evaluation signals

- silhouette_scores.csv exists and every silhouette value lies in the valid range [-1, 1].
- For each dataset, latent dimension, and phenotype grouping, the RMT row shows the maximum mean silhouette score, matching Figure 6A-6D.
- The RMT advantage over baselines grows as the number of phenotype groups (7, 15, 26) and the number of latent dimensions increase.
- RMT uses exactly the 1,500 most signal-like genes selected by discarding genes responsible for localized eigenvectors in the randomized data.
- All compared methods are evaluated on the same 3,005-cell Zeisel et al. input matrix with identical latent-dimension sweeps.

## Limitations

- Randomly only works with Python 3, not Python 2.
- The benchmark relies on pre-defined phenotype labels (marked datasets); performance on datasets without known populations is not demonstrated by this comparison.
- The paper provides no version or release information for the publicly available algorithm code, complicating exact reproduction.

## Evidence

- [results] RMT outperforms baselines on marked populations: [evidence span withheld — anchor status: not_found]
- [results] silhouette metric and groupings: [evidence span withheld — anchor status: not_found]
