---
name: single-cell-denoising-method-benchmarking
description: Use when a sparse single-cell expression matrix with known cell-phenotype
  labels (e.g., 95% sparse PBMC or mouse cortex data) must be denoised and you need
  to verify whether RMT feature selection outperforms MAGIC, scImpute, ZIFA, ZIMB-WaVE,
  scVI, DCA, Seurat, or HVG baselines.
when_to_use_negative:
- Input is bulk RNA-seq or already a denoised/imputed feature table — the RMT decomposition
  assumes a highly sparse raw single-cell matrix (about 95% sparsity) as noise plus
  low-rank signal
- No ground-truth phenotype labels or known cell populations exist, since the benchmark
  is scored by silhouette of marked phenotypes rather than intrinsic metrics
- Only one denoising method is of interest without comparison baselines — the skill
  is a comparative benchmark across at least the RMT, imputation, zero-inflated, and
  deep-learning families
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Randomly
  role: 'RMT-based denoising: eigenvalue/eigenvector-driven feature selection removing
    genes responsible for localized eigenvectors, and low-rank signal extraction'
  repo: https://github.com/luisaparicio/randomly
- name: MAGIC
  role: Imputation baseline compared against RMT in the silhouette-score benchmark
- name: scImpute
  role: Imputation baseline compared against RMT in the silhouette-score benchmark
- name: ZIFA
  role: Zero-inflated dimensionality-reduction baseline compared against RMT
- name: ZIMB-WaVE
  role: Zero-inflated dimensionality-reduction baseline compared against RMT
- name: scVI
  role: Non-linear neural-network baseline compared against RMT
- name: DCA
  role: Non-linear neural-network baseline compared against RMT
- name: Seurat
  role: Standard scRNA-seq processing baseline compared against RMT
- name: pip
  role: Installation channel for the randomly package (pip install randomly)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:93bda9a36d9d3b36257b30e865cdb44db42f783331871550c2519ba4dcd4b109
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
  - Seurat
  - ZIFA
  - ZIMB-WaVE
  - pip
  - scImpute
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# single-cell-denoising-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the RMT-based denoising pipeline (Randomly) against imputation, zero-inflated, and deep-learning baselines on sparse single-cell expression matrices by measuring mean silhouette scores of known phenotype clusters across latent-space dimensions.

## When to use

Use when a sparse single-cell expression matrix with known cell-phenotype labels (e.g., 95% sparse PBMC or mouse cortex data) must be denoised and you need to verify whether RMT feature selection outperforms MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, or HVG baselines.

## When NOT to use

- Input is bulk RNA-seq or already a denoised/imputed feature table — the RMT decomposition assumes a highly sparse raw single-cell matrix (about 95% sparsity) as noise plus low-rank signal
- No ground-truth phenotype labels or known cell populations exist, since the benchmark is scored by silhouette of marked phenotypes rather than intrinsic metrics
- Only one denoising method is of interest without comparison baselines — the skill is a comparative benchmark across at least the RMT, imputation, zero-inflated, and deep-learning families

## Inputs

- Sparse cell-gene expression matrix of 6,573 human PBMCs (Kang et al.) with 13 phenotype labels
- Sparse cell-gene expression matrix of 3,005 mouse cortex cells (Zeisel et al.) with 7-, 15-, and 26-group phenotype labels
- List of baseline denoising methods to compare (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw)

## Outputs

- silhouette_scores.csv with mean silhouette score per method, per latent dimension, and per dataset/phenotype grouping
- Denoised/low-rank expression representations for each method and latent-space embedding used for silhouette computation

## How to apply

Load the sparse cell-gene expression matrices with their phenotype labels (6,573-cell human PBMCs from Kang et al.; 3,005-cell mouse cortex cells from Zeisel et al.). Denoise each matrix with Randomly, which models the data as a random matrix plus a low-rank perturbation: it randomizes the dataset (permuting cell labels per gene) to detect sparsity-induced eigenvector localization, discards the ~1,500 most signal-like genes responsible for localized eigenvectors in the randomized case, and extracts a low-rank signal representation. Run each baseline (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw matrix) on the same input. Project every denoised result into a reduced latent space, sweeping over latent dimensions, and compute the mean silhouette coefficient of the phenotype clusters (13 PBMC phenotypes; 7, 15, and 26 mouse cortex groupings). Tabulate mean silhouette score per method, latent dimension, and dataset into silhouette_scores.csv, reproducing the Figure 6A-6D comparison.

## Related tools

- **Randomly** (RMT-based denoising: eigenvalue/eigenvector-driven feature selection removing genes responsible for localized eigenvectors, and low-rank signal extraction) — https://github.com/luisaparicio/randomly
- **MAGIC** (Imputation baseline compared against RMT in the silhouette-score benchmark)
- **scImpute** (Imputation baseline compared against RMT in the silhouette-score benchmark)
- **ZIFA** (Zero-inflated dimensionality-reduction baseline compared against RMT)
- **ZIMB-WaVE** (Zero-inflated dimensionality-reduction baseline compared against RMT)
- **scVI** (Non-linear neural-network baseline compared against RMT)
- **DCA** (Non-linear neural-network baseline compared against RMT)
- **Seurat** (Standard scRNA-seq processing baseline compared against RMT)
- **pip** (Installation channel for the randomly package (pip install randomly))

## Examples

```
pip install randomly
```

## Evaluation signals

- silhouette_scores.csv exists and every silhouette value lies in the valid range [-1, 1]
- RMT rows attain the maximum mean silhouette score per dataset and per latent dimension, as reported in Figure 6A-6D
- The RMT advantage over baselines grows as the number of phenotype populations and latent-space dimensions increase
- The RMT-selected feature set contains the 1,500 most signal-like genes obtained after discarding genes driving localized eigenvectors in the randomized (sparsity-only) data
- Both benchmark datasets (6,573-cell PBMC with 13 phenotypes; 3,005-cell mouse cortex with 7/15/26 groupings) are represented in the results table

## Limitations

- Randomly only works with Python 3 (not Python 2)
- The approach assumes roughly 95% of the information in single-cell data is RMT-compatible noise and about 3% is sparsity-induced spurious signal; departures from this regime may degrade denoising
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data if not removed via the randomized-data filter
- No version or release information is provided for the publicly available algorithm code, so exact reproducibility may be affected

## Evidence
